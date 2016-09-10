title: 云计算 第 6 课 AWS API
categories:
- Technique
date: 2016-01-15 20:19:41
toc: true
tags:
- CMU
- 云计算
- AWS
- API
---

上一讲介绍了通过代码，而不是界面或者命令行，来控制 Azure，整个过程还是比较繁琐的，这一讲我们来看看，如何通过代码来控制 AWS。

<!-- more -->

---

AWS 可以通过命令行工具或者 SDK 来访问几乎所有的功能，这一讲主要介绍一些最常用的工具和功能。下面的视频是 AWS SDK 的一个简要介绍，与此同时给出了一个 Java SDK 的例子。

### Java AWS SDK 

[Java AWS SDK](http://aws.amazon.com/cn/sdk-for-java/) 包括：

+ AWS Java Library：总而言之就是所需的各种包，网络请求，验证，错误处理等等
+ 样例代码
+ Eclipse 支持：包含一个插件

[AWS SDK 视频介绍（墙外）](https://youtu.be/6Ru_f9WVIno)

AWS 使用 RESTful Web API，用 SDK 可以方便一些。

在[这里](http://aws.amazon.com/tools)找到对应的工具，找到 SDK 中对应的库。主要过程有三步：

1. 用密钥初始化 AWS 连接
2. 使用 API 进行操作
3. 关闭 AWS 连接

先下载好 SDK 和 Eclipse 插件（在 eclipse 里安装，Amazon 有给出教程）

### Amazon EC2 API

可以通过 API 做这些事情：

+ 创建 EC2 密钥对
+ 位实例创建安全组并且打开端口
+ 创建启动停止重启和终止 EC2 实例

下面是 Java 和 Python 的代码

Java:

```java
//Load the Properties File with AWS Credentials
Properties properties = new Properties();
properties.load(RunInstance.class.getResourceAsStream("/AwsCredentials.properties"));
BasicAWSCredentials bawsc = new BasicAWSCredentials(properties.getProperty("accessKey"), properties.getProperty("secretKey"));
//Create an Amazon EC2 Client
AmazonEC2Client ec2 = new AmazonEC2Client(bawsc);
//Create Instance Request
RunInstancesRequest runInstancesRequest = new RunInstancesRequest();
//Configure Instance Request
runInstancesRequest.withImageId("ami-3b44d352")
.withInstanceType("t1.micro")
.withMinCount(1)
.withMaxCount(1)
.withKeyName("project1_test")
.withSecurityGroups("MySecurityGroup");

//Launch Instance
RunInstancesResult runInstancesResult = ec2.runInstances(runInstancesRequest);   
//Return the Object Reference of the Instance just Launched
Instance instance=runInstancesResult.getReservation().getInstances().get(0);
```

Python:

```python
from boto.ec2.connection import EC2Connection

# Create a connection
conn = boto.ec2.connect_to_region("us-east-1")
# Launching instance
reservation = conn.run_instances("ami-3b44d352",instance_type = "t1.micro", 
        key_name = "project1_test", security_groups = "MySecurityGroup")
instance = reservation.instances[0]
```

我们同样可以通过 EC2 的 API 来查看实例的状态，例如类 `DescribeInstanceStatusRequest` 的状态：

+ Running
+ Pending
+ Shutting down 等等

更多的文档参考 [AWS SDK for Java Documentation](http://aws.amazon.com/documentation/sdk-for-java/) [AWS Java API Reference](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html) 和 [boto](http://docs.pythonboto.org/en/latest/)

Python 的部分这里不做详细介绍，具体可以参考 [Python SDK 视频（墙外）](https://youtu.be/7IOsOHJKxvY)

### Amazon 命令行 API 工具

我们同样也可以利用命令行来操作 AWS，主页在[这里](http://aws.amazon.com/cn/cli/)。如果打算使用命令行，那么需要：

1. 在实例中下载和安装命令行工具
2. 通过 `aws configure` 来设置密钥（注意这里地区要选择 us-east-1，[参考链接](http://docs.aws.amazon.com/general/latest/gr/rande.html)）

下面是简单的使用 AWS 命令行工具的视频介绍

[AWS 命令行工具（墙外）](https://youtu.be/OSGjoMeHc2A)

启动实例之前先要配置，如下：

```bash
$ aws configure
AWS Access Key ID [None]: YOUR AWS ACCESS KEY
AWS Secret Access Key [None]: YOUR AWS SECRET ACCESS KEY
Default region name [None]: us-east-1
Default output format [None]: json
```

然后可以用下面的命令来创建安全组，密钥对和角色

```bash
$ aws ec2 create-security-group --group-name devenv-sg --description "security group for development environment in EC2"
$ aws ec2 authorize-security-group-ingress --group-name devenv-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
$ aws ec2 create-key-pair --key-name devenv-key --query 'KeyMaterial' --output text > devenv-key.pem
$ chmod 400 devenv-key.pem
```

然后就可以启动并连接实例了，这里需要提供 `AMI_ID(ami-xxxxx)` 和 安全组(sg-xxxxx)

```bash
$ aws ec2 run-instances --image-id YOUR_AMI_ID --security-group-ids SECURITY_GROUP_ID --count 1 --instance-type t2.micro --key-name devenv-key --query 'Instances[0].InstanceId'
```

Amazon CloudWatch 部分之前有介绍过，详细介绍可以查看[文档](http://aws.amazon.com/documentation/cloudwatch/)


