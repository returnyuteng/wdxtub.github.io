title: 云计算 第 4 课 SSH, Linux & Project
categories:
- Technique
date: 2016-01-15 20:19:30
toc: true
tags:
- CMU
- 云计算
- SSH
- Linux
- 项目
---

了解了基本的云平台之后，我们现在来学习一下刚才我们用到的 SSH 和 Linux，最后介绍做项目的要求。

<!-- more -->

---

## Secure Shell(SSH)

简单来说 SSH 是一个用来进行安全数据通信、远程登录和远程命令执行的网络协议。在 EC2 instance 上使用 SSH 与平时相比有一些小小的变化，将在下面介绍。

![](/images/14529074027221.jpg)
（SSH 连接 EC2 的过程）

### 设置 SSHD

在 Linux 上使用 `sudo apt-get install openssh-server`，配置文件在 `/etc/ssh/sshd.config` 中。可以使用 `sudo /etc/init.d/ssh` 或 `sudo service sshd [start|stop|restart]` 来进行管理

### 验证机制

SSH 支持若干中验证机制。比较常见的是通过密码验证。密码验证的安全性取决于密码的长度和复杂度，由于可以被暴力或者基于字典破解，所以对于公共访问的 EC2 instance 来说是比较危险的。

为此，所有的 EC2 instance 都使用基于公钥加密的 key-based 验证。一对密钥包含公钥和私钥，由数学方式如 RSA/DSA 生成并链接。只知道一个 key 的话，几乎不可能破解另外一个 key。

我们也可以生成自己的密钥对，在 Linux 中，密钥对通常存储在 `~/.ssh` 目录下，通过下面的代码就可以生成密钥对：

```bash
# Generate a new key with your email id as a label
ssh-keygen -t rsa -b 4096 -C "email_id@domain.com"

# Enter the file where you want to save the key: (recommended - choose default)

# You will be asked enter a pass-phrase for your key twice.
# (Use a strong pass-phrase. Longer pass-phrases are more secure than shorter ones.)

# After you provide the pass-phrase. The console will print the location of the key and the key fingerprint
```

然后就可以导入这个密钥并用以连接 EC2，具体的做法可以看[这里](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)

举个例子，下面的代码会在远程实例执行 `uname -a` 命令，这种做法在批量运行远程实例命令的时候非常有用：

```bash
ssh -i private_key_file.pem some-instance.ec2.amazonaws.com 'uname –a'
```

如果遇到 SSH 的 permission-denied 问题，很可能是由于不正确权限所导致，私钥的 unix permission 为 `600` 且包含私钥的目录的权限应为 `700`

在执行 SSH 命令时，如果太久没有操作，就会自动断开，执行的脚本也会终止，如果不希望这样，有以下几种方式可以避免 SSH 超时：

