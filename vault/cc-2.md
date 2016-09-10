title: 云计算 第 2 课 AWS 简介
categories:
- Technique
date: 2016-01-15 10:29:53
toc: true
tags:
- CMU
- 云计算
- AWS
---

开发过程中最麻烦的往往不是开发本身，而是环境搭建，尤其是调通各种接口和服务。这一讲会介绍 AWS 的基本使用。

<!-- more -->

---

## Amazon Web Services AWS

AWS 是目前最广泛应用的云服务，包含各种不同的服务，比如 Amazon Simple Storage Service (S3) 和 Amazon Elastic Compute Cloud (EC2)。在本课程中会使用 Amazon EC2 并且利用 S3 作为永久存储。与此同时也会使用其他的一些技术比如：Elastic Block Store(EBS), CloudWatch 和 DynamoDB。更多的信息请参考 [AWS 网站](http://aws.amazon.com/)

[视频介绍(墙外)](https://www.youtube.com/watch?v=jOhbTAU4OPI#action=share)

### AWS Authentication

除了 AWS 本身的帐号外，访问 API 以及各类 AWS 服务需要使用的一对『钥匙』：**AWS Access Key ID** 以及 **AWS Secret Key**。

生成『钥匙』可以按照以下步骤：

1. 登录 AWS 账户
2. 在右上角的下拉菜单中选择 Security Credentials (安全证书)
	+ 如果是第一次访问的话会弹出一个提示是继续还是看一下教程，可以根据自己的需要来选择
3. 在 Access Key (访问密钥)那一栏创建一个新的密钥，就会包含 **AWS Access Key ID** 以及 **AWS Secret Key**。

成功的话如下图所示，请注意：您一次最多可拥有两个访问密钥（活跃或非活跃）。

![2.pic_hd](/images/2.pic_hd.jpg)

注意！AWS 密钥拥有控制 AWS 帐号的完全权限，所以务必要保存在安全的地方，做到以下几点：

1. 不要与任何人共享 AWS 密钥
2. 不要在代码中用明文保存密钥
3. 不要把包含 AWS 密钥的代码提交到任何库中

比较安全的做法是利用环境变量来保存密钥：

```bash
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_HERE"
export AWS_SECRET_KEY="YOUR_SECRET_KEY_HERE"
```

在代码中就可以使用 `System.getenv("AWS_ACCESS_KEY_ID")` 或者 `os.environ['AWS_ACCESS_KEY_ID']` 来访问，不会出现明文，我在自己电脑里用得名字是：`AWSAccessKeyId` 和 `AWSSecretKey`

### AWS Managment Console

[AWS 概览和账单查看视频教程（墙外）](https://youtu.be/EfnfcIqq_IU)

在右上角的下拉菜单中可以查看花费，如下图所示：

![c](/images/cc1.jpg)

左边的不同选项可以查看不同的信息，这里提示一下，因为之前已经转换为学校付款，所以建议把之前绑定的个人信用卡删除，避免因为操作问题被额外扣钱。

## Elastic Compute Cloud (EC2)

简单来说，EC2 提供计算能力。会为你的服务创建虚拟服务器，因此省去了搭建硬件的过程。可以通过界面或者 API 来启动所需的 instance，配置安全和网络以及管理存储。

开始使用 Amazon EC2 的时候，需要了解一些和本地机器不大一样的关键概念，可以从下面的是品种了解到

[EC2 基础（墙外）](https://youtu.be/t0Gg0Y1Gz6Q)

Instance 可以被看做是虚拟机或虚拟服务器，有虚拟 CPU，内存，本地存储和网络带宽，当然运行的时候是要花钱的。具体的配置不能够自定义，只能从几个类型中选择：

+ `t1.micro`：1 个虚拟 CPU，1 G 内存，无本地存储，低带宽。这种会提供免费的使用时间，基本也就够用了。
+ `m1.small`
+ `m1.large`
+ `c1.xlarge`

具体的类型可以参考亚马逊的[介绍](http://aws.amazon.com/ec2/instance-types)，这里不一一列举。

花钱也有不同的方式：

+ 按需：每个 instance 每小时有固定的价钱
+ Reserved：预付款模式
+ Spot Pricing：Bid for unused capacity，给定一个价格，如果浮动到价格以下，那么就帮你开启，并以当时的价格计价

instance 的存储会在 Elastic Block Storage(EBS)。

Amazon Machine Image (AMI) = 操作系统 + 软件。

一旦一个 instance 启动之后，就有了一个公网的 IP，可以通过 Security Group 来进行访问，比如 22 端口 SSH，80 端口 HTTP，3306 端口 MySQL。

利用密钥对来访问 EC2 instances。

总结一下，启动一个 instance 有以下几个步骤：

1. 选择一个 instance 类型：`t1.micro`
2. 决定一个计价模型：Spot Pricing
3. 选择一个 AMI：Ubuntu AMI
4. 定义一个 Security Group
5. 使用密钥对来连接

### EC2 术语

这部分为了保证准确，使用英文原文：

**EC2 Instance**: EC2 instances are virtual servers that you can configure and launch on Amazon EC2. It can be thought of as a copy of a software image, Amazon Machine Image (AMI), that is actively running on the Amazon EC2 cloud. Instances run on host computers at Amazon’s data center, but this is typically transparent to the user. Instances have many parameters including:

1.**Amazon Machine Image (AMI)**: Is a template that contains a complete software image (operating system, applications, libraries and data). You launch instances using specific AMIs, which are copies of the AMI running as virtual servers in the cloud. You can launch multiple instances of an AMI. 

![](/images/14528880846285.jpg)

2.**Instance Type**: Amazon offers different instance types which have varying amounts of compute and storage available to them. They range from t1.micro to cc2.8xlarge and have different on-demand/reserved and spot prices.

3.**Regions and Availability Zones**: Instances can be launched across various AWS regions, which are distributed geographically (Virginia/Singapore etc.). Each region has a number of availability zones, which are distinct locations within a data center and are engineered to be isolated from other availability zones in the same region. This allows users to spawn instances in the same region but across availability zones and protect applications from the potential failure of a single availability zone.

![](/images/14528881433917.jpg)

4.**Pricing**: Instances can be launched using three different pricing models:

+ The on-demand pricing is a fixed hourly rate that you pay for the instance. It can range from a few cents to a few dollars per hour depending on the instance type.
+ If you want to use an instance for a fixed amount of time, you can purchase reserved instances (which are typically calculated yearly).
+ In addition there are spot instances which are described below.

5.**Spot Instances**: Spot instances are special instances which allow users to bid for unused computing capacity. You can specify an hourly rate that you are willing to pay to use an instance. In addition, for each instance type under each availability zone, Amazon maintains a spot price which reflects the current demand for the instance type. If your bidding price is more than the spot price, an instance will be launched at the current spot price and will continue to run until the spot price exceeds your bid price. Spot instances are volatile but very useful for getting instances for a few hours at prices that are typically much lower than the on-demand prices.

6.**Instance Limits**: AWS has preset limits on the number of simultaneous instances, per instance type, that an account can provision. You can check what the instance limits are for your account by checking the limits under the EC2 dashboard.

### Amazon EC2 Storage

针对于 EC2 instance，Amazon 提供了不同类型的存储：

![](/images/14528883079086.jpg)

**Amazon EC2 Instance Store**: Instance stores are storage volumes that are present on the host computer that the instances are running on. Instance stores are temporary (ephemeral), block level storage. Instance store data is cleared when an instance is stopped or terminated.

**Amazon Elastic Block Store (EBS)**: EBS is a SAN-style storage system that can be used with EC2 instances. EBS presents volumes to the user that can be created independently of an instance and attached to instances as needed. EBS volumes are persistent and flexible. Multiple EBS volumes can be attached to an instance, and an EBS volume can be detached from an instance and attached to another. EBS incurs additional charges (GB/month) over and above EC2 instance charges. EBS volumes can also be backed up by creating a snapshot, which is stored in Amazon S3.

**Amazon Simple Storage Service (S3)**: Amazon S3 is an object storage service which has a web services interface to store and retrieve data. Instances can access data directly on S3 using the web services interface. Amazon S3 is also used to store snapshots of EBS volumes.

### 启动并连接到 instance

[启动 EC2 instance 视频教程（墙外）](https://youtu.be/z-NoOLk2U-g)

具体的步骤如下：

1. 进入控制台，选择 EC2 控制面板（地区注意选择弗吉尼亚北部，右上角那里）
2. 点击实例，现在什么都没有，选择创建一个新实例
3. 之后的 project 中会给出 AMI id，可以在社区 AMI 一栏里搜索选择
4. 这里我们选择 Amazon Linux AMI，选择 `t2.micro`
5. 点击下一步进行具体的配置，这里暂时不需要修改
6. 点击下一步选择存储
7. 点击下一步可以给 instance 一些标签，注意格式应该是 `Project:<Project Number>`，如 `Project:0` 或 `Project:1.1` 这样
8. 点击下一步配置 Security Group，这里可以不用修改，但是如果提高安全性的话可以限制可访问的 ip
9. 然后可以查看各种选项
10. 需要选择一个连接的密钥对（也可以在这里创建一个新的），是一个pem 文件，同样注意保存好
11. 这时候就可以启动 instance

等待一段时间，启动之后就可以连接到这个 instance，具体的步骤如下：

1. 从面板中得到 Public DNS 地址
2. 使用 `ssh -i key_file.pem ec2-user@ec2-50-19-54-72-compute-1.amazonaws.com` 来进行连接
3. 需要使用 `chmod 600 key_file.pem` 来修改权限
4. 如果需要 root，直接 `sudo` 即可

### Managing Security Groups

默认只为 SSH 开放 22 端口，这些都可以在 EC2 的控制面板中进行配置：

![](/images/14528910850178.jpg)

### 实例教程

假设已经创建好了一个 instance，这里通过一些实例演示，来进行展示和介绍：

![4.pi](/images/4.pic.jpg)

我们现在本机创建一个 `hello.sh` 的 shell 脚本，里面只有一句话：`echo 'Hello Da Wang'`，用作等下连接之后的测试

在初始化完毕后，通过公有 DNS 利用 SSH 来进行连接：

先把我们刚才写好的脚本文件传到 instance 中：`sudo scp -r -i demo.pem hello.sh  ec2-user@publicdns:\hello`

然后就可以访问了：

![c](/images/cc2.jpg)

可以看到已经远程执行了这个脚本，输出了 `Hello Da Wang`

我们再来试试看从 EC2 拷贝文件到本机，我们在 hello 文件夹里创建一个 `info.txt`，随便输入点内容，然后用 `udo scp -r -i demo.pem ec2-user@publicdns:\hello/info.txt ./` 就可以传输回来。

这些都做完后，我们终止(terminate)这个 EC2 instance

## Simple Storage Service(S3)

Amazon S3 提供了一个简单的用来存储和访问任何数据的 web 服务接口。可以存储的对象数量是无限的，每个对象都可以通过一个唯一的，由开发者指定的 key 来访问。

和常见的文件系统比较起来，S3 没有文件夹的概念，唯一的容器就是一个 bucket，但是文件名例可以有 `/` 来标记不同的层级。

[Amazon S3 入门视频（墙外）](https://youtu.be/1qrjFb0ZTm8)

主要介绍了如何创建/删除 bucket，添加/查看/移动/删除对象。基本的操作都比较简单，这里不介绍。

可以通过 web 界面，命令行工具和 API 三种方式来访问 S3。假设在 S3 中有一个文件的路径是

+ `s3://wdxtub/dawang.txt`

那么可以通过以下两种方式用 http 来访问：

+ `http://s3.amazonaws.com/wdxtub/dawang.txt`
+ `http://wdxtub.s3.amazonaws.com/dawang.txt`

其中 `wdxtub` 是 bucket 名，必须是全局唯一的。如果想要所有人都能够访问，需要设置一下对应的权限

## AWS Command Line Tool (aws-cli)

命令行工具可以最大程度的定制管理 AWS 的各种服务，linux 下的安装和操作可以看：

[AWS 命令行工具指南（墙外）](https://youtu.be/OSGjoMeHc2A)

在 Mac 下可以直接使用 `pip install awscli` 来进行安装，具体的教程可以参考[官方文档](https://aws.amazon.com/cn/cli/)

## CloudWatch

简单来说，CloudWatch 就是一个监控各种云服务的工具，可以参考下面的视频来了解：

[AWS Cloudwatch](https://youtu.be/cu2_AbfXn2k)

