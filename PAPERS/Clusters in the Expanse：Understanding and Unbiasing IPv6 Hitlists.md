# Clusters in the Expanse: Understanding and Unbiasing IPv6 Hitlists
--------
## 摘要
1. 近年来，一些研究提出了 IPv6 地址的目标列表（称为 IPv6 hitlists）的使用
2. 论文表明 IPv6 hitlists 严重聚类，实验使用方法使得 IPv6 hitlists 从量变到质变，测量 6+ 个月，探测 50M+ 地址
3. 新方法检测aliased prefixes，确认我们的前缀aliased率 1.5%
4. 使用 entropy 聚类，把整个 hitlist 分为 6 个不同的地址方案
5. 利用众包（crowdsourcing）执行客户端测量（client measurement，client 区别于 servers or routers）

## 1.引言
- 难点: 整个 IPv6 Internet的扫描是不可能的，由于它的大小超过了技术上可以发送和存储的量级，道德上也超过了发送给一个系统或网络的问询数量。
- 当前策略: 最先进的 IPv6 Internet 扫描仍基于 IPv4 Internet 扫描早年所使用的方法，使用目标 IP 地址列表，成为 hitlist，作为 IPv4 地址空间的代表性的子网
- 挑战： 就 hitlist 而言，IPv6 地址空间也伴随着特有的、不同的挑战。
	1. hitlisty 结果可能是有偏（biased）的（如不能代表整个 Internet）由于不平衡的 AS 和前缀代表或 IP 地址别名
	2. 由于单个网络和单个机器有着同样的大的划分空间，很容易覆盖一个有着无数的 IP 地址的 hitlist
	3. 地址可能只在短期内使用，因此没有重复使用的压力
