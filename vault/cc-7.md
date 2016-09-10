title: 云计算 第 7 课 AWS 动手玩
categories:
- Technique
date: 2016-01-16 14:41:04
toc: true
tags:
- CMU
- 云计算
- AWS
- 实例
---

了解了基本的原理和工具后，这一讲我们用一个实际的例子来上手 AWS。

<!-- more -->

---

通常来说在云平台上用来计算的资源叫做虚拟机，也叫实例 instance。不是所有的实例都是一样的，在 AWS 中有一些可以定制的选择，比如说操作系统和硬件配置等等。

这一讲的主要任务是了解虚拟机并在上面跑一些应用程序，借此来理解不同配置的不同表现。我们都知道更好的配置就性能更强，但是：

+ 更贵的是不是真的更好？
+ 有没有什么办法来测量其性能表现？
+ 性能的差别有多少？

这些问题都不简单，但是这一讲会给你一点点启发。

我们首先会启动三种不同的 EC2 实例，利用内置的性能测评来比较它们的相对性能表现，最后用公共访问的 web 服务器来测试性能。

## 启动实例与性能测评

虽然我们知道有几种不同的配置，但是除了纸面上的差别，真正的性能差别有多少呢？现在我们就来分别测试一下，按照以下步骤进行

1. 启动三个分别为 `t2.small`, `t2.medium`, `t2.large` 的实例，硬盘选择 30GB 的 General SSD，镜像选择 Ubuntu Server 14.04 LTS (HVM), General SSD Volume Type (ami-d05e75b8)。这里最好选择 **spot instances**，可以省点钱，更多请查看[这里](https://aws.amazon.com/ec2/spot/)。如果 spot instances 不可用的话，那么选择 on-demand instances
2. 在启动这些实例的时候，需要指定 **Security Group**，创建允许 HTTP(TCP 端口 80) 和 SSH(TCP 端口 22) 的 security group
3. 同样也需要指定一个已经存在的密钥对文件（或者生成一个）以连接这些实例
4. 启动实例之后，可能需要等待几分钟，在 web 控制台可以查看目前的状况
5. 用 SSH 及 PEM 文件来连接到这些实例。

启动完成后大概是这样：

![cc20](/images/cc20.jpg)

## 系统性能测评

我们需要在已经启动的实例中安装性能测评工具。通过这个来评估完成一个任务所需的最佳实例组合以做到多快好省。

虽然云服务提供商是利用虚拟化技术来提供资源，但是需要知道实例的性能一来不是稳定的，二来也不保证达到某个水平。这里我们使用 `sysbench` 来评估性能，它是一个轻量级跨平台的性能测试软件，能够快速评估 CPU，内存和文件读写性能。

用 `ssh -i keyfile.pem ubuntu@dnsaddress` 来进行连接

然后执行下面的命令安装：

```bash
sudo apt-get update
sudo apt-get install sysbench
```

安装完成之后就可以按照给出的三个配置来进行测试。

### CPU 测试

分别在三个不同的机器运行下列命令，统计结果：

```bash
sysbench --num-threads=8 --test=cpu --cpu-max-prime=20000 run
sysbench --num-threads=1 --test=cpu --cpu-max-prime=40000 run
sysbench --num-threads=4 --test=cpu --cpu-max-prime=50000 run
```

表后面的三列分别是 `t2.small`, `t2.medium`, `t2.large` 的总时间

Max Prime | Thread | Time(s) | Time(m) | Time(l)
:--: | :--: | :--: | :--: | :--:
20000 | 8 | 29.4015s | 14.8129s | 14.8693s 
40000 | 1 | 77.7609s | 76.0034s | 77.1984s
50000 | 4 | 104.8054s | 51.5801 | 52.4278s

可以看到其实在当前任务下，`t2.medium` 与 `t2.large` 的性能差别并不大，就可以根据价格来进行取舍了。

### 文件 IO 性能测试

AWS 通常会保证 Input/Output Operations per Second (IOPS)。不同的硬盘访问逻辑对性能影响很大（缓存导致），传统的机械硬盘受寻址时间和旋转速度的影响，固态硬盘受驱动和设备控制器的影响。

同样用下面的命令来进行测试：

```bash
sysbench --test=fileio --file-total-size=20G prepare 
# 可能需要等待30分钟，使用 byobu / screen / tmux 来保持连接
# 或者可以在 iterm 设置中 profiles -> sessions -> When idel, send ASCII code
# 又或者 在客户端的 ~/.ssh/ 文件夹中添加 config 文件，并添加配置： ServerAliveInterval 60 

sysbench --test=fileio --file-total-size=20G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run
```

\ | t2.small | t2.medium | t2.large
:--: | :--: | :--: | :--:
Read(Gb) | 3.8986 | 3.9448 | 5.6735
Written(Gb) | 2.599 | 2.6299 | 3.7823
Total trans(Gb) | 6.4976 | 6.5747 | 9.4559
Speed(Mb/sec) | 22.178 | 22.442 | 32.276

比较一下表中的数据，是不是随着价格的提高，性能也有对应比例的提高？

可以看到从 small 到 medium 几乎没有提升，但是到 large 就有比较大的提升

## Web 服务器性能测试

这门课上我们会学会如何配置，搭建和部署 web 服务。所以需要熟悉如何折腾服务器。这里我们会在每个实例中安装和部署一个简单的 web 服务器，修改一下主页以便测试和评分。

[Apache 简介视频（墙外）](https://youtu.be/6_tPobCyF9o)

LAMP = Linux + Apache + MySQL + PHP

用以下命令安装 apache：`sudo apt-get install apache2`

安装完成之后，用自己的浏览器访问这个实例的地址，应该就可以看到 Apache 的欢迎页面（这里需要在 Security Group 中允许 HTTP 访问）

![](/images/14529800450459.jpg)

如果访问 `cd /var/www/html` 就可以看到有一个 `index.html` 页面。

我们把这个页面的内容替换成：

`15619 is awesome!` （`sudo vim index.html` 然后命令模式下 `dG`）

再次访问的时候就可以看到变化了

然后我们来安装 apachebench，也就是服务器管理器，命令如下：

`sudo apt-get install apache2-utils`

安装完成后，输入 `ab` 应该能看到命令界面，一个简单的测试命令为：

`ab -n 1000 -c 100 http://localhost/` (一次 1000 个 request，一共 100 次，最后是想要测试的页面)，然后就可以看到各种数据统计。

在之后项目中，可能会用一组 load generators 来访问 web service 以检验你的 web 服务的性能和正确性。通常来说用平均每秒可以处理的请求数来衡量性能。正确率会基于字符串或正则表达式来拼配。类似于下图：

![](/images/14529809204231.jpg)

地址为 `http://p0.loadgen.theproject.zone`

VM 的地址就是 EC2 的 public dns 地址；然后输入 andrew id 和提交密码(在页面上方)

测试成功的话会显示 `Success!! Check TPZ for your score`，然后就可以课程网站上查看自己的成绩。

## Vertical Scaling

根据需要来改变所需的系统资源称为 Vertical Scaling。

在这个场景中，我要启动一个 load generator，它会在一个数据中心实例上创造一些负载。通过一个恒定的负载，来比较不同类型实例的吞吐量（用 rps 来衡量，每秒处理的访问量）。

在这里我们不能 SSH 连接到这些实例，但是可以通过访问特定的 endpoint 来查看当前的情况，具体步骤如下：

1. 先创建两个实例，一个是 `m3.medium` 类型为 `ami-ba2c77d0` 的 load generator 实例，另一个是 `m3.medium` 类型为 `ami-ec144c86` 的数据中心实例（这里我用的是竞价型，比较便宜）
2. 如果是用竞价型，那么需要在实例创建时再添加一次标签，因为之前添加的是请求的标签
3. 访问 load generator，如下图所示

![c](/images/cc21.jpg)

输入提交密码和 andrew id 之后，进入下一步并输入数据中心的访问地址。第一次会出错，只要刷新一下就好，然后就可以等待测试完成，记录下日志数据。

然后使用 `m3.large` 再来重复一次，比较一下得到的数据

RPS | Minute 1 | Minute 2 | Minute 3 | Minute 4 | Minute 5
:--: | :--: | :--: | :--: | :--: | :--:
m3.medium | 215.12 | 467.34 | 460.19 | 466.87 | 466.79
m3.large | 1133.86 | 1911.31 | 1870.61 | 1866.46 | 1849.03

可以看到从 medium 到 large 的提升还是非常大的。

完成之后记得删除对应的实例

