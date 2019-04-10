# c++ thread

## 文件I/O
- close() 函数与记录锁
	- int close(int fd); #关闭文件会释放该进程施加在该文件上的所有记录锁
	- 记录锁，见 fcntl 函数
- socket 套接字函数 read | write | close , 与文件操作没有本质区别
- ssize_t read(int fd, void *buf, size_t nbytes)	
	- 参数 buf、nbytes 使用
```
#define BUFFSIZE 4096
while((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0)
	...
```
	- 4096 设置: linux ext4 文件系统，磁盘块长度 4096 字节 + Linux 上不同缓冲长度读操作实际CPU时间考虑
- int fcntl(int fd, int cmd, .../* int arg */);
	- 改变打开文件的属性，5 种功能
		1. 赋值已有描述符
		2. 获取/设置文件描述符状态
		3. 获取/设置文件标志状态
		4. 获取/设置文件异步I/O所有权
		5. 获取/设置记录锁
			- int fcntl(int fd, int cmd, .../* struct flock *flockptr */);
```
// flock 结构体
struct flock{
	short l_type; /* 锁的类型 共享读锁F_RDLCK、独占性写锁F_WRLCK、解锁F_UNLCK */
	short l_whence;	
	short l_start;
	short l_len;
	short l_pid;
}
```
			- 读写锁：共享读锁F_RDLCK、独占性写锁F_WRLCK
			- 锁的自动继承与释放
				- fork 产生子进程不继承
				- exec 新程序可以继承原程序的锁     
				
## IPC
1. 匿名管道 PIPE 无名管道
	- pipe 或 shell 使用 |
	- 有公共祖先进程间通信，如父子进程间通信
		- int pipe(int fd[2]); #由参数 fd 返回两个文件描述符，fd[0] 为读打开、fd[1] 为写打开
2. 命名管道 FIFO 有名管道
	- mkfifo
	- 无需进程关联
	- [匿名与命名对比实例](https://cloud.tencent.com/developer/article/1005566)
3. 消息队列（message queue）
	- 消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点
4. 信号量
	- 信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段
5. 共享内存
	- 最快进程间通信，两次内存拷贝，对比套接字四次内存拷贝
	- 使用：mmap & System V 
	- 
6. 套接字

## 多线程
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                