# nmap之端口扫描技术与算法
----------
译者:[kimii](https://github.com/kimii)
参考:[Nmap Network Scanning Guide](https://nmap.org/book/scan-methods.html)
## 引言
采用 ultra_scan 算法，以下详解

1. ### TCP SYN(Stealth) SCAN(-sS) 
	1. -sS 即 SYN 扫描，root下默认扫描方式（不加该参数）
	2. 可扫描状态及原理
		1. open: 
			1. a --SYN--> b
			2. a <--SYN/ACK-- b
			3. a --RST--> b            #称 half-open
		2. closed:
			1. a --SYN--> b
			2. a <--RST-- b
		3. filtered:
			1. a --SYN--> b
			2. a --SYN--> b   #try again;depend on retransmission
		4. 包跟踪参数 --packet-trace -d(even -d5)
	3. 补充：不能绕过防火墙
	
2. ### TCP Connect Scan(-sT)
	1. 无特权用户下默认扫描方式；使用场景:没有 raw packet privileges 或者扫描IPv6网络；使用 connect 系统调用；与 TCP FTP Bounce Scan(-b) 是 nmap 中仅有的不需要特权用户的扫描
	2. 可扫描状态及原理
		1. open: 
			1. a --SYN--> b
			2. a <--SYN/ACK-- b
			3. a --ACK--> b
			4. a <--Data:banner-- b  #依赖于具体服务
			5. a --RST--> b
		2. closed:同 -sS 中
		3. filtered:同 -sS 中

3. ### UDP Scan(-sU)
	1. 发送空的 UDP header(no data)到目标端口
	2. response 划分状态
		1. open
			1. 一些端口偶尔回复（大部分系统认为空包不合法）
		2. open|filtered
			1. 由于 open port 很少回复该 probe，会直接丢弃,但防火墙等本身也丢包
		3. closed
			1. ICMP不可达错误（type 3,code 3）
		4. filtered
			1. 其它ICMP不可达错误（type 3,code 1,2,9,10,13）
	3. 两问题
		1. slow:Linux 的 ICMP response rate limiting，具体指默认限制 ICMP 不可达消息的速率，eg:Linux 2.4.20 中net/ipv4/icmp.c 限制每主机 1个/s，means 65536-ports needs 18h，解决方法:
			1. 增加主机并行度，--min-hostgroup,（eg:~=100,意味着接收 100/s)
			2. 先扫常见端口，eg:-F
			3. 添加 --version-intensity 0 到服务探测扫描，意味着 nmap 对给定端口仅尝试最可能的 probes
			4. 在防火墙后扫描:防火墙限速
			5. 使用 --host-timeout 跳过慢主机
			6. 使用 -v 有预估时间，just wait and do sth instead   
		2. 有歧义状态: open|filtered，解决方法:
			1. 使用服务探测：加入-sV(or -A),即使用 -sUV,发送特定 probes 到 open|filtered 端口再回复为 open，特定 probes 是由于 UDP 服务一般定义自己的包结构而非 nmap 通用的，所以要用到 nmap-service-probes
			2. 专门的 traceroute 如 hping2（eg:hping2 --udp --traceroute -t 8 -p 53 scanme.nmap.org)来判断防火墙区分 filtered
			3. 针对公共端口尝试特定应用程序的工具,eg:SNMP爆破
			
4. ### TCP FIN,NULL,and Xmas Scans(-sF,-sN,-sX)
	 1. 依据 RFC793:目标端口关闭，进入 segment 不设 RST 导致回复中带 RST<--区分-->到open port的包不设 SYN,RST,ACK,drop the segment,return
	 2. 应用|优点：绕过某些防火墙(eg:检查 SYN bit)或包过滤路由；比 SYN 更隐蔽(stealth),但某些 IDS 可检测
	 3. 分类及原理
		 1. Null Scan(-sN):不设任何位(TCP flag header is 0)
		 2. Fin Scan(-sF):只设TCP FIN 位
		 3. Xmas Scan(-sX):设置 FIN, PSH, and URG flags
		 4. 状态
			 1. open|filtered
			 2. closed
			 3. filtered
	 4. 问题
		1. 一些系统不支持 RFC793,不支持 eg:Microsoft Windows,many Cisco devices,and IBM,支持的 eg:Unix-based systems；判断是否支持：Test T2(a null packet to a open port),T2(R=N)表明系统支持该协议**??**
		2. 不能区分 open|filtered，加 -sV 对该扫描大多数不使用（对比 -sU），换 SYN

