title: 云计算 第 3 课 Azure 简介
categories:
- Technique
date: 2016-01-15 17:21:49
toc: true
tags:
- CMU
- 云计算
- Azure
---

现在我们已经了解了 AWS 的基本使用。接下来看看另一个云服务提供商 Microsoft Azure 的基本配置和使用。

<!-- more -->

---


## Microsoft Azure

简单来说，就是微软提供的云服务，提供 PaaS 和 IaaS 服务，于此同时支持不同的编程语言/工具/框架。在这门课中会设置好 Azure Virtual Machines (VMs) 以及使用其存储服务。

[Azure 介绍视频（墙外）](https://youtu.be/ZNgvZE0MLeo)

不想看视频的话，我帮大家看了，总结要点如下：

1. 为什么要用云呢？多快好省
2. 为什么要用微软的云呢？因为微软大法好，我们自己的服务也就是在 Azure 上
3. 包括什么呢？应用服务，数据服务，设施服务
4. 数据中心在哪里呢？美国欧洲中国日本澳大利亚
5. 我们的用户是谁？遍布世界各地
6. 『我们好靠谱的大家快来用吧！』

可以通过访问 [Azure Portal](https://portal.azure.com/) 或者 [Azure Manage](https://manage.windowsazure.com/) 来访问 Azure 控制台，其中 Azure Portal 是一个新的控制台，尽量用这个。需要更多信息，可以查看[这里](https://azure.microsoft.com/en-us/documentation/articles/fundamentals-introduction-to-azure)

## Microsoft Azure Compute

Azure 计算服务提供了可以配置启动登录安装运行软件的虚拟机，按照以下步骤来创建一个虚拟机：

1. 登录[Azure Portal](https://portal.azure.com/)
2. 新建 -> 计算 -> Ubuntu Server 14.04 LTS，选择 `资源管理器` 作为 `部署模型`
3. 设置姓名、用户名和密码，新建一个资源组，点击确定
4. 然后在大小选择中点击查看全部，选择 `A0 标准`，也就是最便宜的那个
5. 后面的大部分内容都可以用默认值，然后点击确定就创建完成了

Azure 有两种不同的部署模型：Azure Service Manager(ASM) 和 Azure Resource Manager(ARM)，后者更新也有更好的 API，所以我们在这门课中尽量会用这一种。

注意，在一种模型例部署的资源在另一种里看不到，更多信息可以访问[这里](http://blogs.msdn.com/b/cloud_solution_architect/archive/2015/03/17/rbac-and-the-azure-resource-manager.aspx)

### 实例教程

这里我们对 Azure 做上一讲中对 Amazon EC2 同样的事情：SSH 连接，双向复制文件，并尝试做一些基本操作。

新建好的虚拟机大概情况是这样：

![7.pi](/images/7.pic.jpg)

有了这些信息，就可以用 `ssh wdxtub@ipaddress` 来进行连接，连接成功之后会要求输入密码，密码验证之后就可以看到对应的命令行了。

![c](/images/cc3.jpg)

同样的，在服务器上新建一个叫做 `hello` 的文件夹，然后利用 `scp hello.sh wdxtub@ipaddress:\hello` 把我们的脚本文件上传上去。

![c](/images/cc4.jpg)

就可以看到我们在脚本中输出的内容了。然后我们新建一个文本文件，随便写点内容，利用 `scp wdxtub@ipaddress:\hello/info2.txt ./` 就可以把文件复制到本地了

操作完成之后记得停止虚拟机，不然运行着就不断在花钱的。

## Microsoft Azure Storage

类似于 Amazon S3，是一个可拓展的存储系统，有下面四种形态：

1. Azure Blobs，类似 Amazon S3 的对象存储系统
2. Azure Tables，数据库
3. Azure Queues，应用的消息队列
4. Azure Files，文件系统

现在我们主要接触一下 Azure Blobs，首先先要建立一个 Azure Storage 账户，按照下面步骤即可：

1. 新建 -> 数据+存储器 -> 存储(Storage account)
2. `部署模型` 选择 `资源管理器`，然后指定唯一的名字
3. 指定一个现有的资源组（刚才新建的那个就可以），然后在类型里选择 `L 本地冗余`（最便宜那个）
4. 然后就可以创建了

等待一段时间部署，部署完成后就可以通过 Azure 命令行工具来进行访问了。

![c](/images/cc5.jpg)

可以在服务这一栏里看到之前提到的四种类型。

## Microsoft Azure CLI

在[这里](https://azure.microsoft.com/en-us/downloads/) 的 Command-line tools 一栏中选择对应操作系统的版本下载安装，这里我会用 mac 版本。

安装很简单，下载下来一个 dmg 文件，然后一步步按照提示安装即可，安装完成之后就可以在终端里输入 `azure` 来使用了，类似下面这样：

![c](/images/cc6.jpg)

这门课里会使用 ARM/Azure Resource Manager(v2)，所以我们用 `azure config mode arm` 切换到这个模式，更多信息点击[这里](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-azurerm-versus-azuresm)

这一步比较麻烦，首先先要到[另一个控制台](https://manage.windowsazure.com)进行操作，在 Active
 Deirectory 页面下新建一个用户，然后会给你一个初始密码。然后再在设置中把刚才的用户添加位管理员。具体参考[创建用户](https://azure.microsoft.com/en-us/documentation/articles/active-directory-create-users/)

这些都做完之后，才可以用刚才给的帐号密码在[网页版](https://manage.windowsazure.com)登录，注意第一次需要修改密码，然后才可以用命令行登录，具体命令是：

```bash
azure login 
```

然后会给出一个地址，让你访问来进行验证，如下图所示：

![c](/images/cc7.jpg)

经过一系列的账户验证后，在命令行中会显示登录成功。就可以正常使用了，比如 `azure location list` 查看数据中心位置列表，更详细的教程在[这里](https://azure.microsoft.com/en-us/documentation/articles/azure-cli-arm-commands/)


