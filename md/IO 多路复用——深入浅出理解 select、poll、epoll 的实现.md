> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/367591714)

**前言：**
-------

在 linux 系统中，实际上所有的 I/O 设备都被抽象为了文件这个概念，一切皆文件，Everything is File，磁盘、网络数据、终端，甚至进程间通信工具管道 pipe 等都被当做文件对待。

在了解多路复用 select、poll、epoll 实现之前，我们先简单回忆复习以下两个概念：

一、什么是多路复用：
----------

*   多路: 指的是多个 socket 网络连接;
*   复用: 指的是复用一个线程、使用一个线程来检查多个文件描述符（Socket）的就绪状态
*   多路复用主要有三种技术：select，poll，epoll。epoll 是最新的, 也是目前最好的多路复用技术；

二、五种 IO 模型：
-----------

```
[1]blockingIO - 阻塞IO
[2]nonblockingIO - 非阻塞IO
[3]signaldrivenIO - 信号驱动IO
[4]asynchronousIO - 异步IO
[5]IOmultiplexing - IO多路复用
```

### **阻塞式 I/O 模型：**

![](https://pic3.zhimg.com/v2-e340321a0d0e8bc7311e5dad107624ae_r.jpg)

进程 / 线程在从调用 recvfrom 开始到它返回的整段时间内是被阻塞的，recvfrom 成功返回后，应用进程 / 线程开始处理数据报。主要特点是进程阻塞挂起不消耗 CPU 资源，能及时响应每个操作；实现难度低，适用并发量小的网络应用开发，不适用并发量大的应用，因为一个请求 IO 会阻塞进程，所以每请求分配一个处理进程（线程）去响应，系统开销大。

### **非阻塞式 I/O 模型：**

![](https://pic1.zhimg.com/v2-9fdf0ff499b2b514d1238b1fca212638_r.jpg)

进程发起 IO 系统调用后，如果内核缓冲区没有数据，需要到 IO 设备中读取，进程返回一个错误而不会被阻塞；进程发起 IO 系统调用后，如果内核缓冲区有数据，内核就会把数据返回进程。

*   进程轮询（重复）调用，消耗 CPU 的资源；
*   实现难度低、开发应用相对阻塞 IO 模式较难；
*   适用并发量较小、且不需要及时响应的网络应用开发；

### 信号驱动 IO**：**

![](https://pic4.zhimg.com/v2-4c502410a82ee33a30e1698589f8d88b_r.jpg)

当进程发起一个 IO 操作，会向内核注册一个信号处理函数，然后进程返回不阻塞；当内核数据就绪时会发送一个信号给进程，进程便在信号处理函数中调用 IO 读取数据。

特点：回调机制，实现、开发应用难度大；

### 异步 IO

![](https://pic1.zhimg.com/v2-044ce7b648d62eaef1c1859cc6703cf8_r.jpg)

当进程发起一个 IO 操作，进程返回（不阻塞），但也不能返回果结；内核把整个 IO 处理完后，会通知进程结果。如果 IO 操作成功则进程直接获取到数据。

特点：

*   不阻塞，数据一步到位；Proactor 模式；
*   需要操作系统的底层支持，LINUX 2.5 版本内核首现，2.6 版本产品的内核标准特性；
*   实现、开发应用难度大；
*   非常适合高性能高并发应用；

### **IO 复用模型**

![](https://pic3.zhimg.com/v2-c3bc4f57375fbd0dbe49692426b84dd2_r.jpg)

大多数文件系统的默认 IO 操作都是缓存 IO。在 Linux 的缓存 IO 机制中，操作系统会将 IO 的数据缓存在文件系统的页缓存（page cache）。也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓存区拷贝到应用程序的地址空间中。这种做法的缺点就是，**需要在应用程序地址空间和内核进行多次拷贝，这些拷贝动作所带来的 CPU 以及内存开销是非常大的**。

至于为什么不能直接让磁盘控制器把数据送到应用程序的地址空间中呢？**最简单的一个原因就是应用程序不能直接操作底层硬件。**

总的来说，IO 分两阶段：

1) 数据准备阶段

2) 内核空间复制回用户进程缓冲区阶段。如下图：

![](https://pic3.zhimg.com/v2-89162cb4d5cc7fe4071ade36d3000d8a_r.jpg)

三、I/O 多路复用之 select、poll、epoll 详解
--------------------------------

目前支持 I/O 多路复用的系统调用有`select，pselect，poll，epoll。`与多进程和多线程技术相比，`I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程`，也不必维护这些进程 / 线程，从而大大减小了系统的开销。

I/O 多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但 select，poll，epoll 本质上都是同步 I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步 I/O 则无需自己负责进行读写，异步 I/O 的实现会负责把数据从内核拷贝到用户空间

### **select**

```
int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

*   readfds：内核检测该集合中的 IO 是否可读。如果想让内核帮忙检测某个 IO 是否可读，需要手动把文件描述符加入该集合。
*   writefds：内核检测该集合中的 IO 是否可写。同 readfds，需要手动把文件描述符加入该集合。
*   exceptfds：内核检测该集合中的 IO 是否异常。同 readfds，需要手动把文件描述符加入该集合。
*   nfds：以上三个集合中最大的文件描述符数值 + 1，例如集合是 {0,1,5,10}，那么 maxfd 就是 11
*   timeout：用户线程调用 select 的超时时长。

*   设置成 NULL，表示如果没有 I/O 事件发生，则 select 一直等待下去。
*   设置为非 0 的值，这个表示等待固定的一段时间后从 select 阻塞调用中返回。
*   设置成 0，表示根本不等待，检测完毕立即返回。

函数返回值：

*   大于 0：成功，返回集合中已就绪的 IO 总个数
*   等于 - 1：调用失败
*   等于 0：没有就绪的 IO

从上述的 select 函数声明可以看出，fd_set 本质是一个数组，为了方便我们操作该数组，操作系统提供了以下函数：

```
// 将文件描述符fd从set集合中删除 
void FD_CLR(int fd, fd_set *set); 

// 判断文件描述符fd是否在set集合中 
int  FD_ISSET(int fd, fd_set *set); 

// 将文件描述符fd添加到set集合中 
void FD_SET(int fd, fd_set *set); 

// 将set集合中, 所有文件描述符对应的标志位设置为0
void FD_ZERO(fd_set *set);
```

select 函数监视的文件描述符分 3 类，分别是 writefds、readfds、和 exceptfds，当用户 process 调用 select 的时候，select 会将需要监控的 readfds 集合拷贝到内核空间（假设监控的仅仅是 socket 可读），然后遍历自己监控的 skb(SocketBuffer)，挨个调用 skb 的 poll 逻辑以便检查该 socket 是否有可读事件，遍历完所有的 skb 后，如果没有任何一个 socket 可读，那么 select 会调用 schedule_timeout 进入 schedule 循环，使得 process 进入睡眠。如果在 timeout 时间内某个 socket 上有数据可读了，或者等待 timeout 了，则调用 select 的 process 会被唤醒，接下来 select 就是遍历监控的集合，挨个收集可读事件并返回给用户了，相应的伪码如下：

```
int select(
    int nfds,
    fd_set *readfds,
    fd_set *writefds,
    fd_set *exceptfds,
    struct timeval *timeout);
// nfds:监控的文件描述符集里最大文件描述符加1
// readfds：监控有读数据到达文件描述符集合，传入传出参数
// writefds：监控写数据到达文件描述符集合，传入传出参数
// exceptfds：监控异常发生达文件描述符集合, 传入传出参数
// timeout：定时阻塞监控时间，3种情况
//  1.NULL，永远等下去
//  2.设置timeval，等待固定时间
//  3.设置timeval里时间均为0，检查描述字后立即返回，轮询

/* 
* select服务端伪码
* 首先一个线程不断接受客户端连接，并把socket文件描述符放到一个list里。
*/
while(1) {
  connfd = accept(listenfd);
  fcntl(connfd, F_SETFL, O_NONBLOCK);
  fdlist.add(connfd);
}
/*
* select函数还是返回刚刚提交的list，应用程序依然list所有的fd，只不过操作系统会将准备就绪的文件描述符做上标识，
* 用户层将不会再有无意义的系统调用开销。
*/
struct timeval timeout;
int max = 0;  // 用于记录最大的fd，在轮询中时刻更新即可
// 初始化比特位
FD_ZERO(&read_fd);
while (1) {
    // 阻塞获取 每次需要把fd从用户态拷贝到内核态
    nfds = select(max + 1, &read_fd, &write_fd, NULL, &timeout);
    // 每次需要遍历所有fd，判断有无读写事件发生
    for (int i = 0; i <= max && nfds; ++i) {
        // 只读已就绪的文件描述符，不用过多遍历
        if (i == listenfd) {
            // 这里处理accept事件
            FD_SET(i, &read_fd);//将客户端socket加入到集合中
        }
        if (FD_ISSET(i, &read_fd)) {
            // 这里处理read事件
        }
    }
}
```

下面的动图能更直观的让我们了解 select：

![](https://pic3.zhimg.com/v2-320be0c91e2a376199b1d5eef626758e_r.jpg)

通过上面的 select 逻辑过程分析，相信大家都意识到，select 存在三个问题：

> [1] 每次调用 select，都需要把被监控的 fds 集合从用户态空间拷贝到内核态空间，高并发场景下这样的拷贝会使得消耗的资源是很大的。  
> [2] 能监听端口的数量有限，单个进程所能打开的最大连接数由 FD_SETSIZE 宏定义，监听上限就等于 fds_bits 位数组中所有元素的二进制位总数，其大小是 32 个整数的大小（在 32 位的机器上，大小就是 3232，同理 64 位机器上为 3264），当然我们可以对宏 FD_SETSIZE 进行修改，然后重新编译内核，但是性能可能会受到影响，一般该数和系统内存关系很大，具体数目可以`cat /proc/sys/fs/file-max`察看。32 位机默认 1024 个，64 位默认 2048。

![](https://pic3.zhimg.com/v2-e842f26b011cf5d7cf05cce6b7a8d6e6_r.jpg)

> [3] 被监控的 fds 集合中，只要有一个有数据可读，整个 socket 集合就会被遍历一次调用 sk 的 poll 函数收集可读事件：由于当初的需求是朴素，仅仅关心是否有数据可读这样一个事件，当事件通知来的时候，由于数据的到来是异步的，我们不知道事件来的时候，有多少个被监控的 socket 有数据可读了，于是，只能挨个遍历每个 socket 来收集可读事件了。

### **poll**

poll 的实现和 select 非常相似，只是描述 fd 集合的方式不同。针对 select 遗留的三个问题中（问题 (2) 是 fd 限制问题，问题 (1) 和(3)则是性能问题），poll 只是使用 pollfd 结构而不是 select 的 fd_set 结构，这就解决了 select 的问题(2)fds 集合大小 1024 限制问题。但 poll 和 select 同样存在一个性能缺点就是包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。

```
struct pollfd {
　　 int fd;           /*文件描述符*/
　　 short events;     /*监控的事件*/
　　 short revents;    /*监控事件中满足条件返回的事件*/
};
int poll(struct pollfd *fds, unsigned long nfds, int timeout);
```

函数参数：

*   fds：struct pollfd 类型的数组, 存储了待检测的文件描述符，struct pollfd 有三个成员：
*   fd：委托内核检测的文件描述符
*   events：委托内核检测的 fd 事件（输入、输出、错误），每一个事件有多个取值
*   revents：这是一个传出参数，数据由内核写入，存储内核检测之后的结果
*   nfds：描述的是数组 fds 的大小
*   timeout: 指定 poll 函数的阻塞时长

*   -1：一直阻塞，直到检测的集合中有就绪的 IO 事件，然后解除阻塞函数返回
*   0：不阻塞，不管检测集合中有没有已就绪的 IO 事件，函数马上返回
*   大于 0：表示 poll 调用方等待指定的毫秒数后返回

函数返回值：

*   -1：失败
*   大于 0：表示检测的集合中已就绪的文件描述符的总个数

下面是 poll 的函数原型，poll 改变了 fds 集合的描述方式，使用了 pollfd 结构而不是 select 的 fd_set 结构，使得 poll 支持的 fds 集合限制远大于 select 的 1024。poll 虽然解决了 fds 集合大小 1024 的限制问题，从实现来看。很明显它并没优化大量描述符数组被整体复制于用户态和内核态的地址空间之间，以及个别描述符就绪触发整体描述符集合的遍历的低效问题。poll 随着监控的 socket 集合的增加性能线性下降，使得 poll 也并不适合用于大并发场景。

```
poll服务端实现伪码：
struct pollfd fds[POLL_LEN];
unsigned int nfds=0;
fds[0].fd=server_sockfd;
fds[0].events=POLLIN|POLLPRI;
nfds++;
while {
    res=poll(fds,nfds,-1);
    if(fds[0].revents&(POLLIN|POLLPRI)) {
        //执行accept并加入fds中，nfds++
        if(--res<=0) continue
    }
    //循环之后的fds
    if(fds[i].revents&(POLLIN|POLLERR )) {
        //读操作或处理异常等
        if(--res<=0) continue
    }
}
```

### **epoll**

在 linux 的网络编程中，很长的时间都在使用 select 来做事件触发。在 linux 新的内核中，有了一种替换它的机制，就是 epoll。相比于 select，epoll 最大的好处在于它不会随着监听 fd 数目的增长而降低效率。如前面我们所说，在内核中的 select 实现中，它是采用轮询来处理的，轮询的 fd 数目越多，自然耗时越多。并且，在 linux/posix_types.h 头文件有这样的声明：  
#define __FD_SETSIZE 1024  
表示 select 最多同时监听 1024 个 fd，当然，可以通过修改头文件再重编译内核来扩大这个数目，但这似乎并不治本。

创建一个 epoll 的句柄，size 用来告诉内核这个监听的数目一共有多大。这个参数不同于 select() 中的第一个参数，给出最大监听的 fd+1 的值。需要注意的是，当创建好 epoll 句柄后，它就是会占用一个 fd 值，在 linux 下如果查看 / proc / 进程 id/fd/，是能够看到这个 fd 的，所以在使用完 epoll 后，必须调用 close() 关闭，否则可能导致 fd 被耗尽。

epoll 的接口非常简单，一共就三个函数：

*   epoll_create：创建一个 epoll 句柄
*   epoll_ctl：向 epoll 对象中添加 / 修改 / 删除要管理的连接
*   epoll_wait：等待其管理的连接上的 IO 事件

**epoll_create 函数**

```
int epoll_create(int size);
```

*   **功能：**该函数生成一个 epoll 专用的文件描述符。
*   **参数 size:** 用来告诉内核这个监听的数目一共有多大，参数 size 并不是限制了 epoll 所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议。自从 linux 2.6.8 之后，size 参数是被忽略的，也就是说可以填只有大于 0 的任意值。
*   **返回值：**如果成功，返回 poll 专用的文件描述符，否者失败，返回 - 1。

epoll_create 的源码实现：

```
SYSCALL_DEFINE1(epoll_create1, int, flags)
{
    struct eventpoll *ep = NULL;

    //创建一个 eventpoll 对象
    error = ep_alloc(&ep);
}

//struct eventpoll 的定义
// file：fs/eventpoll.c
struct eventpoll {

    //sys_epoll_wait用到的等待队列
    wait_queue_head_t wq;

    //接收就绪的描述符都会放到这里
    struct list_head rdllist;

    //每个epoll对象中都有一颗红黑树
    struct rb_root rbr;

    ......
}
static int ep_alloc(struct eventpoll **pep)
{
    struct eventpoll *ep;

    //申请 epollevent 内存
    ep = kzalloc(sizeof(*ep), GFP_KERNEL);

    //初始化等待队列头
    init_waitqueue_head(&ep->wq);

    //初始化就绪列表
    INIT_LIST_HEAD(&ep->rdllist);

    //初始化红黑树指针
    ep->rbr = RB_ROOT;

    ......
}
```

其中 eventpoll 这个结构体中的几个成员的含义如下：

*   **wq：** 等待队列链表。软中断数据就绪的时候会通过 wq 来找到阻塞在 epoll 对象上的用户进程。
*   **rbr：** 红黑树。为了支持对海量连接的高效查找、插入和删除，eventpoll 内部使用的就是红黑树。通过红黑树来管理用户主进程 accept 添加进来的所有 socket 连接。
*   **rdllist：** 就绪的描述符链表。当有连接就绪的时候，内核会把就绪的连接放到 rdllist 链表里。这样应用进程只需要判断链表就能找出就绪进程，而不用去遍历红黑树的所有节点了。

**epoll_ctl 函数**

```
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

*   **功能：**epoll 的事件注册函数，它不同于 select() 是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。
*   **参数 epfd:** epoll 专用的文件描述符，epoll_create() 的返回值
*   **参数 op:** 表示动作，用三个宏来表示：

1.  EPOLL_CTL_ADD：注册新的 fd 到 epfd 中；
2.  EPOLL_CTL_MOD：修改已经注册的 fd 的监听事件；
3.  EPOLL_CTL_DEL：从 epfd 中删除一个 fd；

*   **参数 fd:** 需要监听的文件描述符
*   **参数 event:** 告诉内核要监听什么事件，struct epoll_event 结构如:
*   **events** **可以是以下几个宏的集合：**
*   EPOLLIN ：表示对应的文件描述符可以读（包括对端 SOCKET 正常关闭）；
*   EPOLLOUT：表示对应的文件描述符可以写；
*   EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
*   EPOLLERR：表示对应的文件描述符发生错误；
*   EPOLLHUP：表示对应的文件描述符被挂断；
*   EPOLLET ：将 EPOLL 设为边缘触发 (Edge Trigger) 模式，这是相对于水平触发 (Level Trigger) 来说的。
*   EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个 socket 的话，需要再次把这个 socket 加入到 EPOLL 队列里
*   **返回值：**0 表示成功，-1 表示失败。

**epoll_wait 函数**

```
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

*   **功能：**等待事件的产生，收集在 epoll 监控的事件中已经发送的事件，类似于 select() 调用。
*   **参数 epfd:** epoll 专用的文件描述符，epoll_create() 的返回值
*   **参数 events:** 分配好的 epoll_event 结构体数组，epoll 将会把发生的事件赋值到 events 数组中（events 不可以是空指针，内核只负责把数据复制到这个 events 数组中，不会去帮助我们在用户态中分配内存）。
*   **参数 maxevents:** maxevents 告之内核这个 events 有多少个 。
*   **参数 timeout:** 超时时间，单位为毫秒，为 -1 时，函数为阻塞。
*   **返回值：**

1.  如果成功，表示返回需要处理的事件数目
2.  如果返回 0，表示已超时
3.  如果返回 - 1，表示失败

```
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <fcntl.h>
#include <stdlib.h>
#include <cassert>
#include <sys/epoll.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include<iostream>
const int MAX_EVENT_NUMBER = 10000; //最大事件数
// 设置句柄非阻塞
int setnonblocking(int fd)
{
    int old_option = fcntl(fd, F_GETFL);
    int new_option = old_option | O_NONBLOCK;
    fcntl(fd, F_SETFL, new_option);
    return old_option;
}

int main(){

    // 创建套接字
    int nRet=0;
    int m_listenfd = socket(PF_INET, SOCK_STREAM, 0);
    if(m_listenfd<0)
    {
        printf("fail to socket!");
        return -1;
    }
    // 
    struct sockaddr_in address;
    bzero(&address, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = htonl(INADDR_ANY);
    address.sin_port = htons(6666);

    int flag = 1;
    // 设置ip可重用
    setsockopt(m_listenfd, SOL_SOCKET, SO_REUSEADDR, &flag, sizeof(flag));
    // 绑定端口号
    int ret = bind(m_listenfd, (struct sockaddr *)&address, sizeof(address));
    if(ret<0)
    {
        printf("fail to bind!,errno :%d",errno);
        return ret;
    }

    // 监听连接fd
    ret = listen(m_listenfd, 200);
    if(ret<0)
    {
        printf("fail to listen!,errno :%d",errno);
        return ret;
    }

    // 初始化红黑树和事件链表结构rdlist结构
    epoll_event events[MAX_EVENT_NUMBER];
    // 创建epoll实例
    int m_epollfd = epoll_create(5);
    if(m_epollfd==-1)
    {
        printf("fail to epoll create!");
        return m_epollfd;
    }



    // 创建节点结构体将监听连接句柄
    epoll_event event;
    event.data.fd = m_listenfd;
    //设置该句柄为边缘触发（数据没处理完后续不会再触发事件，水平触发是不管数据有没有触发都返回事件），
    event.events = EPOLLIN | EPOLLET | EPOLLRDHUP;
    // 添加监听连接句柄作为初始节点进入红黑树结构中，该节点后续处理连接的句柄
    epoll_ctl(m_epollfd, EPOLL_CTL_ADD, m_listenfd, &event);

    //进入服务器循环
    while(1)
    {
        int number = epoll_wait(m_epollfd, events, MAX_EVENT_NUMBER, -1);
        if (number < 0 && errno != EINTR)
        {
            printf( "epoll failure");
            break;
        }
        for (int i = 0; i < number; i++)
        {
            int sockfd = events[i].data.fd;
            // 属于处理新到的客户连接
            if (sockfd == m_listenfd)
            {
                struct sockaddr_in client_address;
                socklen_t client_addrlength = sizeof(client_address);
                int connfd = accept(m_listenfd, (struct sockaddr *)&client_address, &client_addrlength);
                if (connfd < 0)
                {
                    printf("errno is:%d accept error", errno);
                    return false;
                }
                epoll_event event;
                event.data.fd = connfd;
                //设置该句柄为边缘触发（数据没处理完后续不会再触发事件，水平触发是不管数据有没有触发都返回事件），
                event.events = EPOLLIN | EPOLLRDHUP;
                // 添加监听连接句柄作为初始节点进入红黑树结构中，该节点后续处理连接的句柄
                epoll_ctl(m_epollfd, EPOLL_CTL_ADD, connfd, &event);
                setnonblocking(connfd);
            }
            else if (events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR))
            {
                //服务器端关闭连接，
                epoll_ctl(m_epollfd, EPOLL_CTL_DEL, sockfd, 0);
                close(sockfd);
            }
            //处理客户连接上接收到的数据
            else if (events[i].events & EPOLLIN)
            {
                char buf[1024]={0};
                read(sockfd,buf,1024);
                printf("from client :%s");

                // 将事件设置为写事件返回数据给客户端
                events[i].data.fd = sockfd;
                events[i].events = EPOLLOUT | EPOLLET | EPOLLONESHOT | EPOLLRDHUP;
                epoll_ctl(m_epollfd, EPOLL_CTL_MOD, sockfd, &events[i]);
            }
            else if (events[i].events & EPOLLOUT)
            {
                std::string response = "server response \n";
                write(sockfd,response.c_str(),response.length());

                // 将事件设置为读事件，继续监听客户端
                events[i].data.fd = sockfd;
                events[i].events = EPOLLIN | EPOLLRDHUP;
                epoll_ctl(m_epollfd, EPOLL_CTL_MOD, sockfd, &events[i]);
            }
            //else if 可以加管道，unix套接字等等数据
        }
    }


}
```

如下图，可以帮助我们理解的更加丝滑 (/ 手动狗头)：

![](https://pic2.zhimg.com/v2-70f8e9bc1a028d252c01c32329e49341_r.jpg)

### epoll 的边缘触发与水平触发

**水平触发 (LT)**

关注点是数据是否有无，只要读缓冲区不为空，写缓冲区不满，那么 epoll_wait 就会一直返回就绪，水平触发是 epoll 的默认工作方式。

**边缘触发 (ET)**

关注点是变化，只要缓冲区的数据有变化，epoll_wait 就会返回就绪。  
这里的数据变化并不单纯指缓冲区从有数据变为没有数据，或者从没有数据变为有数据，还包括了数据变多或者变少。即当 buffer 长度有变化时，就会触发。  
假设 epoll 被设置为了边缘触发，当客户端写入了 100 个字符，由于缓冲区从 0 变为了 100，于是服务端 epoll_wait 触发一次就绪，服务端读取了 2 个字节后不再读取。这个时候再去调用 epoll_wait 会发现不会就绪，只有当客户端再次写入数据后，才会触发就绪。  
这就导致如果使用 ET 模式，那就必须保证要「一次性把数据读取 & 写入完」，否则会导致数据长期无法读取 / 写入。

**epoll 为什么比 select、poll 更高效？**
-------------------------------

*   epoll 采用红黑树管理文件描述符  
    从上图可以看出，epoll 使用红黑树管理文件描述符，红黑树插入和删除的都是时间复杂度 O(logN)，不会随着文件描述符数量增加而改变。  
    select、poll 采用数组或者链表的形式管理文件描述符，那么在遍历文件描述符时，时间复杂度会随着文件描述的增加而增加。
*   epoll 将文件描述符添加和检测分离，减少了文件描述符拷贝的消耗  
    select&poll 调用时会将全部监听的 fd 从用户态空间拷贝至内核态空间并线性扫描一遍找出就绪的 fd 再返回到用户态。下次需要监听时，又需要把之前已经传递过的文件描述符再读传递进去，增加了拷贝文件的无效消耗，当文件描述很多时，性能瓶颈更加明显。  
    而 epoll 只需要使用 epoll_ctl 添加一次，后续的检查使用 epoll_wait，减少了文件拷贝的消耗。

四、总结
----

select，poll，epoll 都是 IO 多路复用机制，即可以监视多个描述符，一旦某个描述符就绪（读或写就绪），能够通知程序进行相应读写操作。 但 select，poll，epoll 本质上都是**同步 I/O**，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步 I/O 则无需自己负责进行读写，异步 I/O 的实现会负责把数据从内核拷贝到用户空间。

*   select，poll 实现需要自己不断轮询所有 fd 集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而 epoll 其实也需要调用 epoll_wait 不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪 fd 放入就绪链表中，并唤醒在 epoll_wait 中进入睡眠的进程。虽然都要睡眠和交替，但是 select 和 poll 在 “醒着” 的时候要遍历整个 fd 集合，而 epoll 在 “醒着” 的时候只要判断一下就绪链表是否为空就行了，这节省了大量的 CPU 时间。这就是回调机制带来的性能提升。
*   select，poll 每次调用都要把 fd 集合从用户态往内核态拷贝一次，并且要把 current 往设备等待队列中挂一次，而 epoll 只要一次拷贝，而且把 current 往等待队列上挂也只挂一次（在 epoll_wait 的开始，注意这里的等待队列并不是设备等待队列，只是一个 epoll 内部定义的等待队列）。这也能节省不少的开销。

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th></th><th>select</th><th>poll</th><th>epoll</th></tr><tr><td>性能</td><td>随着连接数的增加，性能急剧下降，处理成千上万的并发连接数时，性能很差</td><td>随着连接数的增加，性能急剧下降，处理成千上万的并发连接数时，性能很差</td><td>随着连接数的增加，性能基本没有变化</td></tr><tr><td>连接数</td><td>一般 1024</td><td>无限制</td><td>无限制</td></tr><tr><td>内存拷贝</td><td>每次调用 select 拷贝</td><td>每次调用 poll 拷贝</td><td>fd 首次调用 epoll_ctl 拷贝，每次调用 epoll_wait 不拷贝</td></tr><tr><td>数据结构</td><td>bitmap</td><td>数组</td><td>红黑树</td></tr><tr><td>内在处理机制</td><td>线性轮询</td><td>线性轮询</td><td>FD 挂在红黑树，通过事件回调 callback</td></tr><tr><td>时间复杂度</td><td>O(n)</td><td>O(n)</td><td>O(1)</td></tr></tbody></table>