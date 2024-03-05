> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/jiushimanya/article/details/82684525)

里面有疑问或者不正确的地方可以给我留言  
对 TCP/IP、UDP、[Socket](https://so.csdn.net/so/search?q=Socket&spm=1001.2101.3001.7020) 编程这些词你不会很陌生吧？随着网络技术的发展，这些词充斥着我们的耳朵。那么我想问：

1.  什么是 TCP/IP、UDP？
2.  Socket 在哪里呢？
3.  Socket 是什么呢？
4.  你会使用它们吗？

#### 什么是 TCP/IP、UDP？

TCP/IP（Transmission Control Protocol/Internet Protocol）即传输控制协议 / 网间协议，是一个工业标准的协议集，它是为广域网（WANs）设计的。  
UDP（User Data Protocol，用户数据报协议）是与 TCP 相对应的协议。它是属于 TCP/IP 协议族中的一种。  
这里有一张图，表明了这些协议的关系。  
![](https://img-blog.csdn.net/20180913104811138?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppdXNoaW1hbnlh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
TCP/IP 协议族包括运输层、网络层、链路层。现在你知道 TCP/IP 与 UDP 的关系了吧。  

#### Socket 在哪里呢？  

在图 1 中，我们没有看到 Socket 的影子，那么它到底在哪里呢？还是用图来说话，一目了然。  
![](https://img-blog.csdn.net/2018091310482621?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppdXNoaW1hbnlh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
原来 Socket 在这里。  

#### Socket 是什么呢？  

Socket 是应用层与 TCP/IP 协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket 其实就是一个门面模式，它把复杂的 TCP/IP 协议族隐藏在 Socket 接口后面，对用户来说，一组简单的接口就是全部，让 Socket 去组织数据，以符合指定的协议。  

##### 你会使用它们吗？  

前人已经给我们做了好多的事了，网络间的通信也就简单了许多，但毕竟还是有挺多工作要做的。以前听到 Socket 编程，觉得它是比较高深的编程知识，但是只要弄清 Socket 编程的工作原理，神秘的面纱也就揭开了。  
一个生活中的场景。你要打电话给一个朋友，先拨号，朋友听到电话铃声后提起电话，这时你和你的朋友就建立起了连接，就可以讲话了。等交流结束，挂断电话结束此次交谈。 生活中的场景就解释了这工作原理，也许 TCP/IP 协议族就是诞生于生活中，这也不一定。  
![](https://img-blog.csdn.net/20180913104844708?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppdXNoaW1hbnlh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

先从服务器端说起。服务器端先初始化 Socket，然后与端口绑定 (bind)，对端口进行监听 (listen)，调用 accept 阻塞，等待客户端连接。在这时如果有个客户端初始化一个 Socket，然后连接服务器 (connect)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束。

============================================

我们深谙信息交流的价值，那网络中进程之间如何通信，如我们每天打开浏览器浏览网页 时，浏览器的进程怎么与 web 服务器通信的？当你用 QQ 聊天时，QQ 进程怎么与服务器或你好友所在的 QQ 进程通信？这些都得靠 socket？那什么是 socket？socket 的类型有哪些？还有 socket 的基本函数，这些都是本文想介绍的。本文的主要内容如下：

1、网络中进程之间如何通信？  
2、Socket 是什么？  
3、socket 的基本操作  
3.1、socket() 函数  
3.2、bind() 函数  
3.3、listen()、connect() 函数  
3.4、accept() 函数  
3.5、read()、write() 函数等  
3.6、close() 函数  
4、socket 中 TCP 的三次握手建立连接详解  
5、socket 中 TCP 的四次握手释放连接详解  
6、一个例子  

### 1、网络中进程之间如何通信？  

本地的进程间通信（IPC）有很多种方式，但可以总结为下面 4 类：

消息传递（管道、FIFO、消息队列）  
同步（互斥量、条件变量、读写锁、文件和写记录锁、信号量）  
共享内存（匿名的和具名的）  
远程过程调用（Solaris 门和 Sun RPC）  
但这些都不是本文的主题！我们要讨论的是网络中进程之间如何通信？首要解决的问题是如何唯一标识一个进程，否则通信无从谈起！在本地可以通过进程 PID 来唯一标识一个进程，但是在网络中这是行不通的。其实 TCP/IP 协议族已经帮我们解决了这个问题，网络层的 “ip 地址” 可以唯一标识网络中的主机，而传输层的 “协议 + 端口” 可以唯一标识主机中的应用程序（进程）。这样利用三元组（ip 地址，协议，端口）就可以标识网络的进程了，网络中的进程通信就可以利用这个标志与其它进程进行交互。

使用 TCP/IP 协议的应用程序通常采用应用编程接口：UNIX BSD 的套接字（socket）和 UNIX System V 的 TLI（已经被淘汰），来实现网络进程之间的通信。就目前而言，几乎所有的应用程序都是采用 socket，而现在又是网络时代，网络中进程通信是无处不在，这就是我为什么说 “一切皆 socket”。

### 2、什么是 Socket？  

上面我们已经知道网络中的进程是通过 socket 来通信的，那什么是 socket 呢？socket 起源于 Unix，而 Unix/Linux 基本哲学之一就是 “一切皆文件”，都可以用“打开 open –> 读写 write/read –> 关闭 close” 模式来操作。我的理解就是 Socket 就是该模式的一个实现，socket 即是一种特殊的文件，一些 socket 函数就是对其进行的操作（读 / 写 IO、打开、关闭），这些函数我们在后面进行介绍。

socket 一词的起源  
在组网领域的首次使用是在 1970 年 2 月 12 日发布的文献 IETF RFC33 中发现的，撰写者为 Stephen Carr、Steve Crocker 和 Vint Cerf。根据美国计算机历史博物馆的记载，Croker 写道：“命名空间的元素都可称为套接字接口。一个套接字接口构成一个连接的一端，而一个连接可完全由一对套接字接口规定。” 计算机历史博物馆补充道：“这比 BSD 的套接字接口定义早了大约 12 年。”

### 3、socket 的基本操作  

既然 socket 是 “open—write/read—close” 模式的一种实现，那么 socket 就提供了这些操作对应的函数接口。下面以 TCP 为例，介绍几个基本的 socket 接口函数。

#### 3.1、socket() 函数  

int socket(int domain, int type, int protocol);  
socket 函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而 socket() 用于创建一个 socket 描述符（socket descriptor），它唯一标识一个 socket。这个 socket 描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。

正如可以给 fopen 的传入不同参数值，以打开不同的文件。创建 socket 的时候，也可以指定不同的参数创建不同的 socket 描述符，socket 函数的三个参数分别为：

domain：即协议域，又称为协议族（family）。常用的协议族有，AF_INET、AF_INET6、AF_LOCAL（或称 AF_UNIX，Unix 域 socket）、AF_ROUTE 等等。协议族决定了 socket 的地址类型，在通信中必须采用对应的地址，如 AF_INET 决定了要用 ipv4 地址（32 位的）与端口号（16 位的）的组合、AF_UNIX 决定了要用一个绝对路径名作为地址。  
type：指定 socket 类型。常用的 socket 类型有，SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET 等等（socket 的类型有哪些？）。  
protocol：故名思意，就是指定协议。常用的协议有，IPPROTO_TCP、IPPTOTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC 等，它们分别对应 TCP 传输协议、UDP 传输协议、STCP 传输协议、TIPC 传输协议（这个协议我将会单独开篇讨论！）。  
注意：并不是上面的 type 和 protocol 可以随意组合的，如 SOCK_STREAM 不可以跟 IPPROTO_UDP 组合。当 protocol 为 0 时，会自动选择 type 类型对应的默认协议。

当我们调用 socket 创建一个 socket 时，返回的 socket 描述字它存在于协议族（address family，AF_XXX）空间中，但没有一个具体的地址。如果想要给它赋值一个地址，就必须调用 bind() 函数，否则就当调用 connect()、listen() 时系统会自动随机分配一个端口。

#### 3.2、bind() 函数  

正如上面所说 bind() 函数把一个地址族中的特定地址赋给 socket。例如对应 AF_INET、AF_INET6 就是把一个 ipv4 或 ipv6 地址和端口号组合赋给 socket。

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);  
函数的三个参数分别为：

sockfd：即 socket 描述字，它是通过 socket() 函数创建了，唯一标识一个 socket。bind() 函数就是将给这个描述字绑定一个名字。  
addr：一个 const struct sockaddr * 指针，指向要绑定给 sockfd 的协议地址。这个地址结构根据地址创建 socket 时的地址协议族的不同而不同，如 ipv4 对应的是：  
struct sockaddr_in {  
sa_family_t sin_family;  
in_port_t sin_port;  
struct in_addr sin_addr;  
};

struct in_addr {  
uint32_t s_addr;  
};  
ipv6 对应的是：  
struct sockaddr_in6 {  
sa_family_t sin6_family;  
in_port_t sin6_port;  
uint32_t sin6_flowinfo;  
struct in6_addr sin6_addr;  
uint32_t sin6_scope_id;  
};

struct in6_addr {  
unsigned char s6_addr[16];  
};  
Unix 域对应的是：  
#define UNIX_PATH_MAX 108

struct sockaddr_un {  
sa_family_t sun_family;  
char sun_path[UNIX_PATH_MAX];  
};  
addrlen：对应的是地址的长度。  
通常服务器在启动的时候都会绑定一个众所周知的地址（如 ip 地址 + 端口号），用于提供服务，客户就可以通过它来接连服务器；而客户端就不用指定，有系统自动分配一个端口号和自身的 ip 地址组合。这就是为什么通常服务器端在 listen 之前会调用 bind()，而客户端就不会调用，而是在 connect() 时由系统随机生成一个。

网络字节序与主机字节序  
主机字节序就是我们平常说的大端和小端模式：不同的 CPU 有不同的字节序类型，这些字节序是指整数在内存中保存的顺序，这个叫做主机序。引用标准的 Big-Endian 和 Little-Endian 的定义如下：

　　a) Little-Endian 就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。

　　b) Big-Endian 就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。

网络字节序：4 个字节的 32 bit 值以下面的次序传输：首先是 0～7bit，其次 8～15bit，然后 16～23bit，最后是 24~31bit。这种传输次序称作大端字节序。由于 TCP/IP 首部中所有的二进制整数在网络中传输时都要求以这种次序，因此它又称作网络字节序。字节序，顾名思义字节的顺序，就是大于一个字节类型的数据在内存中的存放顺序，一个字节的数据没有顺序的问题了。

所以： 在将一个地址绑定到 socket 的时候，请先将主机字节序转换成为网络字节序，而不要假定主机字节序跟网络字节序一样使用的是 Big-Endian。由于 这个问题曾引发过血案！公司项目代码中由于存在这个问题，导致了很多莫名其妙的问题，所以请谨记对主机字节序不要做任何假定，务必将其转化为网络字节序再 赋给 socket。

#### 3.3、listen()、connect() 函数  

如果作为一个服务器，在调用 socket()、bind() 之后就会调用 listen() 来监听这个 socket，如果客户端这时调用 connect() 发出连接请求，服务器端就会接收到这个请求。

int listen(int sockfd, int backlog);  
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);  
listen 函数的第一个参数即为要监听的 socket 描述字，第二个参数为相应 socket 可以排队的最大连接个数。socket() 函数创建的 socket 默认是一个主动类型的，listen 函数将 socket 变为被动类型的，等待客户的连接请求。

