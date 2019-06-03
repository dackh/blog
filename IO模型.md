# **区分**
一个输入操作通常包括两个阶段：
- 等待数据准备好
- 从内核向进程复制数据
对于一个套接字上的输入操作，第一步通常涉及等待网络数据从网络到达。当所等待数据到达后，它被复制到内核中的某个缓冲区。第二部就是把数据从内核缓冲区复制到应用进程缓冲区。

同步IO和异步IO的区别就在于第二个步骤是否阻塞，如果实际的IO读写阻塞请求过程，那么就是同步IO，因此阻塞IO、非阻塞IO、IO复用、信号驱动IO都是同步IO，如果不阻塞，而是操作系统帮你做完IO操作再将结果返回给你，那么就是异步IO。

阻塞IO和非阻塞IO的区别在第一步，发起IO请求是否会被阻塞，如果阻塞直到完成那么就是传统的阻塞IO，如果不阻塞，那么就是非阻塞IO。



# **阻塞式IO**
应用进程被阻塞，知道数据复制到应用进程缓冲区才返回。

应该注意到，在阻塞过程中，其他程序还可以执行，因此阻塞并不意味着整个操作系统被阻塞。因为其他程序还可以运行，因此不消耗CPU时间，这种模型的CPU利用率会比较高。

![clipboard.png](https://segmentfault.com/img/bVbgAYV)

# **非阻塞IO**
应用程序执行系统调用之后，内核返回一个错误码。应用程序还可以继续运行，但是需要不断的执行系统调用来获知IO是否完成，这种方式成为轮询。

由于CPU要处理更多的系统调用，因此这种模型的CPU利用率比较低。

![clipboard.png](https://segmentfault.com/img/bVbgAZA)

# **IO复用**
使用select或者poll等待数据，并且可以等待多个套接字中的任何一个变为可读。这一过程会被阻塞，当某一个套接字可读时返回，之后使用recvfrom把数据从内核复制到进程中。

它可以让一个进程具有处理多个I/O事件的能力，又被称为Event Driven IO,即事件驱动IO。

![clipboard.png](https://segmentfault.com/img/bVbgA3t)

当用户调用select时，会阻塞在select调用上，同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

**I/O 多路复用的特点是通过一种机制一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，select()函数就可以返回。**


跟blocking IO相比，IO复用需要两个system call(select和recvfrom)，而blocking IO只需要一个system call，但是用select的优势在于它可以同时处理多个connection。

这是因为用户process是被select这个函数阻塞，而不是被socker IO阻塞。
 
# **信号驱动IO**
应用进程使用sigaction系统调用，内核立即被返回，应用进程可以继续执行，也就是说等待数据阶段应用程序时非阻塞的。内核在数据到达时向应用进程发送SIGIO信号，应用进程收到之后在信号处理程序中调用recvfrom将数据从内核中复制到应用进程。

相比于非阻塞IO的轮询方式，信号驱动I/O的CPU利用率更高。

![clipboard.png](https://segmentfault.com/img/bVbgA4k)

# **异步IO**
应用进程执行aio_read系统调用会立即返回，应用进程可以继续执行，不会被阻塞，内核会在所有操作完成之后向应用进程发送信号。

异步IO与信号驱动IO的区别在于，异步IO的信号是通知应用进程IO完成，而驱动IO的信号是通知应用程序可以开始IO。

![clipboard.png](https://segmentfault.com/img/bVbgA8a)

# **select & poll & epoll**
select poll epoll这三个都是常用的IO复用的系统调用。

### select
select函数：


	int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeeval *timeout)

select监视的文件描述符有三类，readfds、writefds、exceptfds，调用select函数之后会阻塞，知道有描述符就绪（可读、可写、有except)，或者超时(timeout指定等待时间），当select函数返回后，需要遍历fdset来找到就绪的描述符。

select的其中给一个缺点是能够监视的文件描述符存在最大限制，在linux上一般为1024。

### poll
poll函数：

	int poll(struct pollfd *fds, unsigned int nfds, int timeout);

不同于select使用三个位图来表示fdset的方式，poll使用一个pollfd的指针实现
	
	struct pollfd{
		int fd; 		/* file descriptor */
		short events;   /* requested events to watch */
		short revents;  /* returned events witnessed */
	}

pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。

### select、poll的缺点
- 单个进程能够监视的文件描述符的数量有限制，因为select、poll在内核中都是通过轮询的方式扫描文件描述符来判断事件发生，文件描述符越多，性能越差。
- 内核、用户空间的拷贝问题，每次调用select，都需要把fd集合从用户态拷贝到内核态。
- 当监视的文件描述符发生变化的时候，select返回整个fd数组，应用程序需要遍历整个fdset才能发现哪个fd发生了事件。
- select的触发方式是水平触发（LE），应用程序如果没有完成对一个已经就绪的文件描述符进行IO操作，那么以后每次调用select调用时，还是会将这些文件描述符通知进程。

### epoll
相比于select、poll，epoll提供了三个函数。

	int epoll_create(int size);  //创建一个epoll句柄，size告诉内核监听的数量多大。
	int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；		//注册监听的事件类型。
	int epoll_wait(int epfd,struct epoll_event *event,int maxevent,int timeout);	//等待事件的发生生。

**epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。**

#### epoll_create
epoll_create创建一个epoll句柄，size用来告诉内核监听的数量多大，它会占用一个fd值。

epoll可以监听大量的fd，在内存为4G的情况下，可以达到10万多个文件描述符。
#### epoll_ctl
epoll的事件注册函数，它不同于select是在监听时告诉内核要监听什么类型的事件，而是先注册要监听的事件类型。

第一个参数为epoll的返回值，第二个参数op表示动作，用三个宏表示：

- EPOLL_CTL_ADD：注册新的fd到epfd中。
- EPOLL_CTL_MOD：修改已经注册的fd的监听事件。
- EPOLL_CTL_DEL：从epfd中删除一个fd。

第三个参数为监听的fd，第四个参数epoll_event是告诉内核需要监听什么事。

	struct epoll_event{
		_unit32_t events;		/*Epoll events*/
		epoll_data_t data;		/*User data variable*/
	}

events可以是以下几个宏的集合：

- EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
- EPOLLOUT：表示对应的文件描述符可以写；
- EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
- EPOLLERR：表示对应的文件描述符发生错误；
- EPOLLHUP：表示对应的文件描述符被挂断；
- EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
- EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里

#### epoll_wait
等待事件的产生，类似于select调用，参数events用来从内核得到事件的集合，maxevents告诉内核这个events有多大，这个maxevents的值不能大于创建epoll_cteate时的size,参数timeout是超时时间。该函数返回需要处理的事件数目，如返回0表示已超时。


**注意**：epoll在内核中维护了一个事件表（mmap的红黑树？），无需像Select那样，每次调用都需要重复的将文件描述符集合传入内核。当epoll_wait 检测到事件，那就将这些事件从事件表复制到第二个参数epoll_event * events中，返回这些就绪的事件，所以在用户空间中，只需遍历这些就绪事件即可，而不是像select那样还需要在用户空间遍历所有的fd，找出发生事件就绪的fd。


# 参考：
- [cs-notes-socket](https://github.com/CyC2018/CS-Notes/blob/master/notes/Socket.md#%E4%B8%80io-%E6%A8%A1%E5%9E%8B)
- [聊聊IO多路复用之select、poll、epoll详解](https://www.jianshu.com/p/dfd940e7fca2)
- [IO多路复用与Go网络库的实现](https://ninokop.github.io/2018/02/18/IO%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8%E4%B8%8EGo%E7%BD%91%E7%BB%9C%E5%BA%93%E7%9A%84%E5%AE%9E%E7%8E%B0/)
- [IO复用模式--select、poll、epoll详解](https://blog.csdn.net/u013679744/article/details/79188768)
