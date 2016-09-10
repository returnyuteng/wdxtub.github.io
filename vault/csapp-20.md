title: 深入理解计算机系统 第 20 课 Network Programming I
categories:
- Technique
toc: true
date: 2016-04-06 07:33:58
tags:
- CMU
- 组成原理
- 计算机
- 网络
---

网络已经成为了我们『智能』生活中最重要的一部分，这两节课我们会通过 C API 来了解具体的网络相关编程（用更高层次的封装只会更轻松！）

<!-- more -->

---

## 网络架构

计算机网络的知识可谓是非常『保值』的，因为这么多基础设备还在运行着，基本机制在短时间内很难改变，关于网络的另一个版本的讲解在[这里](./2016/02/10/internet-protocol/)，我觉得也非常不错，大家感兴趣可以看看。

客户端-服务器模型是网络应用最广泛使用的模型，作为耳熟能详的概念，这里不多提，相信通过下图就能大致明白：

![](/images/14599482011858.jpg)

网络相关的处理，都是通过网络适配器来完成的，具体在硬件上为：

![Hardware Organization of a Network Host](/images/14599482515878.jpg)

根据应用范围和架构层级，可以分成三个部分：

+ SAN - System Area Network
    + Switched Ethernet, Quadrics QSW, ...
+ LAN - Local Area Network
    + Ethernet, ..
+ WAN - Wide Area Network
    + High speed point-to-point phone lines

> 最底层 - Ethernet Segment

Ethernet segment consists of a collection of **hosts** connected by wires (twisted pairs) to a **hub**.

通常范围是房间或一层楼

![Ethernet Segment](/images/14599485111822.jpg)

+ 每个 Ethernet 适配器有一个唯一的 48 位的地址（也就是 MAC 地址），例如 `00:16:ea:e3:54:e6`
+ 不同主机间发送的数据称为帧(frame)
+ Hub 会把每个端口发来的所有数据复制到其他的端口
    + 所有的主机都可以看到所有的数据（注意安全问题）

> 下一层 - Bridged Ethernet Segment

![Bridged Ethernet Segment](/images/14599487658645.jpg)


通常范围是一层楼，通过不同的 bridge 来连接不同的 ethernet segment。Bridge 知道从某端口出发可达的主机，并有选择的在端口间复制数据。

为了从概念上简化，我们可以认为，所有的 hub, bridge 可以抽象为一条线，如下图所示：

![Conceptual View of LANs](/images/14599488669806.jpg)

> 下一层 - internets

不同的（也许不兼容）的 LAN 可以通过 router 来进行物理上的连接，这样连接起来的网络称为 internet（注意是小写，大写的 Internet 可以认为是最著名的 internet）

![internets](/images/14599491906184.jpg)

> internet 的逻辑结构

![Logical Structure of an internet](/images/14599492763163.jpg)

+ Ad hoc interconnection of networks
    + 没有特定的拓扑结构
    + 不同的 router 和 link 差异可能很大
+ 通过在不同的网络间跳转来传递 packet
    + Router 是不同网络间的连接
    + 不同的 packet 可能会走不同的路线

## 网络协议

在不同的 LAN 和 WAN 中传输数据，就要守规矩，这个规矩就是协议。协议负责做的事情有：

+ 提供 naming scheme
    + 定义 host address 格式
    + 每个主机和路由器都至少有一个独立的 internet 地址
+ 提供 delivery mechanism
    + 定义了标准的传输单元 - packet
    + Packet 包含 header 和 payload
        + header 包括 packet size, source 和 destination address
        + payload 包括需要传输的数据  

在这样的协议下，具体的数据传输是：

![Transferring internet Data Via Encapsulation](/images/14599502534733.jpg)

PH = Internet packet header, FH = LAN frame header

> Globle IP Internet(upper case)

Internet 是 internet 最为著名的例子。主要基于 TCP/IP 协议族：

+ IP (Internet Protocal)
    + Provides **basic naming scheme** and unreliable **delivery capability** of packets (datagrams) from **host-to-host**
+ UDP (Unreliable Datagram Protocol)
    + Uses IP to provide **unreliable** datagram delivery from **process-to-process**
+ TCP (Transmission Control Protocol)
    + Uses IP to provide **reliable** byte streams from **process-to-process** over **connections**

Accessed via a mix of Unix file I/O and functions from **sockets interface**.（很多东西不是很好翻译，用原文比较准确）

![Hardware and Software Organization of an Internet Application](/images/14599506938536.jpg)

## Internet 的程序员视角

+ 主机有 32 位的 IP 地址 - 23.235.46.133
    + IPv4 - 32 位地址，IPv6 - 128 位地址
+ IP 地址被映射到域名 - 23.235.46.133 映射到 www.wdxtub.com
+ 不同主机之间的进程，可以通过 connection 来交换数据

> IP 地址

我们会用一个叫做 IP address struct 的东西来存储，并且 IP 地址是以 network byte order（也就是大端）来进行存储的

