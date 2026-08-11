---
title: "Overlaybd — The Ultimate Image Format for Every Workload"
date: 2026-08-10
slug: overlaybd-the-ultimate-image-format
tags: ["overlaybd", "image", "container", "virtualization", "VM", "sandbox"]
---

Containers, VMs, and agent sandboxes all start the same way: from an
image. Yet the mainstream image formats were each designed for exactly
one of them — OCI tar+gzip layers for containers, qcow2/VHD/VMDK for
VMs — and each carries structural costs that only grow with scale.

[Overlaybd](https://github.com/containerd/overlaybd) takes a different
route: a **layered, lazily-loaded, seekably-compressed block-device
image format**. One format serves containers, VM-isolated secure
containers, agent sandboxes, and full virtual machines alike — and it
has been proven for years in some of the largest production fleets in
the world. This post synthesizes the case.

## The Two Camps, and Why Both Fall Short

Existing image stacks fall into two architectural camps:

- **Filesystem-based images.** The dominant OCI tar+gzip format served
  through overlayfs; lazy-pull variants (eStargz, SOCI) that defer
  extraction but keep tar and filesystem semantics; single-filesystem
  images (squashfs, EROFS); and FUSE-based formats (Nydus/RAFS).
- **Block-based VM disks.** qcow2, VHD/VHDX, and VMDK — each a virtual
  block device plus a fixed-granularity allocation table, chained one
  file per snapshot.

Measured against what these workloads actually need — sub-second cold
start, efficiency at thousands of concurrent instances, stability and
security as business-critical infrastructure, and one format across
runtimes and guest OSes — both camps fail structurally.

### Cold start

The standard OCI image must be downloaded, decompressed, and extracted
in full before the process can run — even though only a small fraction
of image data is touched during startup. For multi-gigabyte images that
means tens of seconds to minutes. P2P tools (Dragonfly, Kraken) speed
up *distribution* but cannot skip the download-and-decompress steps.
Lazy-pull variants defer extraction yet remain bound to archive
structure and filesystem semantics.

### O(n) lookup with depth

Filesystem stacking resolves every file access — open, stat, readdir —
by walking each layer's directory tree top-down: O(n) in the number of
layers. The VM camp has the same disease in a different shape: a read
miss in a qcow2/VHD/VMDK backing chain falls through parent files until
some snapshot holds the data. Vendor guidance says the quiet part out
loud — VMware supports at most 32 snapshots per chain and recommends
2–3; Microsoft flags "more than 50 checkpoints" as a problem; QEMU
ships `block-commit`/`block-stream` to collapse chains. The
distribution model demands deep stacks (base OS, runtime, dependencies,
app, per-tenant customization); the formats only tolerate shallow ones.

### Index memory, multiplied by depth

qcow2's two-level L1/L2 table runs ~12.5 MB per 100 GB of virtual disk
at the default 64 KB cluster size — *per file*, and every link in a
backing chain carries its own table and cache. A 30-snapshot chain has
a 30 × 12.5 MB index problem. QEMU's default L2 cache is only ~1 MB,
forcing a choice between wasting host memory or wasting I/O on table
misses. VMDK and VHD/VHDX sit on the same trade-off curve at different
points (smaller tables bought with coarser grains).

### The granularity dilemma

For fixed-granularity tables, index size is proportional to virtual
size ÷ cluster size, while copy-on-write cost is proportional to
cluster size. Small clusters: cheap writes, huge tables. Large
clusters: tiny tables, but a 4 KB write can trigger a multi-megabyte
copy. No fixed-granularity format escapes this curve — you can move
along it, but not off it. Filesystem formats pay the analogous tax as
file-level copy-up: the first modification to lower-layer data copies
the *entire file*.

### Serving-path complexity and security

Filesystem-based formats must parse complex metadata (xattrs, ACLs,
symlinks, device nodes) in host or hypervisor context — a large attack
surface facing untrusted guest content. Served into a VM, every
metadata operation becomes a FUSE/virtio-fs round-trip; a single
`open()` on a nested path may cost five or more cross-VM round-trips.
Worse, virtio-fs/9p have been shown to let guests bypass memory-cgroup
and ephemeral-storage accounting entirely
([kata-containers#12203](https://github.com/kata-containers/kata-containers/issues/12203)).
And when the userspace FUSE daemon dies, the mounted filesystem
typically hangs — a serious liability for infrastructure that serves
around the clock.

## Overlaybd's Advantages

Overlaybd presents each image as a virtual block device backed by a
stack of block-level layers. The design choices that follow from that
one decision resolve each problem above.

### A merged extent index: O(1) at any depth

At load time, overlaybd merges all layer indices into a single LSMT index of
variable-length extent records (16 bytes each). Chain depth never
enters the data path: a read resolves against one index whether the
image carries two layers or fifty. Index size tracks fragmentation, not
virtual size — in Alibaba's production environment, images larger than
50 GB average **under 300 KB** of merged index, small enough to stay
fully memory-resident across thousands of concurrent instances.

{{< img src="img-index.png" alt="Merged index sizes in Alibaba production" width="80%" >}}

### A query engine built for speed

LBA lookup is a segment-search problem over a sorted set of
non-overlapping intervals. Overlaybd
replaced the original `std::lower_bound` binary search with a
linearized B+ tree vectorized with AVX-512 batch comparison — over 10×
faster, sustaining hundreds of millions of lookups per second on a
single core:

| Segment count | B+tree + AVX-512 | B+tree + bitmask loop | std::lower_bound |
| --- | --- | --- | --- |
| 1K | 220 M/s | 42.2 M/s | 18.3 M/s |
| 10K | 160 M/s | 30.7 M/s | 12.8 M/s |
| 100K | 108 M/s | 21.8 M/s | 8.6 M/s |
| 1M | 57.4 M/s | 15.2 M/s | 5.6 M/s |

The algorithm is so fast it is even applicable to IP address lookup in
backbone core routers (see my paper [PlanB, NSDI
'26](https://www.usenix.org/conference/nsdi26/presentation/zhang-zhihao)).

### Sector-granularity writes: no copy-on-write, ever

The writable
layer is part of the format itself, indexed at 512-byte sector
granularity — the smallest unit any block I/O can cover — so a write
always covers whole sectors and never triggers a copy. Write cost is
proportional to write size, not file or cluster size. This is what
breaks the granularity dilemma: the mapping is decoupled from write
granularity, so the index stays small *and* writes stay cheap.

### On-demand fetching with seekable compression

Containers and VMs
start by reading image data remotely as needed — a 1 GB+ image launches
in under a second. Data is compressed in small, independently
addressable units (ZFile, LZ4/zstd), so a random read fetches and
decompresses only the relevant unit; because the compressed transfer is
smaller, compressed random reads are often *faster* than uncompressed
ones. Trace-based or file-list-based prefetch warms the cache ahead of
demand, and P2P distribution spreads block fetches across peers when
thousands of instances start at once.

### A stateless, minimal serving path

The host sees only a sequence of
block reads and writes. It never parses filesystem structures, never
interprets symlinks or xattrs, and keeps no session state shared with
the guest — every request carries all information needed to serve it.
The consequences compound: the attack surface is the block interface
itself, one of the oldest and most battle-tested boundaries in
computing; the service recovers from crashes by simply re-attaching the
device; and snapshot, clone, cross-host restore, and migration reduce
to a disk sync plus access to the same backing layers — a decisive
feature for agent sandboxes that fork on every speculative branch. The
userspace block server is built on
[PhotonLibOS](https://github.com/alibaba/PhotonLibOS), a coroutine
runtime that handles tens of thousands of concurrent I/O streams
without thread-per-connection overhead.

## One Format, Every Workload

A block device is the most universal storage abstraction in computing —
anything that can attach one can use overlaybd:

- **Containers** (runc) see a normal ext4 filesystem mounted from the
  device on the host, delivered through a containerd snapshotter and
  the OCI registry ecosystem.
- **Secure containers** (Kata, Firecracker) attach the device via
  virtio-blk: all metadata operations complete in-guest with zero
  cross-VM round-trips, and only data I/O crosses the boundary.
- **Agent sandboxes** get sub-second cold start, cheap snapshot/fork,
  strong isolation for untrusted code, and massive concurrency from
  shared base images.
- **Virtual machines** replace per-file allocation tables with the
  merged extent index, gaining O(1) lookup at any chain depth and
  OCI-style layered distribution through existing registries.
- **Any guest OS** — Linux, Windows, Android, macOS — understands a
  block device, with no guest agent or kernel module required. The
  filesystem inside remains a free choice: ext4, XFS, Btrfs, EROFS,
  NTFS.

Notably, EROFS's answer to the O(n) layer walk — a pre-built merged
view flattened onto a single block device for VM pass-through — is an
explicit acknowledgment of the same conclusion: the way out of
filesystem stacking is the block-device route.

## Production Evidence

This is not a research prototype:

- **Alibaba** has run overlaybd for years across its entire application
  portfolio — Taobao, TMall, AlibabaCloud and more — and commercialized
  it on AlibabaCloud as the container image acceleration offering.
  Function Compute runs function microVMs on the same architecture.
- **[Azure](https://learn.microsoft.com/en-us/azure/aks/artifact-streaming-overview)**
  built AKS Artifact Streaming on overlaybd.
- **[Databricks](https://www.databricks.com/blog/booting-databricks-vms-7x-faster-serverless-compute)**
  reported 7× faster VM startup for serverless compute;
  [Superhuman](https://www.databricks.com/blog/how-superhuman-and-databricks-built-200k-qps-inference-platform-together)
  built a 200K-QPS inference platform on the same infrastructure.
- **[DeepSeek Elastic Compute](https://arxiv.org/html/2606.19348v1)**
  runs its agent execution environment on overlaybd-format images.
- **[fly.io](https://community.fly.io/t/experimental-speedy-machine-creation-with-overlaybd/18958)**
  boots Firecracker microVMs from overlaybd images;
  **[hocus.dev](https://hocus.dev/blog/virtualizing-development-environments)**
  backs microVM development environments with it.
- **[Google Colab](https://medium.com/@gogasca_/using-overlaybd-to-improve-startup-time-6c5f90f23345)**
  starts a 27 GB runtime image in ~170 ms warm, ~5.6 s cold.
- **[Kimi AgentEnv](https://github.com/kvcache-ai/AgentEnv)** (Moonshot
  AI) independently *re-implemented* the open format as its sandbox
  image substrate, achieving sub-second launch latency.

The design is peer-reviewed: [DADI
(ATC '20)](https://www.usenix.org/conference/atc20/presentation/li-huiba)
describes the large-scale deployment at Alibaba, and [FaaSNet
(ATC '21)](https://www.usenix.org/conference/atc21/presentation/wang-ao)
applies it to Function Compute.

## Open Source

Overlaybd is an open-source sub-project of
[containerd](https://www.cncf.io/projects/containerd/) (CNCF
graduated), Apache-2.0 licensed, with fully documented specifications
([LSMT](https://containerd.github.io/overlaybd/#/specs/lsmt.md),
[ZFile](https://containerd.github.io/overlaybd/#/specs/zfile.md)) open
for independent implementation:

- Data I/O path: [containerd/overlaybd](https://github.com/containerd/overlaybd)
- Snapshotter & conversion tools: [containerd/accelerated-container-image](https://github.com/containerd/accelerated-container-image)
- P2P distribution: [data-accelerator/dadi-p2proxy](https://github.com/data-accelerator/dadi-p2proxy)

## Conclusion

Conventional formats ask you to choose: fast cold start *or* small
metadata *or* cheap writes *or* runtime universality. Overlaybd's
block-level design — a merged extent index, sector-granularity writes,
on-demand seekably-compressed fetch, and a stateless block-device
serving path — removes those trade-offs together, and does it with one
format for every consumer. For infrastructure that starts, snapshots,
and distributes workloads at scale, it is the stronger foundation.
