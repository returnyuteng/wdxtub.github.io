title: 深入理解计算机系统 第 21 课 Network Programming II
categories:
- Technique
toc: true
date: 2016-04-06 07:34:02
tags:
- CMU
- 组成原理
- 计算机
- 网络
---

这节课我们来学习如何编写 web 服务器！

<!-- more -->

---

## 架构总览

写服务器，最重要的就是理清思路，上节课我们介绍了诸多概念，尤其是最后提到的 `getaddrinfo` 和 `getnameinfo`，都是我们在搭建过程中必不可少的工具。这里先借下图来介绍具体的实现思路：

![](/images/14599728555354.jpg)

整个的工作流程有 5 步：

1. 开启服务器（`open_listenfd` 函数，做好接收请求的准备）
    + `getaddrinfo`: 设置服务器的相关信息，具体可以参见 图1&2
    + `socket`: 创建 socket descriptor，也就是之后用来读写的 file descriptor
        + `int socket(int domain, int type, int protocol)`
        + 例如 `int clientfd = socket(AF_INET, SOCK_STREAM, 0);`
        + `AF_INET` 表示在使用 32 位 IPv4 地址
        + `SOCK_STREAM` 表示这个 socket 将是 connection 的 endpoint
        + 前面这种写法是协议相关的，建议使用 `getaddrinfo` 生成的参数来进行配置，这样就是协议无关的了
    + `bind`: 请求 kernel 把 socket address 和 socket descriptor 绑定
        + `int bind(int sockfd, SA *addr, socklen_t addrlen);`
        + The process can read bytes that arrive on the connection whose endpoint is `addr` by reading from descriptor `sockfd`
        + Similarly, writes to `sockfd` are transferred along connection whose endpoint is `addr`
        + 最好是用 `getaddrinfo` 生成的参数作为 `addr` 和 `addrlen` 
    + `listen`: 默认来说，我们从 `socket` 函数中得到的 descriptor 默认是 active socket（也就是客户端的连接），调用 `listen` 函数告诉 kernel 这个 socket 是被服务器使用的
        + `int listen(int sockfd, int backlog);`
        + 把 `sockfd` 从 active socket 转换成 listening socket，用来接收客户端的请求
        + `backlog` 的数值表示 kernel 在接收多少个请求之后（队列缓存起来）开始拒绝请求
    + [*]`accept`: 调用 `accept` 函数，开始等待客户端请求
        + `int accept(int listenfd, SA *addr, int *addrlen);`
        + 等待绑定到 `listenfd` 的连接接收到请求，然后把客户端的 socket address 写入到 `addr`，大小写入到 `addrlen`
        + 返回一个 connected descriptor 用来进行信息传输（类似 Unix I/O）
        + 具体的过程可以参考 图3
2. 开启客户端（`open_clientfd` 函数，设定访问地址，尝试连接）
    + `getaddrinfo`: 设置客户端的相关信息，具体可以参见 图1&2
    + `socket`: 创建 socket descriptor，也就是之后用来读写的 file descriptor
    + `connect`: 客户端调用 `connect` 来建立和服务器的连接
        + `int connect(int clientfd, SA *addr, socklen_t addrlen);`
        + 尝试与在 socker address `addr` 的服务器建立连接
        + 如果成功 `clientfd` 可以进行读写
        + connection 由 socket 对描述 `(x:y, addr.sin_addr:addr.sin_port)`
        + `x` 是客户端地址，`y` 是客户端临时端口，后面的两个是服务器的地址和端口
        + 最好是用 `getaddrinfo` 生成的参数作为 `addr` 和 `addrlen` 
3. 交换数据（主要是一个流程循环，客户端向服务器写入，就是发送请求；服务器向客户端写入，就是发送响应）
    + [Client]`rio_writen`: 
    + [Client]`rio_readlineb`: 
    + [Server]`rio_readlineb`:
    + [Server]`rio_writen`: 
4. 关闭客户端（主要是 `close`）
    + [Client]`close`:
5. 断开客户端（服务接收到客户端发来的 EOF 消息之后，断开已有的和客户端的连接）
    + [Server]`rio_readlineb`:
    + [Server]`close`: 

