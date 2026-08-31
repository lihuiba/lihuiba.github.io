---
title: "Time to End the DPU Myth"
date: 2026-08-20
slug: end-of-dpu-myth
tags: ["DPU", "cloud", "virtualization", "myth"]
description: "Every large-scale DPU success comes from a hyperscaler's vertically integrated, in-house program, while the merchant DPU market is a complete wreck. An attribution audit of its seven value propositions: five do not survive scrutiny, one is real but software-replaceable, and one is real but worth only half — half a claim left standing out of seven. Thin value wrapped in a thick narrative is the standard recipe for a myth."
---

## 1. A Successful Pattern Doesn't Redeem a Failed Market

The DPU narrative has been told for years: "zero virtualization
tax", "performance gains", "security isolation", "bare-metal
unification", "iteration agility", "energy efficiency", culminating
in its coronation as "the third pillar of computing" after CPU and
GPU. But the market's answer sits awkwardly with this narrative: it
has rewarded the self-builders, but not the buyers. This article audits
the narrative claim by claim — first the market, then the value
propositions, then the costs, and finally what is left.

Every large-scale DPU success story comes from a hyperscaler's
in-house, vertically integrated program: AWS acquired
[Annapurna Labs](https://en.wikipedia.org/wiki/Annapurna_Labs)
in 2015, and the Nitro system
[made its full debut](https://www.theregister.com/off-prem/2017/11/29/amazon-reveals-nitro-custom-asics-and-boxes-that-do-grunt-work-so-ec2-hosts-can-just-run-instances/802491)
with C5 instances in 2017;
Alibaba Cloud unveiled its
[X-Dragon architecture](https://www.alibabacloud.com/blog/alibaba-cloud-announces-3rd-generation-of-x-dragon-architecture_595397)
in 2017 and
launched [CIPU](https://www.cs.com.cn/ssgs/gsxw/202206/t20220614_6276903.html)
in 2022; Microsoft started with its in-house
[Catapult FPGA SmartNIC](https://www.microsoft.com/en-us/research/project/project-catapult/),
[acquired Fungible's team](https://blogs.microsoft.com/blog/2023/01/09/microsoft-announces-acquisition-of-fungible-to-accelerate-datacenter-innovation/)
in 2023, and shipped the in-house
[Azure Boost DPU](https://www.infoq.com/news/2024/12/azure-boost-dpu-inhouse-chips/)
in 2024. Huawei's QingTian, Baidu's
[Taihang](https://news.sina.cn/sx/2022-12-27/detail-imxycmna3727168.d.html?vt=4),
and [Tencent](https://cloud.tencent.com/developer/article/2105157)
all follow the same in-house path. The only top player that comes close to
"procurement" is Google, which co-designed
[Mount Evans](https://www.reuters.com/technology/intel-google-cloud-launch-new-chip-improve-data-center-performance-2022-10-11/)
with Intel — but that is custom silicon built to Google's spec,
not an off-the-shelf purchase.

The merchant DPU market, meanwhile, is a complete wreck. Fungible
raised over $300M, then
[was acquired by Microsoft](https://blogs.microsoft.com/blog/2023/01/09/microsoft-announces-acquisition-of-fungible-to-accelerate-datacenter-innovation/)
in early 2023 — reportedly
[for only about $190M](https://www.blocksandfiles.com/composable/2022/12/13/fungible-sold-to-microsoft-for-190-million-say-multiple-sources/1596153).
Pensando was
[acquired by AMD](https://ir.amd.com/news-events/press-releases/detail/1057/amd-expands-data-center-solutions-capabilities-with-acquisition-of-pensando)
in 2022 and has faded from view since. Nebulon quietly folded,
with its team
[reportedly absorbed by NVIDIA](https://www.blocksandfiles.com/architecture/2024/02/13/nebulon-blink-twice-if-nvidia-really-has-absorbed-you/1586493).
Intel's IPU product line has been
[reworked repeatedly](https://www.eenewseurope.com/en/intel-muddies-the-data-centre-waters-with-ipu-roadmap/).
NVIDIA BlueField's public deployments concentrate in niches such as
AI networking. For the enterprise market,
[vSphere DSE](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/esx-installation-and-setup/introducing-vmware-vsphere-distributed-services-engine-and-networking-acceleration-by-using-dpus-install.html)
(vSphere on DPUs) never saw adoption at scale — and its DPU firmware
update requires a host reboot, with vMotion evacuating the VMs to
keep them alive. Replacing seamless upgrade with evacuation is
itself the most honest vote.

Chinese independent DPU vendors are a copy of the same verdict,
only starker. The furthest along, JaguarMicro, is now rushing
toward a Hong Kong IPO as "China's first DPU stock" — yet
[its prospectus shows](https://finance.sina.cn/stock/ssgs/2026-07-26/detail-inikcpvy8685542.d.html)
three-year revenue growing from ¥170K to ¥370M against cumulative
losses of nearly ¥2.5B, with
[over 90% of revenue from a single customer: Tencent](https://m.21jingji.com/article/20260718/66b8010480dc12022090eb0e122ddabf_zaker.html)
— which is also its largest shareholder, and which builds its own
DPU anyway. Largest shareholder plus 90% of revenue: that is not an
independent vendor, it is one cloud's department in all but name. As for the
"truly independent" rest: Zuojiang's DPU business was found by
regulators to be "severely misrepresented" —
[confirmed financial fraud, delisting, and its controller detained](https://xinwen.bjd.com.cn/content/s69709d9ce4b0cd719e9c7d2b.html).
[YUSUR](https://news.pedaily.cn/202608/567821.shtml)
is still living on venture rounds, its latest C+ round at a
[pre-money valuation of about ¥5B](https://view.inews.qq.com/a/20260813A0BCOG00);
its market has narrowed to financial low-latency trading — which is
no longer cloud at all — while on the private-cloud side it has
only project-scale cases like a
[bank's cloud platform](https://www.yusur.tech/case/cloudPlatform)
and a
[university edge cloud](https://www.yusur.tech/case/computingNetworkIntegration),
with no access to the public-cloud mainstream; publicly it
offers only
[unverifiable claims like "orders doubled"](https://t.cj.sina.cn/articles/view/1650111241/625ab3090200152jr),
and has never filed a prospectus — the silence about its financials
is itself an answer. NebulaMatrix, YunSilicon, Dayu and the other
startups from the same cohort have gone quiet. In other words, no
independent Chinese DPU vendor has made it — the only one to reach
the IPO door got there precisely by ceasing to be independent.

If the DPU's technical value were substantive, it would have
rewarded buyers as well as self-builders; in reality it has rewarded
only self-builders. That means its margin is so thin that it turns
positive only once all vendor margin is stripped out, at BOM cost.
A successful pattern cannot excuse a failed market — on the
contrary, the split itself is the market's verdict.

## 2. An Attribution Audit of the Value Propositions

The DPU narrative consists of seven value propositions. Audited one
by one, five do not survive.

**"Zero virtualization tax" — relocation, not elimination.** The
virtualization stack does consume some fraction of host compute —
the so-called "virtualization tax". But a DPU merely moves that tax
from host cores to card cores, running the same protocol stack and
the same device models; not one line of work disappears, only the
ledger changes — from sellable host CPU to a one-time sunk card
cost. That is at best a currency conversion, and what you convert
into is stranded capital: host cores not doing infrastructure
can still run tenant workloads, while card silicon can only
ever run infrastructure.

**"Performance gains" — the hardware pipeline is not the DPU's.**
Line-rate small-packet forwarding is indeed hardware territory;
software suits only low-speed virtual networks. But the hardware
pipeline exists in ordinary SR-IOV NICs. What a DPU actually adds —
its programmable part — runs on weak ARM or RISC-V cores that are
slower than the host CPU. Beneath the "hardware offload" packaging, it is
the same software deployed onto the card — it even relies on
SR-IOV, enumerating VFs exactly the way an ordinary NIC does. The
accurate translation of "hardware offload" is: the same software
moved onto weaker hardware. And that weakness only becomes more of
a bottleneck as link speeds march into the hundreds of Gbps.

**"Security isolation" — the boundary is one-way.** Virtio is just
a pair of shared-memory ring queues; security depends on the device
implementation, not on where it runs. Card firmware is
closed-source, less audited, and slower to patch, while the QEMU
device model it replaces is
[continuously fuzzed in public](https://www.qemu.org/docs/master/devel/testing/fuzzing.html).
More critical is DMA: as a PCIe device the card inherently holds
access to host memory, and on bare metal the IOMMU cannot trim it
— compromising the card means compromising the host. The DPU's
security boundary stops the tenant from touching the platform;
it never stops the card from harming the tenant.

**"Bare-metal unification" — real, but worth only half a claim.**
When the host is untrusted, cloud-disk and cloud-network protocols
must be terminated on a card — this is the DPU's hardest-to-replace
function. But it is being eroded from both ends. Cardless bare
metal (switch-scoped VXLANs, iSCSI/NVMe-oF remote boot) is a
legitimate, merely thinner product, as SoftLayer and Equinix Metal
proved long ago. And for customers demanding full VPC/EBS
semantics, a VM occupying (nearly) the whole host approaches
bare-metal performance while inheriting all cloud semantics. The
true irreducible demand shrinks to three cases: bring-your-own
hypervisor stacks, licenses bound to physical hardware, and
compliance clauses that literally mandate dedicated hardware.

**"Iteration agility" — actually the opposite.** Pipeline features
on the card are frozen at tape-out; new protocols wait for the next
silicon generation; firmware and drivers upgrade in lockstep. The
DPU does not decouple software from hardware — it couples iteration
to hardware generations. Meanwhile, upgrading the dataplane without
disturbing tenants is something host software does just as well.

**"The third pillar of computing" — negated by its own name.** DPU
is a rebranding NVIDIA pushed after acquiring Mellanox. The only
defensible reading is "a fixed menu of domain-specific
acceleration" — and domain-specific self-refutes "general".

**"Energy efficiency" — the ledger is bankrupt.** The perf-per-watt
of modern x86 and ARM host CPUs voids the premise that the card's
ARM cores save power; the card itself draws
[75W+](https://networking-docs.nvidia.com/bf3dpu/specifications)
against roughly 25W for an ordinary NIC — a net increase in power.

## 3. But What's the Cost?

Even if you doubt the entire audit above, the DPU's cost list
stands on its own, and it is hard:

**Hardware is irreversible.** Once taped out, nothing can be
changed: how many I/O queues, how many VFs — fixed at manufacture;
any design flaw ships as an erratum and stays for the product's
whole generation. A software defect is a hotfix; a hardware defect
is a generation.

**One more failure domain per server.** Merchant DPUs generally run
a full Linux distribution on-card — kernel CVEs and firmware
updates are all still there, just moved onto a device with worse
tooling and a slower release cadence. A card-firmware bug is a
fleet-wide correlated event — the failure domains multiply
exactly with the fleet.

**Creating a single point of failure on bare metal.** With cardless
bare metal, cloud-side maintenance is transparent to the tenant:
switches are redundant, storage is multipathed, upgrades never
touch the tenant's machine. The DPU inserts the only
cloud-controlled failure domain that needs recurring maintenance
into the tenant's dedicated chassis — and bare metal has no live
migration as a fallback, so a card firmware reboot lands directly
on the tenant's SLA. "Seamless restart" is supposed to be the
answer, yet its delivery is: real inside two or three closed
hyperscaler fleets, unauditable; slideware in the open market; and
on bare metal — where it is needed most — backed by no public data
at all.

**Organizational costs are systematically undercounted.** Entire
silicon and firmware teams, ASIC tape-outs, fleet incidents — these
costs are real, yet no public ROI story has ever disclosed them,
while the benefits can be booked precisely at vCPU prices. More
subtly, the card's existence freezes the basis of comparison: the
pure-software alternative was never built, the controlled
experiment can never be run, and sunk NRE makes "keep funding the
card" look rational forever.

## 4. What Remains

At the end of the audit, one genuine thing remains: **an
independently upgradable and restartable root compute domain** —
the infrastructure service stack gains a lifecycle independent of
both the host kernel and the tenants, and its upgrades and restarts
disturb no one. This is the most substantive residue after all
seven claims are sifted, and it is the twenty-year-old promise of
microkernel architecture: the title of the Xen team's 2005 paper,
[Are Virtual Machine Monitors Microkernels Done Right?](https://www.usenix.org/event/hotos05/final_papers/full_papers/hand/hand.pdf),
was a question — deployment history answered "no", as dom0 grew
irresistibly into a monolith and could not resist architectural
rot. But Xen's failure to build it does not mean software cannot.

With one PCIe bus, the DPU turns a boundary that software
discipline could not hold into one that is physically uncrossable.
That is its most respectable technical footnote: **engineering
discipline frozen in hardware** — organizational value, not
technical value. It is just that the price of achieving this with
hardware is too high.

## 5. Conclusion

The full structure of the DPU myth: seven value propositions, five
fail the attribution audit; one is real but software-replaceable —
and replaceable means it does not stand; half a claim is real but
niche. Seven claims, half left standing, and behind them a
non-trivial bill of costs. Its success belongs to two or three
hyperscalers in a particular historical window — as much chance
as design; its failure belongs to the whole market's open
appraisal of its technical value — and the latter is the truth of
pricing. "Everyone is using it" was never evidence of technical
value; it may only be evidence of historical decisions,
organizational inertia, and a signaling equilibrium.

Thin value wrapped in a thick narrative is the standard recipe for
a myth. And this myth should end.

-------------------

## Suppplement on 2026.08.31
**In-house cards make VM migration easy** — and for a long time
that was true. The obstacle was never "in-house versus merchant";
it is whether device state can stay consistent across migration:
with bare VF passthrough, queue contexts live inside the NIC,
and exporting them completely and restoring them on the new
host's card was beyond what merchant hardware could do — high
performance and migratability thus did not go together. The
in-house solution was to have a program hold the state: the
guest's device is provided by firmware on the card (ena under
Nitro, virtio under X-Dragon), and at migration time the new
host's card simply instantiates it again, and the guest notices
nothing. And it need not be a DPU: any card that can impersonate
the guest's device will do. For years only the in-house builders
could walk this path.

That page has turned, on both roads. Merchant cards learned to
impersonate early — mlx5's vDPA mode
([VDPA support for Mellanox ConnectX devices](https://lwn.net/Articles/828042/))
has presented virtio devices to guests since 2020, with the data
path in hardware; and for those who stayed with bare VF
passthrough, the mlx5 vfio migration variant driver landed in the
mainline kernel in 2022
([Add mlx5 live migration driver](https://lwn.net/Articles/873315/)),
and NVIDIA's official documentation now provides full
[SR-IOV Live Migration](https://networking-docs.nvidia.com/mlnxofedswum/24.10-5.1.6.1lts/sr-iov-live-migration)
support — with state export and restore in place, merchant NIC
VFs can migrate too. **This once-exclusive
value has been worn flat by the merchant path — what remains is
the head start and roadmap ownership: procurement, not
technology.**
