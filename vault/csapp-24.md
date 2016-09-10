title: 深入理解计算机系统 第 24 课 Synchronization - Advanced
categories:
- Technique
toc: true
date: 2016-04-07 11:30:39
tags:
- CMU
- 组成原理
- 计算机
- 同步
---

这节课我们通过『生产者-消费者问题』以及『读者-写者问题』来更深入理解同步机制。

<!-- more -->

---

Semaphores 实际上可以认为是线程之间最简单的通知机制，用来控制共享资源的访问。这之中有两个非常经典的问题：

+ The Producer-Consumer Problem
+ The Readers-Writers Problem

## 生产者-消费者问题

![Producer-Consumer Problem](/images/14600704484721.jpg)

具体的同步模型为：

+ 生产者等待空的 slot，把 item 存储到 buffer，并通知消费者
+ 消费整等待 item，从 buffer 中移除 item，并通知生产者

主要用于

+ 多媒体处理
    + 生产者生成 MPEG 视频帧，消费者进行渲染
+ 事件驱动的图形用户界面
    + 生产者检测到鼠标点击、移动和键盘输入，并把对应的事件插入到 buffer 中
    + 消费者从 buffer 中获取事件，并绘制到到屏幕上

接下来我们实现一个有 n 个元素 buffer，为此，我们需要一个 mutex 和两个用来计数的 semaphore：

+ `mutex`: 用来保证对 buffer 的互斥访问
+ `slots`: 统计 buffer 中可用的 slot 数目
+ `items`: 统计 buffer 中可用的 item 数目

我们直接来看代码，就比较清晰了


```c
// 头文件 sbuf.h
// 包括几个基本操作
#include "csapp.h"

typedef struct {
    int *buf;    // Buffer array
    int n;       // Maximum number of slots
    int front;   // buf[(front+1)%n] is first item
    int rear;    // buf[rear%n] is the last item
    sem_t mutex; // Protects accesses to buf
    sem_t slots; // Counts available slots
    sem_t items; // Counts available items
} sbuf_t;

void sbuf_init(sbuf_t *sp, int n);
void sbuf_deinit(sbuf_t *sp);
void sbuf_insert(sbuf_t *sp, int item);
int sbuf_remove(sbuf_t *sp);
```

然后是具体的实现

```c
// sbuf.c

// Create an empty, bounded, shared FIFO buffer with n slots
void sbuf_init(sbuf_t *sp, int n) {
    sp->buf = Calloc(n, sizeof(int));
    sp->n = n;                  // Buffer holds max of n items
    sp->front = sp->rear = 0;   // Empty buffer iff front == rear
    Sem_init(&sp->mutex, 0, 1); // Binary semaphore for locking
    Sem_init(&sp->slots, 0, n); // Initially, buf has n empty slots
    Sem_init(&sp->items, 0, 0); // Initially, buf has 0 items
}

// Clean up buffer sp
void sbuf_deinit(sbuf_t *sp){
    Free(sp->buf);
}

// Insert item onto the rear of shared buffer sp
void sbuf_insert(sbuf_t *sp, int item) {
    P(&sp->slots);                        // Wait for available slot
    P(&sp->mutext);                       // Lock the buffer
    sp->buf[(++sp->rear)%(sp->n)] = item; // Insert the item
    V(&sp->mutex);                        // Unlock the buffer
    V(&sp->items);                        // Announce available item
}

// Remove and return the first tiem from the buffer sp
int sbuf_remove(sbuf_f *sp) {
    int item;
    P(&sp->items);                         // Wait for available item
    P(&sp->mutex);                         // Lock the buffer
    item = sp->buf[(++sp->front)%(sp->n)]; // Remove the item
    V(&sp->mutex);                         // Unlock the buffer
    V(&sp->slots);                         // Announce available slot
    return item;
}
```

## 读者-写者问题

是互斥问题的通用描述，具体为：

+ 读者线程只读取对象
+ 写者线程修改对象
+ 写者对于对象的访问是互斥的
+ 多个读者可以同时读取对象

常见的应用场景是：

+ 在线订票系统
+ 多线程缓存 web 代理

根据不同的读写策略，又两类读者写者问题，需要注意的是，这两种情况都可能出现 starvation。

> 第一类读者写者问题（读者优先）