![图 1 Generic socket address](/images/14599747281009.jpg)

![图 2 Socket Address Structures](/images/14599747501489.jpg)

![图3 `accept` Illustrated](/images/14599774536738.jpg)

> [Client]Connected Descriptor vs. [Server]Listening Descriptors

 这两个的差别还是需要注意一下：
 
 ![](/images/14599776259093.jpg)

之所以要有这样的差别，是因为这样服务器可以同时处理多个请求（只要 fork 即可）

## 代码讲解

> [Client] `open_clientfd`

用来建立和服务器的连接，协议无关

```c
int open_clientfd(char *hostname, char *port) {
    int clientfd;
    struct addrinfo hints, *listp, *p;
    
    // Get a list of potential server address
    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_socktype = SOCK_STREAM; // Open a connection
    hints.ai_flags = AI_NUMERICSERV; // using numeric port arguments
    hints.ai_flags |= AI_ADDRCONFIG; // Recommended for connections
    getaddrinfo(hostname, port, &hints, &listp);
    
    // Walk the list for one that we can successfully connect to
    // 如果全部都失败，才最终返回失败（可能有多个地址）
    for (p = listp; p; p = p->ai_next) {
        // Create a socket descriptor
        // 这里使用从 getaddrinfo 中得到的参数，实现协议无关
        if ((clientfd = socket(p->ai_family, p->ai_socktype,
                               p->ai_protocol)) < 0)
            continue; // Socket failed, try the next
        
        // Connect to the server
        // 这里使用从 getaddrinfo 中得到的参数，实现协议无关
        if (connect(clientfd, p->ai_addr, p->ai_addrlen) != -1)
            break; // Success
        
        close(clientfd); // Connect failed, try another
    }
    
    // Clean up
    freeaddrinfo(listp);
    if (!p) // All connections failed
        return -1;
    else // The last connect succeeded
        return clientfd;
}
```

> [Server] `open_listenfd`

创建 listening descriptor，用来接收来自客户端的请求，协议无关

```c
int open_listenfd(char *port){
    struct addrinfo hints, *listp, *p;
    int listenfd, optval=1;
    
    // Get a list of potential server addresses
    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_socktype = SOCK_STREAM; // Accept connection
    hints.ai_flags = AI_PASSIVE | AI_ADDRCONFIG; // on any IP address
    hints.ai_flags |= AI_NUMERICSERV; // using port number
    // 因为服务器不需要连接，所以原来填写地址的地方直接是 NULL
    getaddrinfo(NULL, port, &hints, &listp); 
    
    // Walk the list for one that we can successfully connect to
    // 如果全部都失败，才最终返回失败（可能有多个地址）
    for (p = listp; p; p = p->ai_next) {
        // Create a socket descriptor
        // 这里使用从 getaddrinfo 中得到的参数，实现协议无关
        if ((listenfd = socket(p->ai_family, p->ai_socktype,
                               p->ai_protocol)) < 0)
            continue; // Socket failed, try the next
        
        // Eliminates "Address already in use" error from bind
        setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR), 
                    (const void *)&optval, sizeof(int));
        
        // Bind the descriptor to the address
        if (bind(listenfd, p->ai_addr, p->ai_addrlen) == 0)
            break; // Success
        
        close(listenfd); // Bind failed, try another
    }
    
    // Clean up
    freeaddrinfo(listp);
    if (!p) // No address worked
        return -1;
    
    // Make it a listening socket ready to accept connection requests
    if (listen(listenfd, LISTENQ) < 0) {
        close(listenfd);
        return -1;
    }
    return listenfd;
}
```

接下来看一个简单的 socket 服务器实例

> Echo Client: Main Routine

这个客户端做的事情很简单，就是把一段用户输入的文字发送到服务器，然后再把从服务器接收到的内容显示到输出中，具体可以参见注释