5. ### Custom Scan Types with--scanflags
	1. --scanflags URGACKPSHRSTSYNFIN 设置标志位；配合 -sS，扫描行为不变，仅改变特定标志位，custom即订制化的
	2. 分类及原理
		1. Custom SYN/FIN Scan 
			1. 扫过防火墙对入包中仅过滤 SYN 标志的(入包 SYN|ACK 被接收)
		2. PSH Scan
			1. 尝试其他scanflags 组合，如 PSH|URG 和 FIN|PSH(-sF --scanflags PSH)。

6. ### TCP ACK Scan(-sA)
	1. 场景:仅区分 unfiltered 和 filter，即判断防火墙过滤规则
	2. 分类及原理
		1. unfiltered:扫描未过滤系统，open or closed port 返回 RST 包
		2. filtered:ports 不回复或者返回某些 ICMP error 消息(type 3,code 1,2,3,9,10,13)
		3. 
7. ### TCP Window Scan(-sW)
	1. 利用某些系统的实现细节(一些系统 RST 包返回时 TCP Window 的值，open port 非 0 有意义值，closed port 为 0)
	2. 分类及原理
		1. open
			1. TCP RST 回复 window field 非 0
		2. closed
			1. TCP RST 回复 window field 为 0
		3. filtered
			1. 没收到回复(即使重传)
			2. ICMP不可达错误（type 3,code 1,2,3,9,10,13）
	3. 优点：SYN无法区分 open|filtered，对比 SYN 和 FIN scan 结合，-sW 更简单；可以用来确认结果一致性

8. ### TCP Maimon Scan(-sM)
	1. 来源：Uriel Maimon 发现发送 FIN/ACK，很多 BSD-derived 系统开放端口丢弃包(同 RFC 793 TCP 4.中讨论)
	2. 分类及原理
		1. open|filtered
			1. 没收到回复(即使重传)
		2. closed
			1. TCP RST 包
		3. filtered
			1. ICMP不可达错误（type 3,code 1,2,3,9,10,13）
9. ### TCP Idle Scan (-sI)
	1. 1998年 Antirez(写了hping2)提出的新端口扫描技术，非常隐蔽(可指定发包 zoombie host)，允许发现机器之间基于IP的信任关系**？？**
	2. 原理
		1. 基本事实
			1. SYN Scan 原理基本的扫描方法
			2. 机器收到没来由的 SYN/ACK 会回复 RST，没来由的 RST 包会被忽略
			3. IP ID，一些系统简单增加，攻击者可以知道发了多少包
		2. Idle Scan 步骤及原理
			1. 3步
				1. 探测 zoombie's IP ID，记录它
				2. 伪造 zoombie host 向目标端口发送 SYN 包，端口状态判断取决于 IP ID 的增长
				3. 再探测 zoombie's IP ID，记录它
			2. 原理
				1. IP ID 增长 1 表明 not open(closed|filtered);增长 2 表明 open
		3. 找到工作的 idle scan zoombie host
			1. 要求：需要 idle 的，避免多余流量影响 IP ID，一些如 google.com肯定不是
			2. 方法：
				1. 常用的-iR+ping 后可用主机，最好选离源地址或 target 近的；
				2. 简单网络设备，大多 idle 和 使用简单网络栈易于做 IP ID 检测
				3. 在预选的 zoombie 上执行端口扫描和 OS 探测比 执行 ping 要好，设置 -v，输出中有“Incremental 
