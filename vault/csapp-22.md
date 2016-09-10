title: 深入理解计算机系统 第 22 课 Concurrent Programming
categories:
- Technique
toc: true
date: 2016-04-06 22:16:36
tags:
- CMU
- 组成原理
- 计算机
- 并行
---

如果想要充分利用硬件资源，就要尽可能让计算机同时处理多项事务，具体是怎么实现的？又哪些基本的策略？这节课会一一解答。

<!-- more -->

---

首先一定要清楚地意识到：『并行编程不简单』！最主要的原因恐怕就是我们自己的大脑，人脑实际上是一个非常精妙的系统，所采取的并行策略是一明一暗两条线，但是对于明线来说，是线性的，于是就和计算机中并行的概念冲突了。另外时间这个概念也是线性的，这就导致了想要处理好并行程序可能出现的各种问题几乎是不可能的（或者常常要出错）。

常见的错误有仨：竞争条件、死锁和活锁，具体如下：

![Classical problem classes](/images/14599962597558.jpg)

## 服务器的例子

我们前面实现的服务器，一次只能处理一个请求，只有当前的请求处理完了，才能继续处理下一个。

![](/images/14599976220466.jpg)

这里具体讲解一下：Client 1 向 Server 发送连接请求(connect)，Server 接受(accept)之后开始等待 Client 1 发送请求（也就是开始 read），这之后 Client 1 发送具体的内容(write)后转为等待响应(call read)，Server 的 read 接收到了内容之后，发送响应(write) 后仅需进入等待(read)，而 Client 1 接收到了响应(ret read)，最后根据用户指令退出(close)。

而只有当 Client 1 断开之后，Server 才会处理 Client 2 的请求，从图中也可以看到这一点。具体是在哪里等待呢？因为 TCP 会缓存，所以实际上 Client 2 在 `ret read` 之前进行等待，如：

![Where Does Second Client Block](/images/14599992829071.jpg)

为了解决这个问题，我们可以使用并行的策略，同时处理不同客户端发来的请求。

## 并行方法

总体来说，根据系统机制的层级和实现方式，有下面三大类方法：

1. 基于进程
    + 内核自动管理多个逻辑流
    + 每个进程有其私有的地址空间（也就是说进程切换的时候需要保存和载入数据）
2. 基于事件
    + 由程序员手动控制多个逻辑流
    + 所有的逻辑流共享同一个地址空间
    + 这个技术称为 I/O multiplexing
3. 基于线程
    + 内核自动管理多个逻辑流
    + 每个线程共享地址空间
    + 属于基于进程和基于事件的混合体

### 基于进程

为每个客户端分离出一个单独的进程，是建立了连接之后才开始并行，连接的建立还是串行的。

![Spawn separate process for each client](/images/14600273587521.jpg)

具体的代码为

```c

void sigchld_handler(int sig){
    while (waitpid(-1, 0, WNOHANG) > 0)
        ;
    return;
    // Reap all zombie children
}

int main(int argc, char **argv) {
    int listenfd, connfd;
    socklen_t clientlen;
    struct sockaddr_storage clientaddr;
    
    Signal(SIGCHLD, sigchld_handler);
    listenfd = Open_listenfd(argv[1]);
    while (1) {
        clientlen = sizeof(struct sockaddr_storage);
        connfd = Accept(listenfd, (SA *) &clientaddr, &clientlen);
        if (Fork() == 0) {
            Close(listenfd); // Child closes its listening socket
            echo(connfd); // Child services client
            Close(connfd); // Child closes connection with client
            exit(0); // Child exits
        }
        Close(connfd); // Parent closes connected socket (important!)
    }
}
```

关键步骤(accept)的描述为

![Concurrent Server: `accept` Illustrated](/images/14600278440053.jpg)

执行模型为

![Process-based Server Execution Model](/images/14600278862489.jpg)

+ 每个客户端由独立子进程处理
    + 必须回收僵尸进程，来避免严重的内存泄露
