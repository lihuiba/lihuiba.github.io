---
title: "DPU 的神话该结束了"
date: 2026-08-20
slug: end-of-dpu-myth
tags: ["DPU", "云计算", "虚拟化", "神话"]
description: "DPU 的成功部署全部来自头部云厂商的自研垂直整合，而商用 DPU 市场已经全军覆没。逐项审计其七项价值主张：五项经不起归因审查，一项真实但软件可替代，半项真实但小众——七项只剩半项。薄收益加厚叙事，正是神话的标准配方。"
---

## 一、成功的模式不抵失败的市场

DPU 的叙事讲了很多年："虚拟化税归零"、
"性能提升"、"安全隔离"、"裸金属统一"、
"迭代敏捷"、"能效"，直至加冕为继 CPU、GPU
之后的"第三颗算力"。但市场给出的答案与这套
叙事格格不入：它成就了自研者，却没有成就
买家。本文逐项审查这套叙事——先看市场，
再拆主张，再算代价，最后看还剩下什么。

DPU 的大规模成功部署，全部来自头部云厂商的自研垂直整合：AWS 2015 年收购
[Annapurna Labs](https://en.wikipedia.org/wiki/Annapurna_Labs)，
Nitro 系统 2017 年随 C5 实例
[完整亮相](https://www.theregister.com/off-prem/2017/11/29/amazon-reveals-nitro-custom-asics-and-boxes-that-do-grunt-work-so-ec2-hosts-can-just-run-instances/802491)；
阿里云 2017 年发布
[神龙架构](https://baike.baidu.com/item/%E7%A5%9E%E9%BE%99%E6%9E%B6%E6%9E%84/59914172)，
2022 年推出
[CIPU](https://www.cs.com.cn/ssgs/gsxw/202206/t20220614_6276903.html)；
微软从自研
[Catapult FPGA SmartNIC](https://www.microsoft.com/en-us/research/project/project-catapult/)
起步，2023 年
[收购 Fungible 团队](https://blogs.microsoft.com/blog/2023/01/09/microsoft-announces-acquisition-of-fungible-to-accelerate-datacenter-innovation/)，
2024 年推出自研
[Azure Boost DPU](https://www.infoq.com/news/2024/12/azure-boost-dpu-inhouse-chips/)；
华为擎天、
[百度太行](https://news.sina.cn/sx/2022-12-27/detail-imxycmna3727168.d.html?vt=4)、
[腾讯](https://cloud.tencent.com/developer/article/2105157)
同样是自研路线。头部厂商中唯一接近"采购"的是 Google——与 Intel 联合设计
[Mount Evans](https://www.reuters.com/technology/intel-google-cloud-launch-new-chip-improve-data-center-performance-2022-10-11/)，
但那也是按 Google 规格定制的 co-design，不是购买货架产品。

然而商用 DPU 市场已经全军覆没：Fungible 累计融资超过三亿美元，2023 年初
[被微软收购](https://blogs.microsoft.com/blog/2023/01/09/microsoft-announces-acquisition-of-fungible-to-accelerate-datacenter-innovation/)，
据报道
[作价仅约 1.9 亿美元](https://www.blocksandfiles.com/composable/2022/12/13/fungible-sold-to-microsoft-for-190-million-say-multiple-sources/1596153)；
Pensando 2022 年
[被 AMD 收购](https://ir.amd.com/news-events/press-releases/detail/1057/amd-expands-data-center-solutions-capabilities-with-acquisition-of-pensando)
后逐渐边缘化；Nebulon 悄然收场，
[据报道团队被 NVIDIA 吸收](https://www.blocksandfiles.com/architecture/2024/02/13/nebulon-blink-twice-if-nvidia-really-has-absorbed-you/1586493)；
Intel IPU 产品线
[多次调整](https://www.eenewseurope.com/en/intel-muddies-the-data-centre-waters-with-ipu-roadmap/)；
NVIDIA BlueField 的公开落地集中在 AI 网络等特定场景。面向企业市场的
[vSphere DSE](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/esx-installation-and-setup/introducing-vmware-vsphere-distributed-services-engine-and-networking-acceleration-by-using-dpus-install.html)（DPU 版 vSphere）未见规模采用，而其 DPU
固件升级需要宿主重启、靠 vMotion 疏散虚拟机来保证业务不中断——用迁移代替无感升级，
这本身就是最诚实的投票。

国内独立 DPU 厂商的处境是同一份判决书的副本，而且更彻底。跑得最远的云豹智能正在冲刺港股
"国产 DPU 第一股"，但
[招股书显示](https://finance.sina.cn/stock/ssgs/2026-07-26/detail-inikcpvy8685542.d.html)
三年营收从 17 万元涨到 3.7 亿元，累计亏损近 25 亿元，
[九成以上收入来自单一客户腾讯](https://m.21jingji.com/article/20260718/66b8010480dc12022090eb0e122ddabf_zaker.html)——
腾讯同时是第一大股东，而这朵云自己同样自研 DPU。第一大股东加九成营收，这不是独立厂商，
是一朵云的体外部门。再看其余"真正独立"的厂商：左江科技的 DPU 业务被监管认定"严重不实"，
[财务造假坐实，公司退市，实控人被拘留](https://xinwen.bjd.com.cn/content/s69709d9ce4b0cd719e9c7d2b.html)；
[中科驭数](https://news.pedaily.cn/202608/567821.shtml)
仍在一级市场融资续命，最新 C+ 轮
[投前估值约 50 亿元](https://view.inews.qq.com/a/20260813A0BCOG00)；
赛道已收窄到金融低时延交易等细分场景——那已经不是云；私有云侧只有
[银行云平台](https://www.yusur.tech/case/cloudPlatform)、
[高校边缘云](https://www.yusur.tech/case/computingNetworkIntegration)
这类项目制的零星落地，公有云正面战场完全无缘；对外只有
["订单量翻倍"的官方口径](https://t.cj.sina.cn/articles/view/1650111241/625ab3090200152jr)，
从未递交招股书——财务不公开，沉默本身也是一种回答；星云智联、云脉芯联、大禹
智芯等同期创业公司已鲜有声响。换言之，国内没有任何一家跑通的独立 DPU 厂商——
唯一跑到 IPO 门口的那家，恰恰是用"不再独立"换来的。

如果 DPU 的技术价值是实质性的，它应该同时成就采购者和自研者；现实是它只成就了自研者。
这说明它的收益薄到只有在剥掉全部 vendor margin、按 BOM 成本自研时才可能翻正。成功
的模式不能为失败的市场开脱——恰恰相反，分裂本身就是市场给出的判决书。

## 二、价值主张的归因审计

DPU 的叙事由七项价值主张构成。逐项归因之后，五项经不起审查。

**"虚拟化税归零"——搬家，不是消灭。** 虚拟化栈确实需要消耗一定量的算力，
即所谓"虚拟化税"。但 DPU 只是把这笔税从宿主的核搬到卡上的核跑，运行的
还是同一份协议栈、同一个设备模型，工作一行没少，只是换了个地方记账——
从可售的宿主 CPU 资源改记为一次性沉没的卡成本。这充其量是一次换汇，而且
换来的是搁浅资产：宿主核不干基础设施时还能跑租户负载，卡硅永远只能干基础设施。

**"性能提升"——硬件管线不在 DPU 手里。** 小包线速确实是硬件的领地，软件只
适用于低速网络；但硬件转发管线在普通 SR-IOV 网卡里就有。DPU 真正多出来的
可编程部分跑在孱弱的 ARM 或 RISC-V 核上，性能反而不如宿主 CPU。
所谓"硬件卸载"外衣之下，只是把一样的软件部署到卡上跑而已——它连 SR-IOV 都依赖，
枚举 VF 的方式和普通网卡一模一样。"硬件卸载"的准确翻译是：同样的软件搬家到了另
一种孱弱的硬件上而已。而这种孱弱在网速迈向
数百 Gbps 之后，只会愈加成为瓶颈。

**"安全隔离"——边界是单向的。** virtio 就是一对基于
共享内存的 ring queue，安全与否取决于设备实现，
不取决于跑在哪里。卡固件闭源、审计者少、
补丁链条长，而被它替代的 QEMU 设备模型
[常年接受公开 fuzz](https://www.qemu.org/docs/master/devel/testing/fuzzing.html)。
更关键的是 DMA：卡作为 PCIe 设备天然持有对宿主
内存的访问能力，裸金属场景下 IOMMU 无从裁剪——
卡被攻破等于宿主被攻破。DPU 的安全边界防的是
租户碰平台，从来不防卡自身殃及租户。

**"裸金属统一"——真实，但只值半项。** 宿主
不可信时，云盘和云网络的协议必须有卡来终结，
这是 DPU 最难替代的功能。但两端都在侵蚀它：无卡
裸金属（交换机划 VXLAN + iSCSI/NVMe-oF 远程
启动）是合法的、只是更瘦的产品，SoftLayer 和
Equinix Metal 早已证明；而对要求 VPC/EBS 完整
语义的客户，一台（几乎）占满整机的 VM 在性能上
逼近裸金属，且天然继承全部云语义。真正的刚需
只剩三类：自带 hypervisor 的栈、绑定物理硬件的
license、写明"专用硬件"的合规条款。

**"迭代敏捷"——说反了。** 卡上 pipeline 功能在
tape-out 时定型，新协议要等下一代设计，固件驱动
锁步升级。不是 DPU 把软硬件解耦，是它把迭代和
硬件代际耦合。而升级数据面不惊扰租户，宿主软件同
样做得到。

**"第三颗算力"——名字即否定。** DPU 是 NVIDIA
收购 Mellanox 后的 rebranding。可辩护的部分只剩
"固定菜单的领域专用加速"，而领域专用恰恰自我
否定了"通用"。

**"能效"——账本已破产。** 现代 x86 / ARM 宿主 CPU 的
perf/watt 让 DPU 上 ARM 核省电的前提不再成立；卡自身
[75W+](https://networking-docs.nvidia.com/bf3dpu/specifications)
对普通网卡约 25W，是净增功耗。

## 三、但是，代价呢？

即使对上述审计全部存疑，DPU 的代价清单也是
独立的、硬性的：

**硬件不可逆。** 流片定型即不可更改：有多少 I/O
queue、支持多少 VF，在出厂那一刻就已固定；
有没有设计缺陷，则以 errata 的形式终身伴随
一代产品。而软件方案的缺陷则是一次
hotfix。

**每台服务器新增一个故障域。** 商用 DPU 普遍在
卡上跑着完整的 Linux 发行版，内核 CVE、固件升级
一样不少，只是从宿主挪到了工具链更差、发布节奏
更慢的设备上。卡固件 bug 是全舰队相关事件——
规模多大，故障域就有多少。

**在裸金属上制造单点。** 无卡裸金属的云侧维护对
租户是透明的：交换机有冗余，存储有多路径，
升级不碰租户的机器。DPU 把唯一一个由云控制、
需要反复维护的故障域塞进了租户独占的机箱——
而裸金属没有热迁移兜底，卡的固件重启直接砸在
租户 SLA 上。"无痛重启"本应是答案，其兑现范围
却恰好是：头部两三家闭园系统里为真且无法审计，
公开市场停留在幻灯片，在最需要它的裸金属上
拿不出任何公开数据。

**组织成本被系统性低估。** 整建制的芯片与固件团队、
ASIC 流片、舰队事故——这些成本真实存在，
但任何对外讲述的 ROI 都从未披露过它们；
收益却可以按 vCPU 售价精确入账。更微妙的是，
卡的存在冻结了比较基准——纯软件的替代架构
从未被建造，对照实验永远无法成立，沉没的 NRE
让"继续投卡"永远显得划算。

## 四、还剩什么

审计到最后，DPU 还剩下一样真实的东西：**一个
可独立升级和重启的 root 计算域**——基础设施
服务栈拥有独立于宿主内核和租户的生命周期，
它的升级和重启不惊扰任何人。这是七项主张过筛之后
最有实质的残值，也是微内核架构二十年前的许诺：
2005 年 Xen 团队的论文
[Are Virtual Machine Monitors Microkernels Done Right?](https://www.usenix.org/event/hotos05/final_papers/full_papers/hand/hand.pdf)
是个问句，部署史替它回答了"no"——dom0 不可
阻挡地长成单体，没能抵挡住架构腐化。但 Xen 没
做好不代表软件上做不到。

DPU 用一条 PCIe 总线，把软件层面守不住的边界
变成物理上不可跨越的。这是它最体面的技术注脚：
**用硬件形式固化的工程纪律**——组织价值，
不是技术价值。只是用硬件达成这份价值的代价太高了。

## 五、结论

DPU 神话的完整结构是：七项价值主张，五项经不起
归因审查，一项真实但软件可替代——可替代就是
立不住——剩下半项真实但小众。七项主张，
最后只剩半项，而其背后则是一份不短的代价清单。它的成功属于
两三家超大云在特定历史窗口里的偶然与必然，
它的失败属于整个市场对其技术价值的公开评判——
而后者才是定价的真相。"大家都在用"从来不是技术价值的
证据，它也可能只是历史决策、组织惯性和信号均衡的证据。

薄收益加厚叙事，是神话的标准配方。而这个神话该结束了。

--------------------------------

## 2026.08.31 补充

**自研卡便于虚拟机迁移**——这件事在相当长的
时间里是真的。障碍从来不在"自研还是商用"，
而在设备状态能否随迁移保持一致：裸 VF 直通
时，队列上下文驻在网卡内部，要把它完整导出、
在新宿主的卡上原样恢复，商用硬件当年做不到
——高性能与可迁移于是不可兼得。自研卡的解法
是让状态由程序持有：guest 的设备由卡上的固件
提供（Nitro 之下是 ena，神龙之下是 virtio），
迁移时让新宿主的卡重新实例化一次即可，
guest 无感。而且未必是 DPU，只要卡能把
guest 的设备演出来就够。这条路当年只有
自研者走得通。

这一页也翻过去了，两条路都被商用卡走通：
早先会"演"——mlx5 的 vDPA 模式
（[VDPA support for Mellanox ConnectX devices](https://lwn.net/Articles/828042/)）
2020 年起就向 guest 呈现 virtio 设备，数据
路径在卡上硬件；坚持直通裸 VF 的，mlx5 的
vfio migration variant driver 2022 年合入
主线内核
（[Add mlx5 live migration driver](https://lwn.net/Articles/873315/)），
NVIDIA 官方文档已提供完整的
[SR-IOV Live Migration](https://networking-docs.nvidia.com/mlnxofedswum/24.10-5.1.6.1lts/sr-iov-live-migration)
支持——状态的导出与恢复补齐了，商用卡的 VF
从此也能迁移。
**这笔当年独门的价值被商用路径磨平——剩下的
差别只有时间差与路线图归属，那是采购学，
不是技术。**
