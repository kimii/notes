# 合集
- [虚函数概念及实现](https://www.cnblogs.com/xien7/archive/2013/03/12/2954364.html)
	- 形式
		- virtual func() {}	
	- 设计目的
		- 基类（base class）期待派生类（derived class）重新定的函数，基类希望派生类直接继承的不能定义为虚函数
	- 多态
		- 指向派生类的基类指针，访问派生类中同名覆盖成员函数
	- 虚函数表
	- 抽象类
	- 虚继承--菱形继承


# TOP10
## 阿里
1. 进程在内存中结构
	- 进程内存结构
	```
	+——————————————+<—— 高地址 
	|    stack     |<——栈区(向下增长)
	+——————————————+
	|      |       |
	|      v       |
	|      ^       |
	|      |       |
	+——————————————+
	|      ^       |
	|      |       |
	+——————————————+
	|     heap     |<——堆区（向上增长）
	+——————————————+
	|   bss seg    |<——BSS区（未初始化全局变量，用0初始化）
	+——————————————+
	|  data seg    |<——数据区（已初始化全局变量，静态变量和常量数据）
	+——————————————+
	|  text seg    |<——代码区（可执行代码）
	+——————————————+<—— 低地址
	```
	- 解释
		- 代码段： 存放CPU执行的机器指令，代码区是可共享，是只读的
		- 数据区：存放已初始化的全局变量、静态变量（全局和局部）、常量数据
		- BBS区：存放的是未初始化的**全局**变量和静态变量(BSS：Block Started by Symbol)
		- 栈区：由编译器自动分配释放，存放函数的参数值、返回值和局部变量，在程序运行过程中实时分配和释放，栈区由**操作系统自动管理**，无须程序员手动管理
		- 堆区：堆是由malloc()函数分配的内存块，使用free()函数来释放内存，堆的申请释放工作**由程序员控制**，容易产生内存泄漏
	- 参考
		- [链接1](http://www.icode9.com/content-3-1272.html)
		- 
2. TCP三次握手/四次挥手
	- 三次握手示意图

	![三次握手](https://i.imgur.com/kVNTmle.png)
	- 解释
		- 目的：连接服务器指定端口，建立TCP连接，并同步连接双方的序列号和确认号并交换 TCP 窗口大小信息
		- 过程
			- 触发三次握手：客户端connect()
			- 3 step（数据包对应状态为发出方立即改变，接受方收到后改变）
				1. client -> server: 
					- 数据包中 SYN=1;SEQ_NUM=J
					- client -> SYN_SEND 状态
				2. server -> client: 
					- 数据包中 ACK=1;ACK_NUM=J+1;SYN=1;SEQ_NUM=K;
					- server -> SYN_RECV 状态
				3. client -> server: 
					- 数据包中 ACK=1;ACK_NUM=K+1
					- client&server -> ESTABLISHED 状态

	- 四次挥手示意图

	![四次挥手](https://i.imgur.com/vAugTjo.png)
	- 解释
		- 过程
			- 触发四次挥手：客户端或服务器均可(TCP 全双工)
			-  4 step(发起方 active; 接受方 passive; 数据包对应状态为发出方立即改变，接受方收到后改变)
				1. active -> passive: 
					- 数据包中 FIN=1;SEQ_NUM=M
					- active -> FIN_WAIT1 状态； passive -> CLOSE_WAIT 状态
				2. passive -> active: 
					- 数据包中 ACK=1;ACK_NUM=M+1
					- active -> FIN_WAIT2 状态
				3. passive -> active: 
					- 数据包中 FIN=1;SEQ_NUM=N
					- passive -> LAST_ACK 状态; active -> TIME_WAIT 状态
				4. active -> passive: 
					- 数据包中 ACK=1;SEQ_NUM=N+1
					- passive -> CLOSED 状态;
	- 问题
		1. 为什么建立需要 3 次，关闭要 4 次
			- 建立少一次是因为服务器在 LISTEN 状态 SOCKET 收到 SYN 的连接请求，可以 ACK+SYN 一起发送
			- 但关闭连接时，收到对方 FIN，仅仅代表对方没有数据发给你了，不意味着你所有数据发送给对方了，所以可能不会马上关闭 SOCKET，需要再发送一些数据给对方，再发送 FIN，所以拆开 ACK 和 FIN
		2. 为什么是 3 次握手不是 2次
			- 三次握手既要双方做好发送数据的准备工作，也要允许双方就初始序列号进行协商
			- 防止重放：client 请求报文超时到达，server 将回复 ACK 建立连接并进入等待，三次可通过 client 回复 ACK 时间防止
			- 防止死锁：改成两次可能发生死锁，第二步 server 给 client 的 ACK 分组一旦丢失，client 将不知道 server 是否准备好，不知道 server 序列号，在这种情况下，client 将认为连接为建立成功，忽略 server 发来的任何分组，只等待 ACK 分组，但 server 在发出分组超时情况下，将重复发送，形成死锁
		3. 四次挥手 TIME_WAIT 状态需要等2MSL后才能返回到CLOSED状态
			- 确保 ACK 报文被对方接受，TIME_WAIT 状态作用用来重发可能丢失的 ACK 报文
	- 参考
		- [链接](http://www.cnblogs.com/zmlctt/p/3690998.html)

## 腾讯
1. HTTP
	1. 状态码
		- 5 类
			- 1XX(INFO):  信息性状态码，表示接受的请求正在处理
			- 2XX(Success): 成功状态码，表示请求正常处理完毕
				- 200: ok
				- 204: No content
			- 3XX(Redirection): 重定向状态码，表示需要客户端需要进行   附加操作
			- 4XX(Client Error): 客户端错误状态码，表示服务器无法处理请求
				- 400 Bad Request
				- 401 Unauthorized
				- 403 Forbidden
				- 404 Not Found
			- 5XX(Server Error): 服务器错误状态码，表示服务器处理请求出错
				- 500 Internal Server Error	
				- 503 Service Unavailable
	2. 特点
		- 无连接：无连接的含义是限制每次连接只处理一个请求
			- 问题：但随着时间的推移，网页变得越来越复杂，里面可能嵌入了很多图片，这时候每次访问图片都需要建立一次 TCP 连接就显得很低效。
			- 解决：keep-Alive 被提出用来解决这效率低的问题
		- 无状态：HTTP协议是无状态协议，给服务器发送 HTTP 请求之后，服务器根据请求，会给我们发送数据过来，但是，发送完，不会记录任何，无状态特性简化了服务器的设计，使服务器更容易支持大量并发的HTTP请求


2. 

