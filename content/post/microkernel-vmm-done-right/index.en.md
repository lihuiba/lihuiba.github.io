---
title: "The Microkernel VMM, Done Right"
date: 2026-08-24
slug: microkernel-vmm-done-right
tags: ["virtualization", "microkernel", "VMM"]
description: "In 2005 the Xen team asked whether VMMs are microkernels done
  right; over twenty years isolation went to microVMs, the dataplane to
  silicon, discipline to DPUs, and only 'the service stack restarts
  painlessly' went unclaimed — this article traces that history and its
  four obstacles, then offers a pure-software assembly: a thin core
  defined as firmware, stateless service VMs, a dataplane delegated to
  silicon, holding with VM boundaries and the IOMMU the boundary the DPU
  holds with PCIe."
---

## 1. The Estate, Divided

### 1.1 The Promise from Twenty Years Ago

Upgrading the virtualization stack is one of the oldest
pains in cloud operations: the dataplane and the device
models live in the host's kernel and userspace, and the host
has never been able to answer one question — where exactly
should the firmware/software boundary be drawn? In today's
host almost everything is software, so almost every upgrade
can only be a machine-level event: a kernel upgrade means a
host reboot, and reboots bring migrations and maintenance
windows.

In 2005 the Xen team published at HotOS a paper whose title
was the question:
[Are Virtual Machine Monitors Microkernels Done Right?](https://www.usenix.org/event/hotos05/final_papers/full_papers/hand/hand.pdf)
The yardstick of the question is the microkernel's lifelong
pursuit: a tiny privileged layer, services in isolated
domains, fault domains independent of one another. The
question asks: is the VMM the carrier that finally gets
these three done? If they are done, the service stack no
longer shares the host's fate — and the opening question
gets an architectural answer.

The paper's answer is yes, and the argument is a necessity
claim: of the VMM's duties, what must reside in the highest
privilege layer is naturally a tiny sliver — CPU/memory
virtualization and isolation itself; boundaries between
guests are enforced by hardware; device drivers, device
models, and the control plane need none of them live in the
privilege layer, and in principle all can be pushed into
ordinary domains.

But this yes redeemed only the shape. The isolation half
holds naturally; the other half — services living in
ordinary domains should be restartable and upgradable like
ordinary processes — and the paper's own Xen was the first
to try. The VEE'08
[Improving Xen Security through Disaggregation](https://www.cl.cam.ac.uk/research/srg/netos/papers/2008-murray2008improving.pdf)
opened the driver-domain route; the SOSP'11
[Breaking Up is Hard to Do](https://dl.acm.org/doi/pdf/10.1145/2043556.2043575)
(Xoar) broke dom0 into a set of minimally privileged service
domains, each guest's device model running alone in a
stubdom; on paper, the architectural loop was closed.

But deployment history answered with a different word.
Driver domains and stubdoms entered the mainline, yet never
entered production clouds: Citrix's
[Windsor](https://www.slideshare.net/slideshow/windsor-domain-0-disaggregation-for-xenserver-and-xcp/14123110)
wanted this split and died quietly; the knot was xenstore —
this global registry stayed welded inside dom0, every
guest's device tree, configuration, and credentials hanging
off it, and restarting it means rebuilding the whole
machine's guest control plane; thus a dom0 restart is
operationally equivalent to a host reboot. The pure-software
redemption at cloud scale remains suspended to this day.

So service-stack upgrades have no free option: what lives in
userspace restarts in place, at the cost of one stall for
tenants; what lives in the kernel can only ride the host
reboot — tenant migration or outage. The migrations and
maintenance windows spoken of at the opening come precisely
from this. What this failure shows is not that the
architecture was wrong, but that it could not withstand
production. Why it could not — the four obstacles of
chapter 2 are the answer.

### 1.2 The Industry's Actual Answers

The cloud vendors spent fifteen years each writing their own
answer — but they were not answering the same question.

**AWS: move dom0 into hardware.** First it fled Xen for a
minimal KVM-lineage VMM, then moved the entire
infrastructure stack onto the
[Nitro card](https://www.theregister.com/off-prem/2017/11/29/amazon-reveals-nitro-custom-asics-and-boxes-that-do-grunt-work-so-ec2-hosts-can-just-run-instances/802491).
The promise's "isolatable, restartable service domains" were
redeemed — only the carrier changed from process domains to
PCIe endpoints. The architecture is right; the placement is
silicon.

**Microsoft: dataplane down, control plane stays.** From the
early
[Catapult](https://www.microsoft.com/en-us/research/project/project-catapult/)
FPGA SmartNICs to
[AccelNet](https://learn.microsoft.com/en-us/azure/virtual-network/accelerated-networking-overview):
network policy into FPGA and NIC silicon, host software only
the control plane. Half a microkernel solution: the
dataplane got a life of its own; the service stack did not.

**Google: ran in software for a decade first.** Andromeda's
VFP — a packet-processing engine in the host kernel — proved
a software dataplane can carry a hyperscale cloud's VPC: the
dataplane need not be born in silicon. But in the end Google
too co-developed an
[IPU](https://www.reuters.com/technology/intel-google-cloud-launch-new-chip-improve-data-center-performance-2022-10-11/)
with Intel — the staunchest standard-bearer of the software
route moved both the network and the block-storage dataplane
off the host — this offload silicon Google calls
[Titanium](https://cloud.google.com/blog/products/compute/titanium-underpins-googles-workload-optimized-infrastructure)
externally.

**The KVM ecosystem: gradual peeling, not structural
disaggregation.** vhost moved the dataplane out of QEMU's
device model, landing in the kernel;
[vhost-user](https://www.qemu.org/docs/master/interop/vhost-user.html)
moved it into an independent userspace daemon, restartable
on its own; vDPA then handed it to hardware. Three landings,
one constant direction: QEMU's device model hollowed out
piece by piece. The microVM school
([Firecracker](https://www.usenix.org/conference/nsdi20/presentation/agache),
Cloud Hypervisor, rust-vmm) trimmed the VMM itself to a
minimum and rewrote it in Rust — but what it trims is the
VMM, not the service stack: storage, networking, and the
control plane still share the host's fate. Isolation won;
restartability did not.

Four answers, answering isolation, dataplane, attack
surface; the only one that touched restartability, AWS,
locked its answer in silicon that is not for sale. In pure
software, none answered the original question: **can the
firmware/software boundary be drawn thin enough — can
everything above the firmware layer be upgraded and
restarted in place?**

Looking back, the promise was redeemed piecemeal: isolation
went to microVMs — Firecracker won serverless; the dataplane
went to silicon — eswitch flow tables and offload engines of
all kinds, present in ordinary NICs too; the on-card
general-purpose compute that truly separates DPUs from
ordinary NICs took the last piece: discipline. Only the
plainest clause, "the service stack restarts painlessly,"
has no home in pure software.

This is exactly the most substantive residue left by the
[previous article](/en/p/end-of-dpu-myth/)'s audit of DPUs:
an independently upgradable and restartable root compute
domain — engineering discipline frozen into hardware, at too
high a price in hardware. This article asks the other half:
**can the same boundary be held in pure software?**

It can. And every part of the idea is already in service as
someone's equipment — what is missing is only someone to
assemble them into that machine.

## 2. The Four Gravities

The promise's redemption in pure software has been suspended
for twenty years, not because no one thought of it. The
difficulty lies in four layers, like four gravities pulling
the architecture back toward the monolith: the first three
are engineering problems, the last an organizational one.

**The mechanisms exist; the orchestration is missing.**
Restart semantics were never absent: the PV
(paravirtualized) frontend/backend protocol — guest frontend
and service-domain backend talking over shared-memory rings
— leaves a reconnect exit in its state machine; vhost-user
made reconnect an explicit feature; live migration performs
the existence proof daily — a guest moves to another machine
and re-establishes every backend connection under that
machine's dom0. "Replace dom0" needs no new mechanism, only
tearing down and rebuilding the whole web of relationships
among guests, backends, and the registry in dependency
order. The difficulty is the web: xenstore is its hub,
backends depend on one another, and the ordering of teardown
and rebuild is itself the problem. No ecosystem has ever
productized this step.

**The bootstrap loop.** dom0 holds the driver of the device
its own rootfs sits on. Restarting it requires handing
hardware ownership away first — storage, network, console —
yet the domains taking over that ownership must themselves
be launched and managed by dom0. Driver-domaining is the
precondition of bootstrapping; bootstrapping is where the
driver domains live. Xoar tore down the serving side; the
bootstrapping side stayed where it was.

**The gravity of performance.** Every extra domain boundary
costs one more notification and one more copy. The gravity
of engineering optimization always points at taking the
dataplane back: back into the kernel, back into the same
process, back into the same silicon. vhost was one such
retrieval; SR-IOV a larger one. Isolation and performance
renegotiate on every generation of hardware, and performance
almost always wins — so holding the boundary is not enough;
holding it must cost so little that performance has no
motive to defect.

**The organizational asymmetry.** The benefit of dismantling
dom0 diffuses across all future upgrades — every upgrade
becomes a background operation instead of a fleet
maintenance window; the risk concentrates on the proposer's
incident rate this one time. No release manager wants to
take this year's outage in exchange for "next year's
operations will be smoother." The first three obstacles have
technical solutions; the fourth does not — it can only be
absorbed by design: make the assembly cheap enough, and each
restart's blast radius small enough.

## 3. Assembling the Machine

The goal is only one: **draw the firmware/software boundary
as thin as possible — the thin core is the system's
firmware, the equivalent of an in-house DPU's hardened
logic: not part of the software, not a hot-upgrade target,
evolving by versioned releases; everything above the
firmware layer — the service stack and the control plane —
can be upgraded and restarted at any time, without rebooting
the host and without migration.**

Two disciplines follow:

- **The core is firmware, not software.** The thin core is
  the equivalent of an in-house DPU's hardened logic: not a
  hot-upgrade target, evolving by versioned releases, rare
  events on the scale of years; CVEs between releases are
  absorbed by livepatch as a safety valve. It owns only what
  is strictly its own — the smaller it is, the fewer the
  releases.
- **Service domains must be stateless.** State dies wherever
  it lives at restart time — so all state is externalized,
  and nothing that cannot be lost is kept inside a service
  domain.

### 3.1 The Architecture: Three Layers

The architecture has three layers, as the figure shows: at
the bottom, the thin core spans the whole node — trimmed
Linux + KVM, holding only CPU/memory virtualization, IOMMU,
interrupt injection, and boot media; above it, two zones
side by side — the host zone houses the control plane: a
special VM carries the body, while a zero-state daemon in
thin-core userspace executes operations and writes flow
tables; the guests zone houses the function-split service
VMs and the user VMs they serve. Every VM gets one
passthrough VF for its own connectivity; the tenant
dataplane goes through VF passthrough, never through any
service VM; cache is divided by partition, host services and
tenants each in their own. The paragraphs below unfold each
in turn.

![System structure: the host zone (control-plane VM, control-plane daemon) and the guests zone (user VMs, service VMs), two rows each, horizontally aligned, sit atop the full-width thin core; the hardware layer is at the bottom; the tenant dataplane is VF passthrough](arch.en.svg)

**The thin core.** Trimmed Linux + KVM, owning only
CPU/memory virtualization, IOMMU, interrupt injection, and
boot media — no dataplane, no control plane. This layer is
the system's firmware: the equivalent of an in-house DPU's
hardened logic, evolving by versioned releases; KVM itself
is a kernel module, squarely within livepatch's reach — the
smaller the attack surface, the fewer the CVEs and the fewer
the releases, and what slips through between releases is
absorbed by livepatch as a safety valve. What must not be in
the thin core is clearer than what must: no storage drivers,
no device models, no toolstack; on the network side only the
PF driver remains — eswitch management goes to the
control-plane daemon.

**Stateless control plane: a special VM on the host side.**
Registries, configuration, and orchestration state are all
externalized — the sole repository is the cluster-level
control plane; the in-node control-plane VM carries the
control plane's body, zero-state, passively receiving the
cluster's scheduling arrangements, only issuing intent —
landing it is the control-plane daemon's job, and the two
restart independently. The control-plane VM's restart
likewise never stops forwarding — the dataplane lives in
eswitch flow tables, unrelated to it. The xenstore lesson
stands: the global registry must never again be welded into
anything that needs to restart.

**The control-plane daemon.** The NIC's PF and its driver
belong to the thin core; the real control operations the
control-plane VM issues — creating VMs, deleting VMs,
binding and unbinding devices — are landed by this
zero-state process in thin-core userspace; eswitch flow
tables and per-VF policies are also written by it. Its
restart is not the kernel's restart: flow tables live in
silicon, so forwarding never stops during the process's
restart; what pauses is only change. This service does not
live in an isolated domain but as a process beside the thin
core — device ownership (the PF cannot leave the thin core
without an accompanying reset) trumps domain isolation; what
is given up is fault-isolation granularity, what is gained
is that restart and dataplane lifetimes are no longer
entangled.

**Service VMs.** Split by function, not by machine: block
storage and file storage each form their own domain; how
many per domain is set by need — one or many. Each service
VM holds its own hardware via VFIO passthrough, dataplane
and protocol termination running inside; every VM — service
VMs and the control-plane VM alike — also gets one
passthrough VF for its own connectivity. Every service VM is
a dom0 — but a stateless one: a crash counts as a restart,
an upgrade is a restart, both travel the same road. The
finer the split, the smaller the restart blast radius and
the freer the upgrade cadence. Using VMs rather than
processes buys one more thing: restart domain, deployment
domain, and failure domain collapse into a single boundary —
a hardware-enforced one, free of charge.

**Cache partitioning.** Services living on host cores share
L3 with tenants: left unmanaged, service-VM traffic and
daemon scans pollute tenants' cache lines, and this
interference is invisible and unbookable. Today's multi-core
L3s are mostly partitionable: Intel CAT allocates cache ways
per CLOS, AMD's L3 belongs to a CCX so dedicating one CCX is
physical isolation, ARM MPAM partitions by PARTID; Linux's
resctrl unifies the three under one configuration interface.
The response is configuration, not prayer: concentrate the
CPUs used by host services — service VMs and each daemon —
into one or two cache partitions (on AMD, literally one
CCX), tenants use the rest. Pollution thus goes from an
uncontrolled externality to a boundary drawn by
configuration: how much cache services may occupy is written
in config, not in luck.

### 3.2 Networking: The Dataplane Belongs to the NIC

Guests hold SR-IOV VFs; VPC rules are pushed down into the
NIC's eswitch flow tables; per-VF rate limiting and QoS are
standard NIC capabilities. What remains for the control-plane
daemon is only out-of-band work: programming rules,
collecting state.

Thus the control-plane daemon's restart becomes boring:
rules stay in silicon, forwarding never stops; what pauses
in its restart window is only "change" — config pushes,
security-group edits, queued for tens of seconds. Azure's
[Accelerated Networking](https://learn.microsoft.com/en-us/azure/virtual-network/accelerated-networking-overview)
has run this pattern for years: policy in hardware, host
bypassed.

The expressiveness ceiling of eswitch flow tables and
match-action pipelines decides that "dataplane to the NIC"
is forever only partial: what pushes down is the stable and
hot match-action — L2/L3 forwarding, encapsulation, per-VF
rate limiting; what does not is the long tail — stateful
connections, complex NAT and load-balancing semantics, new
protocols. This is not a cost item but a design constraint:
it dictates that the software path is not a transitional
measure but a permanent component, the hardware fast path an
accelerator rather than the sole execution point. The
ceiling itself is unrelated to merchant vs in-house —
in-house DPUs freeze at tape-out too, this being the
embodiment in hardware of chapter 2's "iteration agility"
counter-argument; programmable pipelines (P4-class
match-action, vendors' flow programming APIs) push the
boundary outward, but the boundary does not disappear, and
it moves with someone else's roadmap.

Two points need further explanation:

- **VF migration is now supported.** mlx5's vfio migration
  variant driver is merged into mainline
  ([Add mlx5 live migration driver](https://lwn.net/Articles/873315/)),
  and merchant NIC VFs can migrate from now on; the mature
  production practice is the synthetic fallback path:
  dynamically revoke and restore VFs around maintenance
  events, applications bound to the synthetic device to keep
  connectivity — the official documentation
  [says so explicitly](https://learn.microsoft.com/en-us/azure/virtual-network/accelerated-networking-overview).
  Two caveats: first, what is standardized is the migration
  framework, not the state — the migration data stream is
  vendor-opaque, coverage limited to the same series;
  second, cross-vendor migration has no path — but this is
  not fatal, cross-CPU-model/vendor live migration is
  likewise constrained, and fleets are homogeneous by
  construction.
- **The offload boundary is drawn by others.** Which
  policies push down depends on NIC generation and vendor;
  the two sides of the boundary are heterogeneous — inside
  the card, the vendor's flow API; outside, our software;
  the long tail pays one extra hop when crossing. In-house
  DPUs have ceilings too, but own both sides of the boundary
  and catch the long tail on on-card cores; this design
  trades that ownership for not building silicon.

### 3.3 Storage: Protocol Termination in a Service VM

Guests see only virtio; the storage service VM terminates
the guest's block requests at the front and meets the
storage cluster at the back with a storage protocol — RDMA
or TCP, NVMe-oF or proprietary, deployment-dependent;
credentials and topology invisible to tenants.

Its restart window is the most engineered part of the whole
design, yet needs no redundant instances. In most cases what
restarts/upgrades is the userspace storage service daemon —
process-level restart, seconds or faster; only when the
service VM's kernel itself must be swapped — rare — does
kexec preload the new kernel and compress the restart to
seconds. The stall the guest feels equals the restart
duration; the guest side's absorption capacity is far wider:

- virtio-blk has no hard timeout, and the guest's filesystem
  retries on its own, with an absorption ceiling on the
  order of tens of seconds — this is margin, not expected
  stall;
- on the backend side, the storage protocol's reconnect and
  retry semantics provide the backstop.

Multiple instances exist first for isolation: one instance's
crash or restart touches only its own share; second, to make
restarts easier: rolling per instance, so a single window
need not gamble the whole.

### 3.4 No New Mechanisms, and Every Segment Walked

Nothing in this design is new: the reconnect semantics cited
in chapter 2's first layer, plus VFIO's unbind and rebind,
are the entire inventory the assembly needs. What Xen lacked
was never mechanisms, but turning "replace dom0" into one
orchestrated action. And every segment of this road has been
walked by someone:

- **Nitro** is this architecture — only the service VMs
  moved onto the card. What it proves is that the
  architecture holds at cloud scale; whether the service
  domain lives in host memory or on-card silicon is a
  commercial question, not an architectural one.
- **AccelNet** proves the other half: a hardware dataplane
  plus a software control plane can serve for years.

What no one has walked is assembling them back into pure
software — that is this chapter's work.

## Conclusion

The 2005 question remains open to this day, but the shape of
the answer is already clear. This article's assembly lets
the host answer the opening question: the firmware/software
boundary drawn as thin as possible — everything above the
firmware layer can be upgraded and restarted at any time,
without rebooting the host and without migration, what
tenants feel at most an absorbable stall; the firmware layer
itself, like an in-house DPU's hardened logic, evolves by
versioned releases. The microkernel VMM needs no new
mechanisms; what it lacks is
one serious assembly — and every block the assembly needs
has already run in someone's fleet for years: reconnect
semantics, VFIO, kexec, eswitch flow tables, cache
partitioning — none invented here.

The reasonableness of this architecture lies in making every
boundary explicit and cheap: VM boundaries plus the IOMMU
hold in software the boundary the DPU holds with a single
PCIe bus — hardware-enforced, free of charge; the tax is
booked on fungible host cores, not stranded on-card silicon;
the offload boundary and the cache partitions are written in
configuration, not in vendor roadmaps or luck. What software
buys back is iteration: no tape-out freeze, new semantics
need not wait for the next silicon.

Isolation was never the hard part. Maintenance was — and
what this article gives back to maintenance is its
ordinariness: a crash, an upgrade, both count as one
restart.
