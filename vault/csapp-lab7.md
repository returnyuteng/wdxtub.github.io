title: 深入理解计算机系统 习题课 7 Proxylab
categories:
- Technique
toc: true
date: 2016-04-07 21:52:23
tags:
- CMU
- Proxy
- 习题课
- 计算机
---

这次，我们来自己实现一个多线程带缓存的代理服务器！

<!-- more -->

---

## 准备工作

这次的作业主要分三个部分：

1. Sequential Proxy: 接收客户端发送的 HTTP 请求，解析之后向目标服务器转发，获得响应之后再转发回客户端
2. Concurrent Proxy: 在第一步的基础上，支持多线程
3. Cache Web Objects: 使用 LRU 缓存单独的对象，而不是整个页面

老套路

+ 上传文件 `scp proxylab-handout.tar dawang@shark.ics.cs.cmu.edu:~/513`
+ 登录 `ssh -X dawang@shark.ics.cs.cmu.edu`
+ 解压 `tar xvf proxylab-handhout.tar`

因为我比较习惯在本地写代码，所以把文件复制回来：

+ 服务器至本地 `scp dawang@shark.ics.cs.cmu.edu:~/513/proxylab-handout/proxy.c ./`
+ 本地至服务器 `scp ./proxy.c dawang@shark.ics.cs.cmu.edu:~/513/proxylab-handout/`

测试方法：

+ 查看本机地址 ``

> 评分标准

使用 `./driver.sh` 来进行测试

+ [40] 基本正确性（自动评分）
+ [15] 并行（自动评分）
+ [15] 缓存（自动评分）
+ [20] 实际页面测试
    + [http://www.cs.cmu.edu/~213](http://www.cs.cmu.edu/~213)
    + [http://csapp.cs.cmu.edu](http://csapp.cs.cmu.edu)
    + [http://www.cs.cmu.edu](http://www.cs.cmu.edu)
    + [http://chalkdinosaur.bandcamp.com](http://chalkdinosaur.bandcamp.com)
+ [10] 代码风格

在工作文件夹中使用 `make handin` 来生成提交文件，下载到本地 `scp dawang@shark.ics.cs.cmu.edu:~/513/proxylab-handin.tar ./`
，然后交到 autolab 即可。

样例测试命令：`curl --proxy 128.2.220.15:45962 http://www.cs.cmu.edu`


## Sequential Web Proxy

第一步是实现一个简单的代理服务器，只处理 `HTTP/1.0 GET` 请求。具体步骤为

+ 端口号在命令行指令中指定
    + 申请自己的端口 `$ ./port-for-user.pl dawang`，这里申请的总是偶数，所以如果需要一个额外的端口，直接在端口号 +1 即可
    + 不要随便指定端口，不然很可能干扰到别人
    + `$ ./proxy 12345`
+ 监听从该端口进入的所有请求
+ 解析请求，并转发合法的 HTTP 请求
    + 假设请求为 `GET http://www.cmu.edu/hub/index.html HTTP/1.1`
    + 则主机名 `www.cmu.edu`
    + 请求的页面 `/hub/index.html`
    + HTTP 请求每行以 `\r\n` 结束，以一个空行 `\r\n` 结尾
    + 需要判断地址里有没有带端口
+ 把从服务器获取到的响应返回给客户端

请求的 header 也很重要，一定要有的内容是：

+ `Host`: 如 `Host: www.cmu.edu`
+ `User-Agent`: 如 `User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:10.0.3) Gecko/20120305 Firefox/10.0.3`
+ `Connection`: 必须发送 `Connection: close`
+ `Proxy-Connection`: 必须发送 `Proxy-Connection: close`

## Multiple Concurrent Requests

使用 POSIX 线程，最好在线程一开始执行 `pthread_detach(pthread_self());` 这样就不用自己负责清理线程了。

注意竞争条件，尽量减少共享资源，访问共享资源的时候需要同步。

`open_clientfd` 和 `open_listenfd` 函数是线程安全的

## Caching Web Objects

具体缓存的机制是 LRU，一些具体的参数是：

+ 缓存大小限制 `MAX_CACHE_SIZE = 1 MiB`，注意只缓存 web 对象，其他诸如 metadata 应该忽略
+ 单个文件大小限制 `MAX_OBJECT_SIZE = 100 KiB`
+ 如果有 T 个连接，那么最大的空间为 `MAX_CACHE_SIZE + T * MAX_OBJECT_SIZE`

同步问题可以参考『读者-写者问题』

## 调试工具

+ Telnet: 不安全的 ssh，需要手动构造 HTTP 请求，如果想要测试非法的 header，这个功能就很有用
    + `man telnet`
    + `telnet www.wdxtub.com`
    + `GET http://www.wdxtub.com HTTP/1.0` 
+ cURL: 会自动构建 HTTP 请求
    + `curl http://www.wdxtub.com`
    + 代理模式 `curl --proxy lemonshark.ics.cs.cmu.edu:3092 http://www.wdxtub.com` 
+ `netcat`: 多用途网络工具，用法与 `telnet` 类似
    + `nc catshark.ics.cs.cmu.edu 12345`
    + `GET http://www.cmu.edu/hub/index.html HTTP/1.0`

## 注意事项

+ 大端小端
+ 能够处理各种 URL（合法或非法）
+ 不是所有的内容都是 ASCII 码，注意选择对应的函数来处理二进制文件（图像和视频）
+ 所有的请求都用 `HTTP/1.0` 来转发
+ 需要处理 `SIGPIPE` 信号，默认的操作是关闭进程，这里应该屏蔽这个信号
+ 使用 Robust I/O package 的 `read`, `write`, `fread`, `fwrite` 来增加健壮性
+ 如果调用 `read` 来获取已经被关闭的 socket，会返回 -1，并给出 `ECONNRESET` 错误，不应该因为这个错误而导致进程终结
+ 如果调用 `write` 来获取已经被关闭的 socket，会返回 -1，并给出 `EPIPE` 错误，不应该因为这个错误而导致进程终结
+ 代码注意模块化
+ 因为可以写单独的文件，需要对应更新 Makefile
+ Header 注意格式规范
+ 做 Cache 的时候注意指针


![最终效果](/images/14600850252282.jpg)