+ 不同进程之间不共享数据
+ 父进程和子进程都有 `listenfd` 和 `connfd`，所以在父进程中需要关闭 `connfd`，在子进程中需要关闭 `listenfd`
    + 内核会保存每个 socket 的引用计数，在 fork 之后 `refcnt(connfd) = 2`，所以在父进程需要关闭 connfd，这样在子进程结束后引用计数才会为零

> 优劣

![Pros and Cons of Process-based Servers](/images/14600282200028.jpg)

### 基于事件

服务器会维护一个 connection 数组，包含若干 `connfd`，具体的过程为：

![](/images/14600283355967.jpg)

这里一个很重要的技术是 I/O Multiplexing，感兴趣的同学可以查阅书中对应章节（具体内容会在习题课中介绍）。

> 优劣

![Pros and Cons of Event-based Servers](/images/14600283984253.jpg)

### 基于线程

和基于进程的方法非常相似，唯一的区别是这里用线程。进程其实是比较『重』的，一个进程包括：

![Traditional View of a Process](/images/14600286544772.jpg)

当然，我们也可以从线程的角度来描述进程：

![Alternate View of a Process](/images/14600286291425.jpg)

这两个角度的区别在于，通过线程视角观察，把单独的可执行部分抽离出来了，于是当一个进程有多个线程的时候，看起来像这样：

![A Process With Multiple Threads](/images/14600287485774.jpg)

每个线程有自己的线程 id，有自己的逻辑控制流，也有自己的用来保存局部变量的栈（其他线程可以修改）但是会共享所有的代码、数据以及内核上下文。

和进程不同的是，线程没有一个明确的树状结构（使用 `fork` 是有明确父进程子进程区分的），看起来就像下面这样：

![Logical View of Threads](/images/14600293169627.jpg)

和进程中『并行』的概念一样，如果两个线程的控制流在时间上有『重叠』（或者说有交叉），那么就是并行的，例如：

![Concurrent Threads](/images/14600295257230.jpg)

单核与多核处理器也有一点点不同：

![Concurrent Thread Execution](/images/14600297006546.jpg)

进程和线程的差别已经被说了太多次，这里简单提一下。相同点在于，它们都有自己的逻辑控制流，可以并行，都需要进行上下文切换。不同点在于，线程共享代码和数据（进程通常不会），线程开销比较小（创建和回收）

## Posix Threads (Pthreads) Interface

Pthreads 是一个线程库，基本上只要是 C 程序能跑的平台，都会支持这个标准，具体如下；

![](/images/14600332547386.jpg)

一个例子

![The Pthreads "hello, world" Program](/images/14600332776267.jpg)

这个程序的流程描述为

![Execution of Thread "hello, world"](/images/14600338560676.jpg)

我们用线程的方式重写一次之前的 Echo Server

```c
// Thread routine
void *thread(void *vargp){
    int connf = *((int *)vargp);
    // detach 之后不用显式 join，会在执行完毕后自动回收
    Pthread_detach(pthread_self());
    Free(vargp);
    echo(connfd);
    // 一定要记得关闭！
    Close(connfd);
    return NULL;
}

int main(int argc, char **argv) {
    int listenfd, *connfdp;
    socklen_t clientlen;
    struct sockaddr_storage clientaddr;
    pthread_t tid;
    
    listenfd = Open_listenfd(argv[1]);
    while (1) {
        clientlen = sizeof(struct sockaddr_storage);
        // 这里使用新分配的 connected descriptor 来避免竞争条件
        connfdp = Malloc(sizeof(int));
        *connfdp = Accept(listenfd, (SA *) & clientaddr, &clientlen);
        Pthread_create(&tid, NULL, thread, connfdp);
    }
}
```

具体的执行模型为：

![Thread-based Server Execution Model](/images/14600343334886.jpg)

在这个模型中，每个客户端由单独的线程进行处理，这些线程除了线程 id 之外，共享所有的进程状态（但是每个线程有自己的局部变量栈）。需要注意的有：

![Issues With Thread-Based Servers](/images/14600346185454.jpg)

说了这么多，其实就是同步问题（后面两节课会专门介绍）

> 优劣

![](/images/14600346688214.jpg)

## 总结

这里我们了解了三大类方法的特点，具体的会在习题课结合例子说明，这里是一个简单的总结

![](/images/14600347193577.jpg)