or Broken little-endian incremental”表明可选，一些系统不适用，如 Solaris和某些系统与每个主机通信会启动一个新的 IP ID**??**这些系统怎么做packect counts,对比Linux ifconfig 的网卡包统计
		4. 使用：nmap -PN -p- -sI kiosk.adobe.com www.riaa.com  # -sI zoombieIP dstIP；-Pn:避免 ping 影响 IP ID；测的端口(attacher to target)不能被过滤，SYN Scan 下端口为 open 或 closed 
		5. 实现算法，与基本区别：快速执行的并行度和减少误报的冗余度
			1. 并行：
				1. 问题：扫很多端口，IP ID增长，知道端口开放数量不能确定哪一个，
				2. 二分法：一组 ports，开放数对应增长数<N>，再二分直到只有一个或0个
			2. 可靠性
				1. 二分法继续:Nmap 检查二分后 open 数量一致性<N>，否则退出  
			3. **实验验证**
				1. 命令：nmap -sI Zombie -PN -p20-25,110 -r --packet-trace -v Target #-r：关闭端口随机；内部细节：二分，反复测，逐步确认，频繁测 zoombie 是否 idle
				2. 具体实现参见 idle_scan.cc
    3. 优点：终极隐蔽；攻破一些包过滤防火墙和路由（源地址检查）；
    4. 缺点：花费时间较长；两个挑战:欺骗包（给目标伪装 zoombie，ISP 可能出口过滤）和找到 zoombie host

10. ### IP Protocol Scan(-sO)
	1. 扫描目标机器支持哪些协议号，-p 选项选择协议号(256中)
	2. 原理
		1. 发送的 IP 包修改的是 8-bit IP 协议字段，header 通常是空的，除了已经有实现的 TCP，UDP，ICMP，注意这里观察的是 ICMP **协议不可达**消息而不再是 ICMP 协议端口不可达消息
		2. 回复状态分类
			1. open
				1. 目标主机任何协议有回复(只是回复用到的协议，不需要探测协议)
			2. closed
				1. ICMP 协议不可达错误(type 3，code 2)
			3. filtered
				1. 其它 ICMP 不可达错误(type 3，code 1,3,9,10,or 13)
			4. open|filtered
				1. 没回复(即使重传)
	3. 优点：每个开放协议都有潜在利用价值，可以决定机器的目的和包过滤次序，终端主机通常很少打开除了 TCP,UDP,ICMP,和 IGMP(有时)其它协议，**然而路由有更多，包括路由相关协议如 GRE 和 EGP**，防火墙和 VPN 网关有更多加密相关的协议如 IPsec 和 SWIPE
	4. 缺点：ICMP 协议不可达消息也有速率限制（同 ICMP 端口不可达），但 256 的协议号比 65535 端口号问题要小
	5. 
11. ### TCP FTP Bounce Scan(-b)
	1. FTP 弹跳扫描，基于 FTP 协议(RFC 959)支持的代理 FTP 连接，当用户连接到一个 FTP 服务器，要求发送文件到第三方服务器，这样的特点可以在很多层面滥用，所以大多数服务器已经不在支持它。一个滥用是使得 FTP 服务器扫描其他主机，就是简单要求该服务器发送文件到目标端口，错误信息会区分端口状态。
	2. 历史：1997，nmap 发布时还还很泛滥，现在大多数以修复
	3. 使用：nmap -p 22,23,135 -Pn -v -b XXX.YY.111.2 scanme.nmap.org