1. [关闭 SSH 客户端的超时](https://docs.oseems.com/general/application/ssh/disable-timeout)
2. 用 `nohup` 执行脚本
3. 通过远程屏幕管理(remote sreen management)来保持 terminal sessions

### Remote Screen Management

通常来说我们用 bash shell 来进行远程操作，但是由于网络的缘故，一旦连接断开，所有已经启动的进程都会被终止。为了避免这种情况，使用 `nohup` 命令就非常有用了，即使连接断开，仍旧会继续运行命令。

另外一个比较有用的工具是 `byobu`，是一个多终端管理应用程序，类似于 `screen` 或者 `tmux`。具体可以参考下图：

![](/images/14529090638274.jpg)

## Linux 必备技能

这门课基本上会在 Linux 环境下工作，所以这里给出一些基本的概念和技巧。下面是一些教程：

1. [The Advanced Bash Scripting Guide](http://tldp.org/LDP/abs/html/)
2. [The Art of the Command Line](https://github.com/jlevy/the-art-of-command-line) [中文版本](https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md)
3. [Perl 指南](http://www.perl.org/)
4. [Python 指南](http://www.python.org/)

读的时候需要注意下面的内容：

1. 脚本语法，本地变量和环境变量
2. bash 中的 Quoting and back-ticks
3. 利用 unix 管道进行输入输出重定向
4. 使用类似 `grep`, `awk`, `sed`, `cut` 等等来做基本的文本操作

### 安装软件

用 Ubuntu 的话可以使用强大的 `apt-get` 来安装软件，例如

```bash
sudo apt-get install emacs
sudo apt-get install vim

sudo apt-get update # 用来更新包索引
sudo apt-get upgrade # 用来更新包
```

### 文件所有者和权限

Linux 的文件权限分为三个用户组：`user`, `group` 和 `others`，每组有三个权限：读(`r`)，写(`w`)，执行(`x`)，具体可以参考下表

![c](/images/cc8.jpg)

然后我们可以利用 `chmod` 命令来修改权限，如：`chmod 777 filename.txt`

不用数字的话，可以用字母，`chmod u+x filename.txt` 这个命令就给 user 添加了执行权限。

如果要更改文件所属，使用 `chown` 命令，具体可以参见 `man chown`

### Linux 中的磁盘操作

#### 管理分区

许多 Linux 应用都可以管理磁盘分区，比如说 `parted` 和 `fdisk`。`fdisk` 是比较老的分区管理工具，现在几乎已经被 `parted` 取代，因为 `parted` 支持 GUID 分区表以及大于 2TB 的磁盘，所以我们这里使用 `parted`。

`parted -l` 会列出系统中的所有分区。具体的顺序是按照 `/dev` 下的顺序。在 EC2 实例中，设备是 `dev/xvd**`，`/dev/xvda1` 是操作系统分区，`/dev/xvdb` 是实例的存储分区，注意 RAMDISK 不会出现在 `parted` 的输出当中。

`parted` 还可以用来为每个分区设置类型和文件系统，例如

```bash
parted /dev/xvdX mklabel gpt
parted /dev/xvdX mkpart db ext4 0% 10G
```

如果不加任何参数，`parted` 就会进入交互模式，这里你可以利用 `mkfs` 来格式化一个分区，如 `mkfs.ext4 /dev/xvdX1`

#### 挂载分区

在分区和格式化之后，如果想要使用就需要把分区挂载在某个挂载点上，通常来说会挂载在 `/mnt` 或者 `/mount` 上，可移除的媒介，例如 USB 和 CDROM，会挂载在 `/media` 上，当然，你可以把分区挂载在任何地方，如：

```bash
mkdir /storage/mountpoint
mount /dev/yourdevice /storage/mountpoint
```

### 配置服务和启动

如果想要开机启动，需要参考应用本身的帮助文档，但是总体来说，服务需要安装在 `/etc/init.d` 中，这使得这些脚本可以在启动时运行。例如，如果 mysql 脚本在 `/etc/init.d` 中，那么可以使用这个命令 `service mysql [start|stop|restart]`。

想要了解更多，请参阅[这里](http://www.tldp.org/HOWTO/HighQuality-Apps-HOWTO/boot.html)

## 项目

早开始，早学习，多测试，注意截止时间。即使 AWS 或者 Azure 挂掉也不会推迟截止日期，所以最保险的还是提早开始做（其实做什么事情都是这样）。

可能的情况下尽量用 spot instance，可以给学校省点钱。当然后面的 project 也会有具体的要求，就跟着做即可。

### 管理花费

每个 project 都有花费的限制，可以在 AWS 账户活动页面查看用费状况，如下图所示：

![](/images/14529122617926.jpg)

### 给 AWS 资源添加标签

AWS 资源，包括 EC2 实例，S3 Bucket 等都可以通过键值对来打标签，做每个 project 的时候都要打上对应的标签，例如：

![](/images/14529125832425.jpg)

打标签是必须的，不然会扣分。

注意 spot 实例创建和 spot 实例请求是两个分开的过程，所以得确保在实例启动的时候单独打赏标签。

### 评价和打分

某些项目中会使用教学人员提供的 AMI，包含所需的所有数据，通常来说会包含两个文件：

1. `runner.sh`：bash 脚本，每个问题对应一个函数，完成这些函数并运行脚本来验证答案
2. `submitter`：验证答案之后用这个来提交，需要用到 andrewID 和 提交密码，提交密码可以在每个项目页面的顶端找到

因为只能提交一次，所以确保提交前都弄好。还需要提交对应的源代码，会通过以下的方式来评估：

1. 完成提到的任务
2. 效率和性能
3. 编码风格，可读性，注释，建议参考 [Google Style Guide](https://github.com/google/styleguide)