connect 函数的第一个参数即为客户端的 socket 描述字，第二参数为服务器的 socket 地址，第三个参数为 socket 地址的长度。客户端通过调用 connect 函数来建立与 TCP 服务器的连接。

#### 3.4、accept() 函数  

TCP 服务器端依次调用 socket()、bind()、listen() 之后，就会监听指定的 socket 地址了。TCP 客户端依次调用 socket()、connect() 之后就想 TCP 服务器发送了一个连接请求。TCP 服务器监听到这个请求之后，就会调用 accept() 函数取接收请求，这样连接就建立好了。之后就可以开始网络 I/O 操作了，即类同于普通文件的读写 I/O 操作。

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);  
accept 函数的第一个参数为服务器的 socket 描述字，第二个参数为指向 struct sockaddr * 的指针，用于返回客户端的协议地址，第三个参数为协议地址的长度。如果 accpet 成功，那么其返回值是由内核自动生成的一个全新的描述字，代表与返回客户的 TCP 连接。

注意：accept 的第一个参数为服务器的 socket 描述字，是服务器开始调用 socket() 函数生成的，称为监听 socket 描述字；而 accept 函数返回的是已连接的 socket 描述字。一个服务器通常通常仅仅只创建一个监听 socket 描述字，它在该服务器的生命周期内一直存在。内核为每个由服务器进程接受的客户连接创建了一个已连接 socket 描述字，当服务器完成了对某个客户的服务，相应的已连接 socket 描述字就被关闭。

