# 基本概念
------
- VP(vantage point):源点
- IXP(Internet Exchange point):互联网交换中心


# A Survey of Techniques for Internet Topology Discovery
------
## traceroute
1. UDP,ICMP,TCP traceroute比较
	- UDP，ICMP traceroute 容易被防火墙过滤，TCP traceroute 用于穿透防火墙，但是一些防火墙这样设置--当防火墙后的常见端口上没有接受TCP连接的主机会过滤TCP包，特别是在网络边缘
	- ICMP traceroute 更容易到达目标，而 UDP 能识别更多的连接，但却在到达目标上表现最差
	- 负载均衡路由器通常采用基于包和基于流的负载均衡技术，其中基于流的会导致假路径，可采用Paris traceroute；基于包的没有解决
2. 限制和问题
	- 测量限制：1.路由器对TTL为0的探测包回复没有唯一设定；2.在每一跳记录地址不一定是有效的IP地址；3.每一跳报告的RTT值不能用于准确测量往返目标的延迟;4.layer 2 cloud 对traceroute不透明，如ATM cloud和MLPS技术



# Internet Topology Research Redux
------
## PoP级拓扑
1. PoP的定义是宽松的，对一些提供商这意味着一座带有一群路由器，交换机和其它设备的物理建筑，对其他人这意味着提供服务的都市区，总之PoPs有着明确的地理位置，如街道地址，城市或都市区

# bdrmap: Inference of Borders Between IP Networks
------
## Internet路由器级的拓扑发现和推断是乏味的和易错的，至少有5中原因
1. TCP/IP架构在网络层没有域间边界（interdomain boundaries）的概念,甚至不能完整识别路由器（所有接口）
2. 别名解析的正确性
3. 运营商地址分配和路由器实现实践限制了将traceroute中观察到的IP地址映射到在BGP中宣布该IP地址的最长匹配前缀的组织的规范方法的准确性
4. traceroute测量重复采样靠近源点的连接，导致拓扑结构中不太可能观察到距离网络较远的连接。 这种采样偏差会降低路由器所有者推断的准确性，因为可用的拓扑约束较少
5. 运营商担心向竞争对手或潜在的攻击者透露网络信息，而隐藏内部网络拓扑
## 本文5个主要贡献
1. 介绍了一个可拓展的方法来准确推断给定网络边界或各边界连接的其它网络
2. 开发了一种高效的系统来允许在资源受限的设备（不能保存整个域间图的状态）上部署我们的方案
3. 使用来自4个网络运营商的ground truth和IXP地址使用的数据库验证算法的正确性
4. 我们通过分析大型接入（access）ISP的拓扑结构来了解现代互连协议，从而证明了我们算法的实用性
5. 我们公开了源代码实现（dbrmap,scamper一部分）
## 挑战-推断路由器所有者可能不正确的7大原因
1. 路由器的接口IP地址可能来自邻居的IP地址空间  
	1. p2p中可能来自任何一方，通常是以一个 /30 或 /31 的IPv4地址段
	2. p2c中customer路由器通常使用provider的地址来回复traceroute探测
2. 边界路由器可能在回复traceroute探测时使用第三方地址
3. 边界路由器可能对traceroute探测设置防火墙
4. 虚拟路由器可能使用不同的回复接口
5. Sibling AS行为使得推断组织之间连接的尝试变得混乱
6. IXP拥有的地址在路径上表现不一致
7. 多个AS在BGP中声明相同前缀
## 边界匹配方法


# Layer 1-Informed Internet Topology Measurement
------
## 摘要
1. 本文研究这个假设--服务提供商（SP）基础设施可以用来有效地指导基于网络层TTL-limit测量的拓扑发现，关注有固定的地理位置基础设施如POPs,IXPs等
## Introduction
1. 数据源：physica map 数据来自Atlas项目（超过2600 POP节点，超过3580 POP连接），network-layer map数据来自Ark项目(201109-201303)
2. 数据中特点
	- 在physical maps中发现更多节点和连接，原因可能有i.利用DNS命名的限制；ii.隧道协议的使用如MLPS,或是让节点对探测不可见的layer 3 服务的不足；iii.网络映射基础设施（network mapping infrastructure）的视角有限;iv.事实上layer 3路由配置可能简单地排除了发现所有网络，节点和连接的能力
	- 一些出现在network-layer map的节点/位置/连接没有在physical map中出现，可能是physical maps过时，有意或错误导致的不完整
3. 实验
	- 定义targeting problem:确认layer 3探测的源目的对来揭示physical maps中的更多节点
	- 选择时核心思想：i.源目的对应该在地理上和地址空间上与目标相近；ii.测量的验证需要多个源点。通过在DNS域名的地址信息和PeeringDB中的可用纪录验证了基础设施的身份，分析表明源目的与目标在同一个AS中发现更多的物理基础设施
	- 实验结果激发了一种新的启发式探测目标的算法--POPsicle；发现IXPs在探测给定网络时扮演重要角色，源点与IXPs位置相同效果更好
4. 源点在节点发现上的影响
	- 源点和目标选择上存在3中形式：VPout->Tin;VPin->Tout;VPin->Tin（效果最好，假定不同是由于探测时域间与域内路由的对抗，更能发现路径多样性是由于ECMP--等价路由）,当源点或目标在ISP外时，RIP--域间路由协议发挥作用，比如hot-potato路由和强制选择单条最好路径
	- 在IXPs发生了许多layer 2的对等连接，并且大多由地方和区域ISP推动，Tier-1 ISP表现较慢，这提供了诱人的机会因为一般较小的网络更难发现，并且通常不部署LG服务器

