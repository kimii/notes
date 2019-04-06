# [Prais Traceroute](https://paris-traceroute.net/about/)
1. 路由器传播流量的政策(负载均衡的策略)
	1. per-packet
		- 只在乎保持均衡
	2. per-flow
		- 路由器将属于同一流的所有包转发给同一接口
		- 一般的流判别(flow identifier): 5元组 或 它们的组合，也包括其它三个字段
			- 5-tuple: 源IP地址，目的IP地址，协议，源端口，目的端口
			- other-3fields: TOS(the IP type of Service)，ICMP code, ICMP header checksum  
	3. per-destination
		- per-flow 的弱化，只在乎目的IP地址
		- 与传统路由方式一致--目的路由，不讨论
	4. 讨论
		1. 路由器负载均衡器采取 per-flow 还是 per-packet 取决于 路由器制造商（router manufactuter）、OS 版本、网络运营商（network operator）的具体配置
		2. per-flow 导致一般的 traceroute 会有问题
			1. 丢失节点和链接
			2. 包含错误链接
			3. 参考图
				- ![image](https://paris-traceroute.net/images/load_balancer.gif)
		3. paris-traceroute 的修正
			1. 参考图
				- ![image](https://paris-traceroute.net/images/traceroute_load_balancers.gif)
			2. 修正方式： 修改前 28 字节（IP 20 + UDP8 | ICMP8）
				1. UDP 探测包： 设置 checksum 字段，修改 payload 来生成一致的 checksum
				2. ICMP 探测包： 设置 ICMP identifier 和 sequence number 保持所有到目标的头部校验和一致 
			3. traceroute 探测方式头部对比图
				- ![image](https://github.com/kimii/notes/blob/master/PAPERS/pics/traceroute-packet-headers-cmp.png)