#### 3.5、read()、write() 等函数  

万事具备只欠东风，至此服务器与客户已经建立好连接了。可以调用网络 I/O 进行读写操作了，即实现了网咯中不同进程之间的通信！网络 I/O 操作有下面几组：

read()/write()  
recv()/send()  
readv()/writev()  
recvmsg()/sendmsg()  
recvfrom()/sendto()  
我推荐使用 recvmsg()/sendmsg() 函数，这两个函数是最通用的 I/O 函数，实际上可以把上面的其它函数都替换成这两个函数。它们的声明如下：

```
#include 

   ssize_t read(int fd, void *buf, size_t count);
   ssize_t write(int fd, const void *buf, size_t count);

   #include 
   #include 

   ssize_t send(int sockfd, const void *buf, size_t len, int flags);
   ssize_t recv(int sockfd, void *buf, size_t len, int flags);

   ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                  const struct sockaddr *dest_addr, socklen_t addrlen);
   ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                    struct sockaddr *src_addr, socklen_t *addrlen);

   ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
   ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

read 函数是负责从 fd 中读取内容. 当读成功时，read 返回实际所读的字节数，如果返回的值是 0 表示已经读到文件的结束了，小于 0 表示出现了错误。如果错误为 EINTR 说明读是由中断引起的，如果是 ECONNREST 表示网络连接出了问题。

write 函数将 buf 中的 nbytes 字节内容写入文件描述符 fd. 成功时返回写的字节 数。失败时返回 - 1，并设置 errno 变量。在网络程序中，当我们向套接字文件描述符写时有俩种可能。1)write 的返回值大于 0，表示写了部分或者是 全部的数据。2) 返回的值小于 0，此时出现了错误。我们要根据错误类型来处理。如果错误为 EINTR 表示在写的时候出现了中断错误。如果为 EPIPE 表示 网络连接出现了问题 (对方已经关闭了连接)。

其它的我就不一一介绍这几对 I/O 函数了，具体参见 man 文档或者 baidu、Google，下面的例子中将使用到 send/recv。

#### 3.6、close() 函数  

在服务器与客户端建立连接之后，会进行一些读写操作，完成了读写操作就要关闭相应的 socket 描述字，好比操作完打开的文件要调用 fclose 关闭打开的文件。

#include  
int close(int fd);  
close 一个 TCP socket 的缺省行为时把该 socket 标记为以关闭，然后立即返回到调用进程。该描述字不能再由调用进程使用，也就是说不能再作为 read 或 write 的第一个参数。

注意：close 操作只是使相应 socket 描述字的引用计数 - 1，只有当引用计数为 0 的时候，才会触发 TCP 客户端向服务器发送终止连接请求。

### 4、socket 中 TCP 的三次握手建立连接详解  

我们知道 tcp 建立连接要进行 “三次握手”，即交换三个分组。大致流程如下：

客户端向服务器发送一个 SYN J  
服务器向客户端响应一个 SYN K，并对 SYN J 进行确认 ACK J+1  
客户端再想服务器发一个确认 ACK K+1  
只有就完了三次握手，但是这个三次握手发生在 socket 的那几个函数中呢？请看下图：

![](https://img-blog.csdn.net/20180913105048585?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppdXNoaW1hbnlh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

图 1、socket 中发送的 TCP 三次握手

从图中可以看出，当客户端调用 connect 时，触发了连接请求，向服务器发送了 SYN J 包，这时 connect 进入阻塞状态；服务器监听到连接请求，即收到 SYN J 包，调用 accept 函数接收请求向客户端发送 SYN K ，ACK J+1，这时 accept 进入阻塞状态；客户端收到服务器的 SYN K ，ACK J+1 之后，这时 connect 返回，并对 SYN K 进行确认；服务器收到 ACK K+1 时，accept 返回，至此三次握手完毕，连接建立。

总结：客户端的 connect 在三次握手的第二个次返回，而服务器端的 accept 在三次握手的第三次返回。

### 5、socket 中 TCP 的四次握手释放连接详解  

上面介绍了 socket 中 TCP 的三次握手建立过程，及其涉及的 socket 函数。现在我们介绍 socket 中的四次握手释放连接的过程，请看下图：

![](https://img-blog.csdn.net/20180913105058539?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppdXNoaW1hbnlh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

图 2、socket 中发送的 TCP 四次握手

图示过程如下：

某个应用进程首先调用 close 主动关闭连接，这时 TCP 发送一个 FIN M；  
另一端接收到 FIN M 之后，执行被动关闭，对这个 FIN 进行确认。它的接收也作为文件结束符传递给应用进程，因为 FIN 的接收意味着应用进程在相应的连接上再也接收不到额外数据；  
一段时间之后，接收到文件结束符的应用进程调用 close 关闭它的 socket。这导致它的 TCP 也发送一个 FIN N；  
接收到这个 FIN 的源发送端 TCP 对它进行确认。  
这样每个方向上都有一个 FIN 和 ACK。

### 6、下面给出实现的一个实例  

首先，先给出实现的截图

![](https://img-blog.csdn.net/20180913105111295?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppdXNoaW1hbnlh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

服务器端代码如下：  
[cpp] view plaincopyprint?

```
#include "InitSock.h"
#include
#include
using namespace std;
CInitSock initSock; // 初始化Winsock库
int main()
{
// 创建套节字
SOCKET sListen = ::socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
//用来指定套接字使用的地址格式，通常使用AF_INET
//指定套接字的类型，若是SOCK_DGRAM，则用的是udp不可靠传输
//配合type参数使用，指定使用的协议类型（当指定套接字类型后，可以设置为0，因为默认为UDP或TCP）
if(sListen == INVALID_SOCKET)
{
printf("Failed socket() \n");
return 0;
}
// 填充sockaddr_in结构 ,是个结构体
sockaddr_in sin;
sin.sin_family = AF_INET;
sin.sin_port = htons(4567); //1024 ~ 49151：普通用户注册的端口号
sin.sin_addr.S_un.S_addr = INADDR_ANY;
// 绑定这个套节字到一个本地地址
if(::bind(sListen, (LPSOCKADDR)&sin, sizeof(sin)) == SOCKET_ERROR)
{
printf("Failed bind() \n");
return 0;
}
// 进入监听模式
//2指的是，监听队列中允许保持的尚未处理的最大连接数
if(::listen(sListen, 2) == SOCKET_ERROR)
{
printf("Failed listen() \n");
return 0;
}
// 循环接受客户的连接请求
sockaddr_in remoteAddr;
int nAddrLen = sizeof(remoteAddr);
SOCKET sClient = 0;
char szText[] = " TCP Server Demo! \r\n";
while(sClient==0)
{
// 接受一个新连接
//（(SOCKADDR*)&remoteAddr）一个指向sockaddr_in结构的指针，用于获取对方地址
sClient = ::accept(sListen, (SOCKADDR*)&remoteAddr, &nAddrLen);
if(sClient == INVALID_SOCKET)
{
printf("Failed accept()");
}
printf("接受到一个连接：%s \r\n", inet_ntoa(remoteAddr.sin_addr));
continue ;
}
while(TRUE)
{
// 向客户端发送数据
gets(szText) ;
::send(sClient, szText, strlen(szText), 0);
// 从客户端接收数据
char buff[256] ;
int nRecv = ::recv(sClient, buff, 256, 0);
if(nRecv > 0)
{
buff[nRecv] = '\0';
printf(" 接收到数据：%s\n", buff);
}
}
// 关闭同客户端的连接
::closesocket(sClient);
// 关闭监听套节字
::closesocket(sListen);
return 0;
}
```

客户端代码：

```
#include "InitSock.h"
#include
#include
using namespace std;
CInitSock initSock; // 初始化Winsock库
int main()
{
// 创建套节字
SOCKET s = ::socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
if(s == INVALID_SOCKET)
{
printf(" Failed socket() \n");
return 0;
}
// 也可以在这里调用bind函数绑定一个本地地址
// 否则系统将会自动安排
// 填写远程地址信息
sockaddr_in servAddr;
servAddr.sin_family = AF_INET;
servAddr.sin_port = htons(4567);
// 注意，这里要填写服务器程序（TCPServer程序）所在机器的IP地址
// 如果你的计算机没有联网，直接使用127.0.0.1即可
servAddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
if(::connect(s, (sockaddr*)&servAddr, sizeof(servAddr)) == -1)
{
printf(" Failed connect() \n");
return 0;
}
char buff[256];
char szText[256] ;
while(TRUE)
{
//从服务器端接收数据
int nRecv = ::recv(s, buff, 256, 0);
if(nRecv > 0)
{
buff[nRecv] = '\0';
printf("接收到数据：%s\n", buff);
}
// 向服务器端发送数据
gets(szText) ;
szText[255] = '\0';
::send(s, szText, strlen(szText), 0) ;
}
// 关闭套节字
::closesocket(s);
return 0;
}

封装的InitSock.h
[cpp] view plaincopyprint?
#include
#include
#include
#include
#pragma comment(lib, "WS2_32") // 链接到WS2_32.lib
class CInitSock
{
public:
CInitSock(BYTE minorVer = 2, BYTE majorVer = 2)
{
// 初始化WS2_32.dll
WSADATA wsaData;
WORD sockVersion = MAKEWORD(minorVer, majorVer);
if(::WSAStartup(sockVersion, &wsaData) != 0)
{
exit(0);
}
}
~CInitSock()
{
::WSACleanup();
}
};
```