总之，IPv6 hitlists 的一个关键点不是 IP 地址的计数，而在于在 ASes 和 前缀上做到有代表性（responsiveness）和均衡性（balance）。本文系统性地处理这些挑战通过
	- 综合的地址发现（Comprehensive Address Discovery）: 在无偏(unbiasing) hitlist 第一步是创建一个综合的 hitlist，其从多个最好的源中提取。见第三章
	- 通过 Entropy 聚合（Clustering by Entropy）: 为了发现和理解整个广阔的 IPv6 空间，利用了 IPv6 地址的熵（Entropy）分析。这个有助于判断地址方案和聚类。见第四章
	- 去别名（De-Aliasing）: 减少潜在的别名前缀的影响，比如单个机器对一个可能的大的前缀的所有地址有回复，假定和实现了一个严格的（rigorous）方法来进行别名前缀检测。见第五章
	- 长期稳定探测（Longitudinal Stability Probing）:为了发现可靠的回复地址，针对我们的 hitlist 实施了长时间的多协议扫描。和预想的那样，已发现的 IP 地址只有一小部分真正地回复探测，这对管理无偏的 hitlist 来说是一个重要的条件
	- 多样化地址集（Diversifying Address Population）: 评估了 3 个正交方法来推动 IPv6 hitlist 的前沿
		- 使用 Entropy/IP 和 6Gen 生成地址，见第 7 章
		- 利用 rDNS 记录，见第 8 章
		- 众包客户端 IP 地址,见第 9 章
	- 绘图用于试探性分析（Plotting for Exploratory Analysis）: 对 IPv6 地址集进行可视化是有挑战的，如 IPv4 方法，比如用希尔伯特曲线刻画整个地址空间，不能拓展到 IPv6。推出新的通过选择性输入的绘图技术，比如 BGP 声明前缀，而不是对整个空间可视化。画法见第 3 章，整个文章使用这些图来直观地展示数据
	- 公开和分享（Publication and Sharing）
	- 高的科学道德和工作可重复性，见[代码链接](https://ipv6hitlist.github.io)		

## 2.相关工作
- 调研了以稀疏的 IPv6 空间为目标的工作，和我们的工作做了全面的比较，见表1，与之前私有工作对比，基于4个指标，从公有源获取的地址数、BGP前缀树、从私有源获取的地址数、是否包含客户端地址、是否执行主动探测、是否实施别名前缀检测
	1. DNS 技术
		- DNS 很早就被作为可能的 IPv6 地址源，Strowes 提出使用 in.addr.arpa IPv4 rDNS 树来获取解析到 IPv6 地址的名字，基于这样的假设--**协议之见的名字很常见**（？naming is common between protocols），它发现了位于 5531 ASes 的 965k IPv6 地址，其中 56.7% 是可回复的；
		- 最近，Fiebig 等遍历了 rDNS 树来获取 2.8M IPv6 地址，它们没有探测这些地址的可回复性
		- Borgolte 等通过 NSEC-walking reverse zones 签名的 DNSSEC 来发现 IPv6 地址
		- 本工作评估了 rDNS 作为一个源的可回复性，发现 rDNS IPv6 地址，作为 hitlist 的一个有价值的补充
	2. 结构属性
		- IPv6 地址空间可能很松散，但网络中的地址计划趋向于预示结构
		- Ullrich 等使用规则挖掘（rule mining）发现了几百 IPv6 地址
		- 使用 Entropy/IP，Foremski 等推出了一个机器学习的算法，通过训练收集来的 IPv6 地址来构建一个地址方案模型和生成新的地址、
		- 本工作，改善了 Entropy/IP 来生成探测 IPv6 的地址，我们介绍了新的 entropy 聚类方法
		- 与利用 IPv4 中密集区域努力类似，Murdock 等推出了 6Gen 来发现 IPv6 地址空间的密集区域和生成临近的地址，本工作基于我们的 hitlist 使用 6Gen 来生成地址，并且对比了 Entropy/IP 的性能
		- Murdock 等也完成了别名前缀检测（APD）的变种，本工作进行了拓展
		- Plonka and Berger 利用了 IPv6 地址的结构计划来允许大的数据集来分享
	3. Hitlists
		- Gasser 等从多个源收集的 hitlist，他们的 149M IPv6 地址的大多数取自非公共不可获取的源（non-public passive sources）
		- 我们基于他们（Gasser 等）的策略构建，但只使用公共可用的源来保证我们的工作可重复
		- **最近，Beverly 等使用大规模 traceroute 分析了 IPv6 拓扑，使用 Gasser 等在其它源中公共可用的 hitlist **
	4. Crowdsourcing
		- 有些研究利用众包（crowdsouring）平台来实施网络测量
		- 我们已以前的工作 Huz 等的工作为基础--使用了 [Amazon Mechanical Turk](https://www.mturk.com/) 平台来测试带宽速度和 IPv6 使用（IPv6 adoption）
		- 对比 Huz 等在 2015 年收集的 38 IPv6 地址，我们发现更多

## *3.IPv6 HITLIST SOURCES(Hitlist 源)
- 利用很多源，见表2；原则上只选用公共可免费获取的源来使得工作可重复；源包括
	1. Domain Lists: 212 M domains 来自多个大的 zones，基于每天 AAAA 记录的解析，生成大约 9.8M 不同 IP 地址。这个源也包括从 Spamhaus、APWG、Phishtank 提供的黑名单中提取的域名，分别利用了 8.5 M、376 k、170 k 域名
	2. FDNS： 一个综合的集合，来自Rapid7 实施的 forward DNS (FDNS) ANY lookups，生成 2.5M 不同地址
	3. CT： Certifcate Transparency (CT) 记录的 TLS 证书中提取的 DNS 域名，不是域名列表（Domain Lists，见1）已经有的部分，提取 16.2M 地址
	4. AXFR and TLDR：从来自TLDR project 和我们自己的 AXFR DNS transfers 的 zone transfers (AXFR) 获取的 IPv6 地址。获取的域名也从每天的 AAAA 记录解析而来。生成 0.5M 不同 IPv6 地址
	5. Bitnodes：为了获取客户端地址，我们使用了 Bitnodes API，提供 Bitcoin 网络的当前 peers，尽管是最小的源，但贡献了 27K 不同的 IPv6 地址，我们仍然发现它添加了客户端地址
	6. RIPE Atlas：我们提取了所有在 RIPE Atlas traceroute中提取的源，包括 RIPE ipmap 项目的所有 IPv6 地址，这又添加了 0.2M 的地址。这些与过去的源高度互斥，可能是它们作为路由器的特性
	7. Scamper： 最后，我们使用在这其它源上的所有 IP 地址使用 scamper 跑了 traceroute 测量，从这些测量中提取了的所有的路由器 IP 地址，这个源表现出很强的增会特点，有 25.9M 不同 IP 地址，我们积累了所有的源，IP 地址在我们的扫描列表中会保持不确定性。我们可能在将来重新考虑这个决定，并且在一定不回复的窗口期之后，移除不回复的IP地址
	8. Address Runup:
  
## 4.ENTROPY CLUSTERING(Entropy 聚类)

## *5.别名前缀检测
### 5.1 多层级（Mutil-Level）别名前缀（aliased prefixes）检测
### 5.2 抗丢失（Loss Resilience）
### 5.3 De-Aliasing 的影响
### 5.4 别名前缀指纹
### 5.5 与 Murdock et al.’s Approach 方法比较
 
## 6.地址探测
### 6.1 回复地址
### 6.2 Cross-protocol Responsiveness
### 6.3 Longitudinal Responsiveness

## *7.主动学习新地址
### 7.1 方法
### 学得的地址
### 学得地址的回复性

## 8.RDNS 作为数据源

## *9.客户端 IPv6 地址
### 9.1 实验部署
### 9.2 众包参与者
### 9.3 客户端回复性


## 10.测量实践
### 10.1 道德考虑
### 10.2 可重复性研究

## 11.讨论

## 12.总结