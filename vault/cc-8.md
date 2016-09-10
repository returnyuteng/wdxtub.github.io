title: 云计算 第 8 课 Azure 动手玩
categories:
- Technique
date: 2016-01-16 14:41:09
toc: true
tags:
- CMU
- 云计算
- Azure
- 实例
---

上手 AWS 后，这一讲我们继续来看看另一个云服务提供商 Azure 要如何上手。

<!-- more -->

---

通常来说在云平台上用来计算的资源叫做虚拟机，也叫实例 instance。不是所有的实例都是一样的，在 AWS 中有一些可以定制的选择，比如说操作系统和硬件配置等等。

这一讲的主要任务是了解虚拟机并在上面跑一些应用程序，借此来理解不同配置的不同表现。我们都知道更好的配置就性能更强，但是：

+ 更贵的是不是真的更好？
+ 有没有什么办法来测量其性能表现？
+ 性能的差别有多少？

这些问题都不简单，但是这一讲会给你一点点启发。

我们首先会启动三种不同的 Azure 虚拟机，利用内置的性能测评来比较它们的相对性能表现，最后用公共访问的 web 服务器来测试性能。

项目的最后会探索一下名为 vertical scaling 的常见规模化技术。这个过程中需要把 Azure 虚拟机的镜像拷贝到自己的存储账户中（很耗时），所以最好尽快开始。

```bash
Data Center: https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/cc15619p0dcv2-osDisk.b7dbe1d0-782c-43b5-a7df-1896844377aa.vhd
Load Generator: https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/cc15619p0lgv2-osDisk.838bf28c-ed03-41f9-9810-4cbfa668d22e.vhd
```

## 启动虚拟机

我们先来创建几个通用的虚拟机，并在上面做一些试验

**第一步**

