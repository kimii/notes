# Clusters in the Expanse: Understanding and Unbiasing IPv6 Hitlists
--------
## 摘要
1. 近年来，一些研究提出了 IPv6 地址的目标列表（称为 IPv6 hitlists）的使用
2. 论文表明 IPv6 hitlists 严重聚类，实验使用方法使得 IPv6 hitlists 从量变到质变，测量 6+ 个月，探测 50M+ 地址
3. 新方法检测aliased prefixes，确认我们的前缀aliased率 1.5%
4. 使用 entropy 聚类，把整个 hitlist 分为 6 个不同的地址模式
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
	- 通过 Entropy 聚合（Clustering by Entropy）: 为了发现和理解整个广阔的 IPv6 空间，利用了 IPv6 地址的熵（Entropy）分析。这个有助于判断地址模式和聚类。见第四
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
		- 使用 Entropy/IP，Foremski 等推出了一个机器学习的算法，通过训练收集来的 IPv6 地址来构建一个地址模式模型和生成新的地址、
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
	6. RIPE Atlas：我们提取了所有在 RIPE Atlas traceroute中的源，包括 RIPE ipmap 项目的所有 IPv6 地址，这又添加了 0.2M 的地址。这些与过去的源高度互斥，可能是它们作为路由器的特性
	7. Scamper： 最后，我们使用在这其它源上的所有 IP 地址使用 scamper 跑了 traceroute 测量，从这些测量中提取了的所有的路由器 IP 地址，这个源表现出很强的增会特点，有 25.9M 不同 IP 地址，我们积累了所有的源，**IP 地址在我们的扫描列表中会保持不确定性**。我们可能在将来重新考虑这个决定，并且在一定不回复的窗口期之后，移除不回复的IP地址
	8. Address Runup(地址集增长): 图 1 表明过去时间内源的累计增长。首先，可以看到所有源中 IPv6 地址集的猛增: 典型的在一年的时间里，增加 10%-100%。第二，基于 DNS 的源--可能揭示服务器的地址，和从 scamper 获取 traceroute 地址占据了整个数据集。由于发现 scamper 爆炸性增长的特殊性，实施了进一步的调查，发现这些 IP 地址的 90.7% 是 **SLAAC** 地址，比如 ff:fe。从那些路由器获取的 MAC 地址的提供商代码表明它们是典型的家庭路由器： 47.9% ZTE，47.7% AVM(Fritzbox)，紧随其后的是 1.2% 的华为，和240 个其它的提供商。这表明我们的源主要是家庭路由器和 CPE 设备。基于这类类型，包括或去掉这些 CPE 设备是可能的
	9. zesplot: 为了研究这些大量的数据，推出的可视化。zesplot 中一个矩形代表一个 IPv6 前缀，不刻画整个空间而是针对输入的前缀。使用基于 squarified tree-maps(方格树状图）的地址使用递归性质（recurse）进行拓展。它从填充用矩形填充垂直行开始，然后是一个水平行，接着又是垂直行并继续。这些前缀按{prefix-size,ASN}排序，所以大的前缀画在图的左上，小前缀画在右下，并保持来自统一 AS 的相似大小的前缀相邻。只要输入是一样的，结果在 zesplot 中一个前缀将是同一个位置。坐标轴在 zesplot 中没有意义。特定的子前缀画在前缀矩形的上半部分（有着很多特定的前缀会导致在缩小的时候形成一个灰色矩形）。白色矩形在该前缀中没有地址。比如，图1c,发现左上亮红色 /19 和中心的 /32s，和非常多特定的 /127s 在右下。此外，基于属于该前缀的地址数填色。看颜色，很快能辨认出数据集中可能的最有代表的前缀点，或者确认一些假设（如大前缀比小前缀覆盖更多地址）。基于数据集，不定大小的 zesplot 可能提供额外的视角如前缀的聚类:在这些图中，所有矩形大小一致，前缀大小只用于排序。推出 zesplot 工具旨在测量研究的更多应用
	10. Input Distribution（输入分布）: 针对每个源画出 AS 的分布，发现明显的不同，如 domainlist 和 CT 只有少数的 AS 就组成了地址的大部分，对比更为平衡的 RIPE Atlas 的源。此外，我们分析 hitlist 地址到 BGP 前缀的分布，见图1c。涵盖了声明的一半 BGP 前缀，却发现一些前缀包含不寻常的大的地址数量，见第 5 章。
	11. Comparison with DNSDB（与 DNSDB 对比）: 我们的 hitlist包含 12.9% 的 IPv6 地址，69.4% 的 ASes，48.7% 的 BGP 前缀属于 DNSDB。“不见的”地址大多来自于大的 CDN 运营商，这可能是由于 DNSDB 从全球被动收集 DNS 数据，不同于从一些地方主动探测。更进一步，我们的 hitlist 大多数不在 DNSDB 中，特别是 ISP 运营商的基础设施。发现我们的 hitlist 和 DNSDB 是互补的。我们没有将 DNSDB 加入到我们的源中，因为它不可公开获取。
	12. 除了已上日常扫描能得到的源外，还对三个分离的输入源进行了深入的案例研究，
		- 使用 6Gen 和 Entropy/IP 新学习到的 IPv6 地址，见第 7 章
		- rDNS-walked IPv6 地址，见第 8 章
		- 众包客户端 IPv6 地址，见第 9 章
 
## 4.ENTROPY CLUSTERING(Entropy 聚类)
### 4.1 结果
- 我们的 hitlist 的 /32 前缀上熵聚类的结果见图2
	- 左侧聚类分布图，从上大小是不同类的百分比
	- 右侧是针对左侧每一类的指纹，半字节（nybble）的熵
- 图解
	- 图2a: 21-32 nybble（all addrs）: 6 类模式
	- 图2b: 17-32 nybble(IIDS): 4 类模式
	- 图3a: UDP/53 地址， 9-32 bybble: 7 类模式
### 4.2 讨论
- 对比 EIP 方法： 发现高层次模式跨网路，而 EIP 在一个网络发现低层级模式
- 不应局限于 /32 前缀

## 5.**别名前缀检测（APD）**
- 别名网络前缀，如一些前缀下的每个可能的 IP 地址都恢复问询，在 IPv4 测量中已被发现。对于 IPv6 测量，这是以一个更有意义的挑战，因为这可以轻松贡献很多的映射到同一服务器的地址，如在 linux 中的 **IP_FREEBIND** 选项。这个特性已经被 CDNs 使用，并已有工作被确认是一个挑战。别名前缀在 hitlists 中会有很多地址（如枚举 /96 的前缀会增加 2^32 个地址），导致使用这些 hitlist 的研究有重大偏差。鉴于此，我们只用有价值的地址填充我们的 hitlist，如地址属于不同主机，并且有均衡的前缀和 AS 分布。这要求可靠的检测和别名前缀的移除，因此以下介绍方法
	- 与[29,56]类似，我们方法的根源是在 **IPv6 大的地址空间中随机一个 IP 地址是不太可能回复的**，因此当探测随机选择的目标，收到一定回复时，一个前缀可以归类为别名
	- Murdock 等对每个 /96 前缀发送 3 个探测包到 3 个随机的地址，一旦收到回复也来自 3 个随机地址，确定前缀为别名
	- 以下描述，如何用几种方法提升方法的效率和有效性，别名检测需要满足两个拓展的条件：（1）检测必须时低带宽的，每个网络只需要少量的数据包；（2）**检测必须适应终端主机（end host），不时是路由器，因此需包含很多别名检测技术 **
### 5.1 多层级（Mutil-Level）别名前缀（aliased prefixes）检测
- 对于我们的日常扫描，执行多层级的别名前缀检测（**aliased prefix detetion，APD（见tum 公开数据 register/apd）**），如不同前缀长度。与使用固定的前缀长度如 /96 的过去的工作不同
- 技术及目标
	- 发 16 个包到前缀的随机地址，16 个中的每个包强制使用一个不同的 nybble 子前缀的遍历。例子，检查 2001:db8:407:8000::/64 是否是别名，对每个 4-bit 的前缀生成一个伪随机的地址，如 2001:db8:407:8000:[0-f]000::/68，见表 3。使用该技术，确保两点
		1. 探测包在特定前缀上
		2. 伪随机 IP 地址，不太可能回复，为目标
	对每个探测的前缀，计算回复的数量地址，如果获取了所有 16 个探测地址的回复，标记前缀为别名
- 实践	
	- 目标选择
		- 在 BGP-announced 和 我们的 hitlist 上跑别名前缀检测。
			- BGP-announced 能在全局上理解别名前缀检测现象，甚至是我们没有任何 IP 地址的前缀
			- hitlist 源能更深度检测我们的目标地址
	- 目标
		- 基于 BGP 的探测，使用每个声明的前缀，不枚举额外的地址
		- 针对自有 hitlist，把所有包含的IP 地址映射到 /64 到 /124 的前缀，4-bit 
		- 进行 APD 探测的前缀需要有超过 100 目标，原因
			1. 高效，ADP 探测需要 32 个包（ICMPv6 和 TCP/80 各 16 个）
			2. 影响，少于 100 的对自有 hitlist 影响小
		特例，/64 前缀不受此限制，为了全面分析已知的 /64 前缀
	- 探测
		- 47.4M 探测来完全覆盖所有 /64 前缀（部分地址聚合不到/64）和总的 49.2M 探测
	- 结果
		- 在多个不同前缀长度实施基于目标的 APD，可能发生四种情况
			1. more and less specific  -> aliased： 大小前缀都 aliased
			2. more and less specific -> non-aliased： 大小前缀都 non-aliased 
				- 有规律的别名和非别名行为 
			3. more specific aliased，less specific non-aliased:大前缀/x aliased, 小前缀/(x+4) non-aliased
				- 在 query 的前缀长度上有结果分歧，举例 /96 -> non-aliased; 6/19 /100 -> aliased;
				- 说明随机检测的必要，上例将整个 /96 -> aliased 不对
			4. more specific non-aliased，less specific aliased:大前缀/x non-aliased, 小前缀/(x+4) aliased
				- 异常，因为一个前缀 /x -> aliased，它的 /(x+4) -> non-aliased 占多数
				- 一个可能的原因：在自前缀探测中丢包
				- 常见例子及原因
					1. /80	
						- 3-5/16 回复率一直
						- SYN proxy，累计 SYN 回复
					2. /116	
						- 15/16 回复连续天，15 个回复包 TCP options 一致，不回复的包一直是 0x0 branch
						- 子前缀被不同与别名系统地处理，表明多层级检测地必要
					3. /120
						- 与以上现象不同，可能回复可能不回复，不定
						- ICMP 速率限制
	- 结果处理及反馈
		- 最长前缀匹配，保证细粒度
		- 在别名前缀中的 IP 地址从 ZMapv6 和 scamper 中移除

### 5.2 抗丢失（Loss Resilience）
1. 跨协议回复合并
	- ICMPv6 或 TCP80 之一回复 -> 回复
2. 多天的 sliding window(滑动窗口)
	- 见表 3
	- 减少不稳定性（aliased 和 non-aliased 间变动）
### 5.3 De-Aliasing （出去 aliase）的影响
- 2018/3/11 的 hitlist
	1. 过滤 aliase 前： 55.1M IPv6 addrs;	10866 ASes;	25465 announced prefixes 
	2. 过滤 aliase 后： 29.4M IPv6 addrs;	10853 ASes;	24648 announced prefixes
- 图 4： 前缀/AS 分布
	- AS 分布：发现别名前缀高度集中在 1 个 AS（Amazon）
	- Prefix 分布：non-aliased 轻微超过总的，原因是多数别名都在 Amazon 声明的 189 个 /48 中（检测到底最小前缀）
- 图 5： 可视化去掉别名前缀的影响

### 5.4 别名前缀指纹
- 假设：不扫描别名前缀的路由假设其中每个 IP 地址都属于同一主机，也就反映相同属性
- 验证：在探测中使用指纹技术，即使用 ZMap 支持 TCP 头部选项的模块，发送一组普遍支持的指纹选项（MSS-SACK-TS-WS），设置 WSS 和 WS 为 1 触发不同回复
- 结果：
	- 前提：指纹技术模糊并有挑战性，因而将其视为指示而非结论，结果作为**验证**，不反馈给扫描
	- 研究了被归为别名的 **20692 **/64 前缀，这些都是 16 个对 TCP80 的成功的 APD 探测；
	- 分析了回复 TTL，验证了回复 TTL 的不一致性，占比 5970/20692，因此用 iTTL 替换 raw TTL
	- 指纹技术包括
		- iTTL: initial TTL，使用不同 iTTL 作为 APD 的被动指示器，即大量不同的 iTTL 表明 non-aliased 前缀
		- Optionstext: 即把TCP选项名转化为字符串，如 MSS-SACK-TS-N-WS 为 95% 回复主机的选择
		- WScale and WSize: 预期不会是常量，主机状态的改变到导致改变。发现仅 5% 别名前缀 WS 选项不一致
		- MSS: 如 iTTL，被动佐证，不一致 MSS 才 1030 个前缀
		- Timestamps: 强指示，提供高可区分的特性，别名前缀应当表现一致。跑 3 中检测：
			1. 是否所有主机发送同样的时间戳
			2. TS 在整个前缀中是单调的
			2. 是否接受到的 TS 和 远端 TCP TS 满足回归系数，R^2 > 0.8
		发现 13202/20692 表现一致性
			- 特例： Linux 自内核 4.10 会根据<SRC_IP, DST_IP>对随机初始化 TS，导致测试失败，但这仍然强指示，APD 确实发现了别名 IP 地址，同时失败不以为不一致，而是不一定
	见表 5，共 1186 个不一致，发现 22 主机有序回复不同 iTTL（64 vs. 255）；一些主机回复 不同 TCP 选项或值。**这些大多来自 CDN**，可能是对这些服务的 TCP-level proxy（代理），可以解释 time-variant 指纹
- 验证
	- 在 2940 /64 non-aliase 有多于 16 个回复 IP 地址的前缀上跑测试，见表 6，说明了别名的区分性；但有 669 non-aliase 前缀通过 TS 测试
		- 分布
			- 99 整个前缀发相同 TS
			- 509 严格增 TS 行为： 不可能碰巧发生，因此调查见原因
			- 91 通过 R^2 测试
		- 原因
			1. 超过 /64 前缀可能是别名，但本文只探测包含超 100 IP 回复地址的前缀，然而只 166/699 满足
			2. 主机可能包含前缀中许多特定地址，而非前缀，导致测试失败，认为 166 是这种  
		- 认识
			- 本文 APD 探测低密度前缀，有些错误
			- 本文 APD 旨在发现前缀中所有地址都属于同一主机，不考虑前缀中特定地址绑定一主机
		- 总结
			- 验证表明
				1. 测试有辨识力
				2. 别民前缀配置多样，很多情况会认为是同一主机

### 5.5 与 Murdock et al.’s Approach 方法比较
- 多发现别名前缀中 992.6 k 个别名 hitlist 地址
- 探测包少一半

## 6.地址探测
步骤
	1. 从 hitlist 源收集地址
	2. 预处理，合并，打乱作为扫描输入
	3. APD 排除别名前缀目标
	4. scamper traceroute 得路由器地址
	5. ZMapv6 对所有目标探测回复性
### 6.1 回复地址
-  本文 hitlist 包含 1.9M 回复的 IPv6 地址
### 6.2 Cross-protocol Responsiveness
- 针对 239M 地址
	- ICMP，TCP/80，TCP/443，UDP/53(DNS)，UDP/443(QUIC)
- 分析见图 7
	- 关联性： 协议 X 前提，协议 Y 回复概率
- 结果（对比 IPv4）
	- 89% 回复 ICMPv6，ICMP 关联度比 v4（73%） 高，因为 ICMPv6 构成 IPv6 一部分
	- 协议相关性: HTTP <-91%- HTTPS <-98%- QUIC
### 6.3 Longitudinal Responsiveness
- 见表 8
## 7.主动学习新地址
### 7.1 方法
- 6 步
	1. 以 non-aliase 前缀作为种子列表
	2. 按 AS 划分每个列表，单个 AS 限制最多 10 万
	3. 对单个 AS 随机采样，作为 Entropy/IP 和 6Gen 的输入
	4. 使用 EIP 和 6Gen 生成 100 万/AS
	5. 对 100 万 AS 随机采样至多 10 万/AS
	6. 主动探测，评估生成地址
### 学得的地址
- 工具	生成 	相比hitlist新生成可路由
	- EIP	118M	116M
	- 6Gen	129M	124M
	- Total	239M	
- EIP 和 6Gen 生成地址重叠少，占Total 0.2% 
### 学得地址的回复性
- 对 239M 探测包括 ICMP、TCP/80、TCP/443、UDP/53、UDP/443 总回复率 0.3% -说明-> 基于学习的策略发现新回复地址是挑战
	- 新生成地址回复中 6Gen >2倍于 EIP
	- 6Gen 和 EIP 生成地址的交集（17k）更易回复（2.5%）
	- 见表格7，协议相关性，从 ICMP、UPD/53 看出 EIP 和 6Gen 不仅重叠少，而且包含的是不同类型
	- 分析 EIP 和 6Gen 的回复地址在 AS 和 Prefix 上的均衡性，见图 9
		- AS 重叠多
		- 6Gen 比 EIP 更均衡
		- 覆盖不同，EIP 包含更多 CDNs 和 Internet services

## 8.RDNS 作为数据源
- rDNS walking 可以作为 IPv6 地址的一个源；walking the rDNS tree 工作量大而且对互联网基础设施造成负担，归为'semi-public'(半公开)
- 对比分析：
	- 与本文 hitlist 交集小
	- 在 AS 分布上更均衡，因此将 rDNS 加入到 hitlist 不增加 bias
- 探测，比较回复率：
	- ICMP: 10% > hitlist 6%
	- HTTP(s): 2%(1%) < 3%(2%)
- 综上，rDNS 加入到 hitlist
## *9.客户端 IPv6 地址
### 9.1 实验部署
- 300 刀的实验
	- 众包平台： Amazon Turk(150 USD) & Profilic Academic(150 USD)
	- 见表格 9 
### 9.2 众包参与者
- 见 表格 9
### 9.3 客户端回复性
- 使用 ICMPv6，只有 17.3% 收集到的地址回复。大多数从本地网获得的 IPv6 地址不回复
- 验证低回复率是设备行为造成：使用 RIPE Atlas 位于相同 AS 探测，得到上界 45.8%，猜测用户系统本地防火墙的过滤导致
- 发现 20% 最后回复的一跳不同于目标 AS，猜测 ISP 过滤
- 观点: 'outbound only' 政策的网路部署
- 经验： 主动探测众包得到地址要在地址收集后迅速执行，因为回复地址数会迅速收缩
## 10.测量实践
- 道德考虑，保证可重复性研究
## 11.讨论
- Time-to-Measurement
	- 以 hitlis 作为输入的特定测量：保证回复率，client 设备需要几分钟内，servers 几周
- 不回复地址
	- 可以用来理解地址模式
- IPv6 hitlist service
	- 公开
## 12.总结
- 多源融合得 50M 地址，一般在别名前缀中，聚类发现 6 类 IPv6 地址模式，使用长期测量，发现协议和源的不稳定性，使用和拓展工具来生成新的地址作为地址集补充。将执行做每天的 IPv6 探测以提供有价值的 hitlist