+ 如果写者没有获取到使用对象的权限，不应该让读者等待
+ 在等待的写者之后到来的读者应该在写者之前处理
+ 也就是说，只有没有读者的情况下，写者才能工作

> 第二类读者写者问题（写者优先）

+ 一旦写者可以处理的时候，就不应该进行等待
+ 在等待的写者之后到来的读者应该在写者之后处理


把前面这些拼到一起，就有了完整的 Server 

![Prethreaded Concurrent Server](/images/14600747953883.jpg)

具体的代码为

```c
sbuf_t sbuf; // Shared buffer of connected descriptors


static int byte_cnt;  // Byte counter
static sem_t mutex;   // and the mutex that protects it


void echo_cnt(int connfd){
    int n;
    char buf[MAXLINE];
    rio_t rio;
    static pthread_once_t once = PTHREAD_ONCE_INIT;
    
    Pthread_once(&once, init_echo_cnt);
    Rio_readinitb(&rio, connfd);
    while ((n = Rio_readlineb(&rio, buf, MAXLINE)) != 0) {
        P(&mutex);
        byte_cnt += n;
        printf("thread %d received %d (%d total) bytes on fd %d\n",
                    (int) pthread_self(), n, byte_cnt, connfd);
        V(&mutex);
        Rio_writen(connfd, buf, n);
    }
}

static void init_echo_cnt(void){
    Sem_init(&mutex, 0, 1);
    byte_cnt = 0;
}

void *thread(void *vargp){
    Pthread_detach(pthread_self());
    while (1) {
        int connfd = sbuf_remove(&sbuf); // Remove connfd from buf
        echo_cnt(connfd);                // Service client
        Close(connfd);
    }
}

int main(int argc, char **argv) {
    int i, listenfd, connfd;
    socklen_t clientlen;
    struct sockaddr_storage clientaddr;
    pthread_t tid;
    
    listenfd = Open_listenfd(argv[1]);
    sbuf_init(&sbuf, SBUFSIZE);
    for (i = 0; i < NTHREADS; i++) // Create worker threads
        Pthread_create(&tid, NULL, thread, NULL);
    while (1) {
        clientlen = sizeof(struct sockaddr_storage);
        connfd = Accept(listenfd, (SA *)&clientaddr, &clientlen);
        sbuf_insert(&sbuf, connfd); // Insert connfd in buffer
    }
}
```

## 线程安全 

在线程中调用的函数必须是线程安全的，定义为：

> A function is thread-safe iff it will always produce correct results when called repeatedly from multiple concurrent threads

主要有 4 类线程不安全的函数

1. Functions that do not protect shared variables
    + 解决办法：使用 P 和 V semaphore 操作
    + 问题：同步操作会影响性能
2. Functions that keep state across multiple invocations
    + Fix: Pass state as part of argument 
3. Functions that return a pointer to a static variable
    + Fix 1: Rewrite function so caller passes address of variable to store result
    + Fix 2: Lock-and-copy
4. Functions that call thread-unsage functions
    + 解决办法：只调用线程安全的函数

另一个重要的概念是 Reentrant Function，定义为：

> A function is **reentrant** iff it accesses no shared variables when called by multiple threads

![](/images/14600771838118.jpg)

Reentrant Functions 是线程安全函数非常重要的子集，不需要同步操作，对于第二类的函数来说（上面提到的），唯一的办法就是把他们修改成 reentrant 的。

标准 C 库中的函数都是线程安全的（如 `malloc`, `free`, `printf`, `scanf`），大多数 Unix 的系统调用也都是线程安全的，除了下面这些例外：

![](/images/14600772808229.jpg)

最后要提的两点是『竞争条件』和『死锁』，对于前者来说，要避免状态的共享，对于后者来说，要保证共享资源按照顺序进行请求。举个例子：

![Deadlocking With Semaphores](/images/14600774132333.jpg)

 留意两个函数中 P 操作的顺序，用 Progress Graph 来表示就是：
 
![Deadlock Visualized in Progress Graph](/images/14600774554184.jpg)

但是如果我们调整顺序：

![Acquire shared resources in same order](/images/14600774851875.jpg)

情况就会变成这样

![Avoided Deadlock in Progress Graph](/images/14600775118044.jpg)

也就解决了死锁的问题。