![IP Address](/images/14599515895838.jpg)

为了方便读，一般用下图的形式来进行表示：

![](/images/14599516332783.jpg)

具体的转换可以使用 `getaddrinfo` 和 `getnameinfo` 函数

> Internet 域名

![Internet Domain Names](/images/14599516793840.jpg)

这里主要需要了解的就是 Domain Naming System(DNS) 的概念，用来做 IP 地址到域名的映射。具体可以用 `nslookup` 命令来查看，下面是一些例子

```bash
$ hostname
wdxtub.local

$ nslookup www.wdxtub.com
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
www.wdxtub.com	canonical name = wdxtub.github.io.
wdxtub.github.io	canonical name = github.map.fastly.net.
Name:	github.map.fastly.net
Address: 23.235.39.133

$ nslookup www.twitter.com
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
www.twitter.com	canonical name = twitter.com.
Name:	twitter.com
Address: 199.16.156.6
Name:	twitter.com
Address: 199.16.156.198
Name:	twitter.com
Address: 199.16.156.230
Name:	twitter.com
Address: 199.16.156.70
```

> Internet Connections

客户端和服务器通过 connection 来发送字节流，特点是：

+ Point-to-point: 连接一对进程
+ Full-duplex: 数据同时可以在两个方向流动
+ Reliable: 字节的发送的顺序和收到的一致

Socket 则可以认为是 connection 的 endpoint，socket 地址是一个 `IPaddress:port` 对。

Port（端口）是一个 16 位的整数，用来标识不同的进程：

+ Ephemeral port: Assigned automatically by client kernel when client makes a connection request
+ Well-known port: Associated with some **service** provided by a server（在 linux 系统上可以在 `/etc/services` 中查看具体的信息）
    + echo server: 7/echo
    + ssh server: 22/ssh
    + email server: 25/smtp
    + web servers: 80/http

> Connection 详解

A connection is uniquely identified by the socket addresses of its endpoints(socket pair - cliaddr:cliport, servaddr: servport)

![Anatomy of a Connection](/images/14599531325600.jpg)

利用不同的端口来连接不同的服务：

![Using Ports to Identify Services](/images/14599534610996.jpg)

> Socket Interface

一系列系统级的函数，和 Unix I/O 配合构造网络应用（在所有的现代操作系统上都可用）。

对于 kernel 来说，socket 是 endpoint of communication；对于应用程序来说，socket 是 file descriptor，用来读写（回忆一下，STDIN 和 STDOUT 也是 file descriptor）。客户端和服务器通过读写对应的 socket descriptor 来进行。

![](/images/14599559864957.jpg)

The main distinction between regular file I/O and socket I/O is how the application "opens" the socket descriptors.

> Generic socket address

![](/images/14599561357752.jpg)

> Internet-specific socket address

实际上占用同样的空间，但是有更详细的信息

![](/images/14599561803255.jpg)

总体的流程为：

![](/images/14599565974716.jpg)

## 常用函数

接下来直接用 PPT 介绍两个重要函数 `getaddrinfo` 和 `getnameinfo`

> getaddrinfo

用来把 hostname, host address, port, service name 的字符串表示转换成 socket address 结构。

![](/images/14599569037369.jpg)

函数原型为：

![](/images/14599569254580.jpg)

这里具体说一下 `result` 这个链表：

![Linked List Returned by `getaddrinfo`](/images/14599569748746.jpg)


 客户端需要遍历这个列表，按顺序访问每个 socket address，直到 `socket` 和 `connect` 函数调用成功。
 
 服务器需要遍历这个列表，直到 `socket` 和  `bind` 函数调用成功

每个 `addrinfo` 结构体为

![addrinfo Struct](/images/14599573080061.jpg)

> getnameinfo

刚好和 `getaddrinfo` 相反，把 socket address 转换成对应的字符串信息，函数原型为

![](/images/14599575876044.jpg)

举个例子

```c
// hostinfo.c
#include "csapp.h"

int main(int argc, char **argv) {
    struct addrinfo *p, *listp, hints;
    char buf[MAXLINE];
    int rc, flags;
    
    // Get a list of addrinfo records
    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_family = AF_INET; // IPv4 only
    hints.ai_socktype = SOCK_STREAM; // Connections only
    if ((rc = getaddrinfo(argv[1], NULL, &hints, &listp)) != 0) {
        fprintf(stderr, "getaddrinfo error: %s\n", gai_strerror(rc));
        exit(1);
    }
    
    // Walk the list and display each IP address
    flags = NI_NUMERICHOST; // Display address instead of name
    for (p = listp; p; p = p->ai_next) {
        getnameinfo(p->ai_addr, p->ai_addrlen, buf, MAXLINE, NULL, 0, flags);
        printf("%s\n", buf);
    }
    
    // Clean up
    freeaddrinfo(listp);
    exit(0);
}
```