```c
// echoclient.c
#include "csapp.h"

int main (int argc, char **argv) {
    int clientfd;
    char *host, *port, buf[MAXLINE];
    rio_t rio;
    
    host = argv[1];
    port = argv[2];
    
    // 建立连接（前面已经详细介绍）
    clientfd = Open_clientfd(host, port);
    Rio_readinitb(&rio, clientfd);
    
    while (Fgets(buf, MAXLINE, stdin) != NULL) {
        // 写入，也就是向服务器发送信息
        Rio_writen(clientfd, buf, strlen(buf));
        // 读取，也就是从服务器接收信息
        Rio_readlineb(&rio, buf, MAXLINE);
        // 把从服务器接收的信息显示在输出中
        Fputs(buf, stdout);
    }
    Close(clientfd);
    exit(0);
}
```

> Iterative Echo Server: Main Rountine

服务器做的工作也很简单，接收到从客户端发送的信息，然后返回一个一模一样的。具体参加注释。

```c
// echoserveri.c
#include "csapp.h"
void echo(int connfd);

int main(int argc, char **argv){
    int listenfd, connfd;
    socklen_t clientlen;
    struct sockaddr_storage clientaddr; // Enough room for any addr
    char client_hostname[MAXLINE], client_port[MAXLINE];
    
    // 开启监听端口，注意只开这么一次
    listenfd = Open_listenfd(argv[1]);
    while (1) {
        // 需要具体的大小
        clientlen = sizeof(struct sockaddr_storage); // Important!
        // 等待连接
        connfd = Accept(listenfd, (SA *)&clientaddr, &clientlen);
        // 获取客户端相关信息
        Getnameinfo((SA *) &clientaddr, clientlen, client_hostname,
                     MAXLINE, client_port, MAXLINE, 0);
        printf("Connected to (%s, %s)\n", client_hostname, client_port);
        // 服务器具体完成的工作
        echo(coonfd);
        Close(connfd);
    }
    exit(0);
}

void echo(int connfd) {
    size_t n;
    char buf[MAXLINE];
    rio_t rio;
    
    // 读取从客户端传输过来的数据
    Rio_readinitb(&rio, connfd);
    while((n = Rio_readlineb(&rio, buf, MAXLINE)) != 0) {
        printf("server received %d bytes\n", (int)n);
        // 把从 client 接收到的信息再写回去
        Rio_writen(connfd, buf, n);
    }
}
```

## 测试工具

测试的时候，我们可以使用 `telnet` 应用来测试服务器（只传输 ASCII 字符串的话，命令行工具无法显示图片），例如：

使用方法 `$ telnet <host> <portnumber>`，例如

![Testing Echo Server with `telnet`](/images/14599877200828.jpg)

## Web 服务器

客户端和服务器通过 HyperText Transfer Protocol(HTTP) 协议进行传输，具体的步骤是

+ 客户端和服务器建立 TCP 连接
+ 客户端请求内容
+ 服务器响应所请求的内容
    + content: a sequence of bytes with an associated MIME (Multipurporse Internet Mail Extensions) types
+ （最终）客户端和服务器关闭连接

![Web Server Basics](/images/14599884494825.jpg)

目前的版本是 [HTTP/1.1 RFC 2616, June, 1999](http://www.w3.org/Protocols/rfc2616/rfc2616.html)

一些 MIME 类型，更详细可见[这里](http://www.iana.org/assignments/media-types/media-types.xhtml)
 
 + `text/html` HTML document
 + `text/plain` Unformatted text
 + `image/gif` Binary image encoded in GIF format
 + `image/png` Binary image encoded in PNG format
 + `imgae/jpeg` Binary image encoded in JPEG format

> 静态内容与动态内容

这部分比较简单，直接上图

![](/images/14599890887501.jpg)

> URL 相关

这部分比较简单，直接上图

![](/images/14599891181741.jpg)

> HTTP Request

这部分比较简单，直接上图

![](/images/14599892040565.jpg)

> HTTP Responses

这部分比较简单，直接上图

![](/images/14599892360582.jpg)

> Example HTTP Transaction

![](/images/14599895910357.jpg)

剩下的概念主要是 CGI 应用，地址中 GET 表达形式，具体会在之后的习题课结合例子进行讲解，这里就不赘述了。不过还是要提一个重要概念：代理

> Proxies

![](/images/14599898580384.jpg)

![](/images/14599898773522.jpg)