访问 [Azure portal](https://portal.azure.com/)并创建一个新的虚拟机，运行 Ubuntu 14.04 LTS，使用 资源管理器作为部署模型。地区选择 East US，以及 Azure pass subscription。

**第二步**

我们需要创建三个不同大小的虚拟机：A1, A2, A3

![](/images/14529892365625.jpg)

**第三步**

在设置->存储中，选择标准，并创建新的存储账户(Standard-LRS)和位于 East US 的网络

![](/images/14529894432878.jpg)

**第四步**

创建好了虚拟机之后，修改对应的网络安全组，在入站安全规则中添加 TCP 80 端口

![](/images/14529899780201.jpg)

![](/images/14529899840018.jpg)

注意三台机器都需要进行添加

**第五步**

给每个虚拟机打上标签

![](/images/14529900566991.jpg)

**第六步**

虚拟机正在运行的时候，就可以通过 IP 来进行访问了，如下：

```bash
ssh [username]@[Public IP address]
```

都登录上之后类似这样：

![c](/images/cc22.jpg)

## 系统性能测评

我们需要在已经启动的实例中安装性能测评工具。通过这个来评估完成一个任务所需的最佳实例组合以做到多快好省。

虽然云服务提供商是利用虚拟化技术来提供资源，但是需要知道实例的性能一来不是稳定的，二来也不保证达到某个水平。这里我们使用 `sysbench` 来评估性能，它是一个轻量级跨平台的性能测试软件，能够快速评估 CPU，内存和文件读写性能。

执行下面的命令安装：

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

表后面的三列分别是 A1, A2, A3 的总时间

Max Prime | Thread | Time(A1) | Time(A2) | Time(A3)
:--: | :--: | :--: | :--: | :--:
20000 | 8 | 74.8729s | 35.7929s | 18.6264s 
40000 | 1 | 195.5052s | 190.4013s | 194.2614s
50000 | 4 | 263.7678s | 125.2128 | 65.6631s

可以看到 Azure 的区分度比 AWS 大很多，不同等级基本上是一倍的性能提高。

### 文件 IO 性能测试

Azure 通常会保证 Input/Output Operations per Second (IOPS)。不同的硬盘访问逻辑对性能影响很大（缓存导致），传统的机械硬盘受寻址时间和旋转速度的影响，固态硬盘受驱动和设备控制器的影响。

同样用下面的命令来进行测试：

```bash
sysbench --test=fileio --file-total-size=20G prepare 
# 可能需要等待30分钟，使用 byobu / screen / tmux 来保持连接
# 或者可以在 iterm 设置中 profiles -> sessions -> When idel, send ASCII code
# 又或者 在客户端的 ~/.ssh/ 文件夹中添加 config 文件，并添加配置： ServerAliveInterval 60 

sysbench --test=fileio --file-total-size=20G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run
```

\ | A1 | A2 | A3
:--: | :--: | :--: | :--:
Read(Mb) | 167.81 | 170.62 | 195.94
Written(Mb) | 111.88 | 113.75 | 130.62
Total trans(Mb) | 279.69 | 284.38 | 326.56
Speed(Mb/sec) | 0.9547 | 0.9706 | 1.0884

比较一下表中的数据，是不是随着价格的提高，性能也有对应比例的提高？

可以看到从 A1 到 A3 几乎没有提升，而且与 aws 的相比，速度也是非常慢，估计是因为机械硬盘的缘故。

## Web 服务器性能测试

这门课上我们会学会如何配置，搭建和部署 web 服务。所以需要熟悉如何折腾服务器。这里我们会在每个实例中安装和部署一个简单的 web 服务器，修改一下主页以便测试和评分。

[Apache 简介视频（墙外）](https://youtu.be/6_tPobCyF9o)

LAMP = Linux + Apache + MySQL + PHP

用以下命令安装 apache：`sudo apt-get install apache2`

安装完成之后，用自己的浏览器访问这个实例的地址，应该就可以看到 Apache 的欢迎页面

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

先要把 load generator 和数据中心的镜像拷贝到自己的存储账户中。地址是：

```bash
Data Center: https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/cc15619p0dcv2-osDisk.b7dbe1d0-782c-43b5-a7df-1896844377aa.vhd

Load Generator: https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/cc15619p0lgv2-osDisk.838bf28c-ed03-41f9-9810-4cbfa668d22e.vhd
```

和 [云计算 第 5 课 Azure API](http://wdxtub.com/2016/01/15/cc-5/) 的方法是一样的，因为我们之前已经创建了 AD 应用之类的东西，有所需的 AppId, AppKey, TenantId 等内容，所以现在可以直接用之前的命令，但是要拷贝的内容换成上面的地址

```bash
# Data Center
azure storage blob copy start https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/cc15619p0dcv2-osDisk.b7dbe1d0-782c-43b5-a7df-1896844377aa.vhd  --dest-account-name PUT_YOUR_STORAGE_ACCOUNT_NAME_HERE --dest-account-key PUT_YOUR_KEY1_HERE --dest-container system

# Load Generator
azure storage blob copy start https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/cc15619p0lgv2-osDisk.838bf28c-ed03-41f9-9810-4cbfa668d22e.vhd  --dest-account-name PUT_YOUR_STORAGE_ACCOUNT_NAME_HERE --dest-account-key PUT_YOUR_KEY1_HERE --dest-container system
```

也需要一定时间，可以用下面的命令来查看进度

```bash
# Data Center
azure storage blob copy show --account-name PUT_YOUR_STORAGE_ACCOUNT_NAME_HERE --account-key PUT_YOUR_KEY1_HERE --container system --blob Microsoft.Compute/Images/vhds/cc15619p0dcv2-osDisk.b7dbe1d0-782c-43b5-a7df-1896844377aa.vhd

# Load Generator
azure storage blob copy show --account-name PUT_YOUR_STORAGE_ACCOUNT_NAME_HERE --account-key PUT_YOUR_KEY1_HERE --container system --blob Microsoft.Compute/Images/vhds/cc15619p0lgv2-osDisk.838bf28c-ed03-41f9-9810-4cbfa668d22e.vhd
```

类似下图

![c](/images/cc14.jpg)

当结果从 pending 变成 success 的时候，就可以用这个镜像来创建一台虚拟机了，具体镜像的 url 可以在存储账户中对应镜像的

创建镜像同样可以用[云计算 第 5 课 Azure API](http://wdxtub.com/2016/01/15/cc-5/)中的 python 脚本来创建，我创建了两个新文件：`azure_p0_create_lg.py` 和 `azure_p0_create_dc.py`

这两个文件基本只需要改动 `create_vm_from_ami` 这个函数，因为参数默认值只有在参数不够的时候才会启用，所以保险起见，在函数体里面把需要更新的内容都再写一次。

这里需要注意 `machine_size` 并没有对应的 D1 类型，所以先随便创建一个，之后去 web 控制台里进行修改

命令如下：

```bash
# lg
python azure_p0_create_lg.py STORAGE_ACCOUNT_NAME SUBSCRIPTION_ID ENDPOINT_URI APPLICATION_ID APPLICATION_SECRET_KEY

#dc
python azure_p0_create_dc.py STORAGE_ACCOUNT_NAME SUBSCRIPTION_ID ENDPOINT_URI APPLICATION_ID APPLICATION_SECRET_KEY
```

创建完成之后，先把 load generator 在 Web 界面改成 D1（可能需要停止虚拟机才能调整）

然后访问 load generator 的 ip，可以看到下面的界面

![c](/images/cc23.jpg)

注意测试的时候数据中心需要填写 dns 而不是 ip 地址，可以在下图这里找到：

![](/images/14529979521284.jpg)

然后按照与上一讲类似的过程，记录下 A1 A2 两种数据中心的测试结果即可。

RPS | Minute 1 | Minute 2 | Minute 3 | Minute 4 | Minute 5
:--: | :--: | :--: | :--: | :--: | :--:
A1 | 494.42 | 704.12 | 712.61 | 715.48 | 718.92
A2 | 991.50 | 1298.85 | 1322.22 | 1329.14 | 1324.32

可以看到从 A1 到 A2 的提升还是非常大的（从配置中也能看得出来）

完成之后记得删除对应的虚拟机