12. ### 扫描代码和算法
	1. 介绍：04 年 Nmap 为了更好的性能和准确度重写了端口扫描引擎，即 ultra_scan，基本包含前面的扫描技术，除了 idle scan 和 FTP bounce scan 使用它们自己的引擎
	2. 考虑的问题：除了以上扫描技术工作原理，还考虑以下问题，端口和主机并行度，延迟评估，丢包检测，时间配置，异常网络状况，包过滤，和恢复速率限制等，以下只是重要的特点，具体的在 ultra_scan 和 scan_engine.cc 中找到
	3. 最重要的算法特点：
		1. 网络条件监控
			1. 一些扫描器声称比 nmap 快因为无状态，但只适合速度比全面性和准确度更重要的快速调查，这不适合安全扫描，无状态扫描无法检测丢包来重传和调节发送速率，nmap 在 ram 中保存了大量状态，标记每个探针以序列号，源目的端口，ID 字段等
			2. nmap 拥塞控制算法；--min-rate(最小发包速率)，--max-retries（最大重传），--min-rate 100--max-retries 0 类似于一个无状态扫描
		2. 主机和端口并行度
			1. --min-hostgroup --min-paralleism -T4 --max-rtt-timeout 等，参见优化 nmap 性能
		3. RTT 估计
			1. nmap 维持 3 个重要的时间相关的值，每个主机有独立的一组值，也为并行主机合并值
				1.  srtt(the smoothed average RTT)
					1. newsrtt = oldsrtt + (instanceRTT - oldsrtt)/8   #
					2. 解释：网络条件变化频繁，不用算术平均值，采用更当前的值
				2.  rttvar
					1.  newrttvar = oldrttvar + (ABS(instanceRTT - oldsrtt) - oldrttvar)/4   
					2.  解释：指的是 RTT 中观测的变量和偏差；想法是如果 RTT 值相当连续就在等待 srtt 后很快放弃，如果该变量相当高，nmap 在放弃一个探针前必须等比 srtt 更长的时间；ABS绝对值；
				3.  timeout
					1.  timeout = newsrtt + newrttvar * 4
					2.  解释：nmap 在放弃一个探针前愿意等待的时间；超时后会重传或者设置 filtered；
			2. 补充：这些公式基于 TCP 和 RFC 2988，Computing TCP's Retransmission Timer 的相关技术，但为了更适应端口扫描做了优化
		4. 拥塞控制
			1. nmap 使用模仿 TCP 的三种算法，来控制扫描的侵略性：congestion window（nmap 用来控制依次发多少包），exponential backoff（nmap 检测都丢包时如何动态减速）,slow start（一个相当快的算法来逐渐增加扫描速度）；所有这些技术包含在 RFC 2581 TCP Congestion Control 中。
			2. 区别：与 TCP 拥塞控制，nmap 记录发送包到接收的比率，一组拥塞窗口改变，变化的数值就乘以这个比率，这是由于 TCP 流假设回复中的 ACK 数来增加拥塞窗口，nmap 面对的是防火墙的限制，发送的包很少收到。
		5. Timing probes
			1. 问题：过滤器丢掉大多数包却没回复，nmap 得发送 20,000 probes 或更多才能找到一个回复的端口，监控网络条件因而变难
			2. 解决：使用 Timing probes 即 port scan pings 找到一个回复端口，发送一个探针/1.25s 到该端口而且不收其他端口的回复来有效监控
		6. Infered Nerghbor Times
			1. 问题：port scan pings 没用，主机不存活或者主机有一对回复端口，nmap 不能完全发现。
			2. 解决：使用为一组机器维持的 timing values，但不假设一组总是共享相似的 timing characteristics，nmap 追踪这些值，差别大就推测更大的超时时间
		7. Adaptive Retransmission
			1. 问题：针对丢包策略
			2. 解决：没检测丢包，nmap 可能只当没有接收到一个探针回复重传一次；当明显大量丢包，nmap 重传 10 次甚至更多，然后警告并放弃重传
		8. Scan Delay
			1. 问题：包回复速率限制，如果 nmap 把这视为正常丢包，会导致持续降速但还是会丢掉大多数探测包
			2. 解决：nmap 尝试检测这种情况，一开始在对目标两探针间 short delay，如果丢包仍然严重，double delay；默认 scan delay 是 1s/探针间，设置用 --max-scan-delay 