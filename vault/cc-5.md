title: 云计算 第 5 课 Azure API
categories:
- Technique
date: 2016-01-15 20:19:35
toc: true
tags:
- CMU
- 云计算
- Azure
- API
---

上一讲介绍了关于 SSH 和 Linux 的相关知识，这一讲我们来看看，如何通过代码，而不是界面或者命令行，来控制 Azure。

<!-- more -->

---

## Microsoft Azure

在 Azure 中，你没有办法通过 web 控制台来启动一个预先定义好的镜像。这一讲会介绍如何通过 Azure API 来创建一个虚拟机。

### Azure 验证和环境配置

这里先用 Azure 命令行工具来登录作为体验。

Azure 使用 Azure Active Directory(Azure AD) 服务来完成身份管理。我们首先需要从 Azure 云服务获取权限，才能使用 Java/Python API。

Azure 需要一个验证的 token，为了获得这个 token，需要完成以下步骤：

1. 安装 Azure 命令行工具
2. 用 Azure 命令行工具登录
3. 使用 Azure 命令行工具创建一个 AD 应用

访问[这里](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/)来获取对应于不同系统的安装包

安装完成后就可以登录，前面的步骤和[云计算 第 3 课 Azure 简介](http://wdxtub.com/2016/01/15/cc-3/) 最后的配置部分是一样的。

首先切换到 Azure Resource Manager (ARM) 模式来使用新的 API：`azure config mode arm`

然后用 `azure login` 登录，登录的时候会让你访问网址并填写对应的验证码，如下图：

![](/images/14529146793205.jpg)

然后就可以按照下面的步骤来在 Azure AD 中创建一个 AD 应用：

**第一步**

运行 `azure ad app create` 命令来创建一个新的 Azure AD 应用：

```bash
azure ad app create --name {Your Application Display Name} --home-page https://{YourApplicationHomePage} --identifier-uris https://{YouApplicationUri} --password {YourApplicationKey} 
```

成功的话，可以看到如下的结果：

![c](/images/cc9.jpg)

记下这里的 `AppId` 和 `Appkey`(就是刚才设置的密码)，等下获取 token 的时候要用

**第二步**

一个实例或者是一个 AD 应用被当做是 service principal 来进行引用，所以我们需要创建一个 service principal，使用刚才得到的 `AppId`

```bash
azure ad sp create {YourApplicationID}
```

成功之后会得到一个 object id，记下来，这个在下一步要用到。

![cc10](/images/cc10.jpg)

现在这个服务没有任何权限，我们接下来就是要给出权限

**第三步**

这一步需要提供 `subscriptionID`，可以在下图所示位置找到

![c](/images/cc11.jpg)

命令如下

```bash
azure role assignment create --objectId {objectID} -o Owner -c /subscriptions/{subscriptionId}/
```

成功后结果如下图：

![c](/images/cc12.jpg)

**第四步**

每个 Azure AD 应用都有一个独立的 `TenantId`，使用下面的命令来获取：

```bash
azure account list --json
```

如下图所示

![c](/images/cc13.jpg)

在使用 Java/Python API 时，需要提供的信息是 `AppId`, `AppKey`(密码) 和 `TenantId` 。

接下来我们需要把系统镜像复制到存储中。

我们会利用 Azure 命令行工具把系统镜像从课程账户赋值到自己的账户中，这在之后的 Project0 和 Project2 中会非常有用。具体步骤如下：

**第一步**

在资源组页面点击添加，创建一个新的资源组

![](/images/14529167646056.jpg)

**第二步**

起个名字，选择 East US，然后点击创建

![](/images/14529169051161.jpg)

**第三步**

在资源组例添加一个存储(storage account)

![](/images/14529169731509.jpg)

![](/images/14529169804085.jpg)

**第四步**

在部署模型中选择资源管理器，点击创建，并设定以下内容：

+ 名字
+ 类型：最便宜那种
+ 诊断：开启
+ 资源组：刚才新建的资源组
+ 位置：East US

![](/images/14529170662970.jpg)

**第五步**

来到刚才的创建的存储账户，点击 Blob

![](/images/14529171471889.jpg)

**第六步**

创建一个名为 `system` 的容器(container)，注意访问类型是 Blob

![](/images/14529172233371.jpg)

**第七步**

记下在设置/访问密钥里面的访问密钥 KEY1

![](/images/14529173037560.jpg)

**第八步**

现在我们的存储账户中已经有了容器，就可以把系统镜像拷贝过来了。需要复制的原因是只能由位于同一个存储账户中的镜像来创建 osdisk

执行下面的代码：

```bash
azure storage blob copy start https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/primertest-osDisk.7ec2e680-5a2f-462b-ba77-cd7b707389d4.vhd  --dest-account-name PUT_YOUR_STORAGE_ACCOUNT_NAME_HERE --dest-account-key PUT_YOUR_KEY1_HERE --dest-container system
```

如下图所示

![c](/images/cc14.jpg)

**第九步**

大概需要二十分钟，这期间可以通过命令行来查看载入的进度

```bash
azure storage blob copy show --account-name PUT_YOUR_STORAGE_ACCOUNT_NAME_HERE --account-key PUT_YOUR_KEY1_HERE --container system --blob Microsoft.Compute/Images/vhds/primertest-osDisk.7ec2e680-5a2f-462b-ba77-cd7b707389d4.vhd
```

当结果从 pending 变成 success 的时候，就可以用这个镜像来创建一台虚拟机了

**第十步**

复制完成后，点击 system 容器，记下镜像的 URL

![](/images/14529201199175.jpg)

有了镜像 URL，接下来就可以来创建新的虚拟机了。

### Azure Python API

为了介绍如何使用 Python API 来创建和管理 Azure 资源，这里通过一个例子来介绍如何通过一个预定义的镜像来创建虚拟机。

这个例子在之后的项目中非常有用。为了使用 Azure Python API 来创建虚拟机，需要完成以下几个步骤：

+ 安装 Python SDK
+ 通过 Python ID 来获取 AD 应用的 token
+ 把虚拟机镜像从课程账户拷贝到个人账户中
+ 利用提供的代码来部署一个虚拟机

首先通过 `pip install azure` 来安装 SDK。获取 AD 应用 token 需要 appId 和 appKey。在这门课中只需要创建一个 AD 应用。

如果遇到问题可以使用 `sudo -H pip install azure --upgrade --ignore-installed six`

其中：

`-H` set the home directory of the new user in the place of the original user's home. (without it, $HOME refers to the original user's home). 


每次执行下面的代码，都会拿到一个新的 token，这个 token 可以被用来创建资源管理对象。

```python
import requests
import os

# make sure you configure these three variables correctly before you try to run the code 
AZURE_ENDPOINT_URL='https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/oauth2/token'
AZURE_APP_ID=YOUR_APP_ID 
AZURE_APP_SECRET=YOUR_APP_SECRET

def get_token_from_client_credentials(endpoint, client_id, client_secret):
    payload = {
        'grant_type': 'client_credentials',
        'client_id': client_id,
        'client_secret': client_secret,
        'resource': 'https://management.core.windows.net/',
    }
    response = requests.post(endpoint, data=payload).json()
    return response['access_token']

# test
if __name__ == '__main__':
    auth_token = get_token_from_client_credentials(endpoint=AZURE_ENDPOINT_URL,
            client_id=AZURE_APP_ID,
            client_secret=AZURE_APP_SECRET)

    print auth_token
```

这里我们需要 OAUTH 2.0 的 endpoint URL，到 web 控制台->浏览->Active Directory。然后会跳转到另一个控制台，点击应用程序，再点击页面下方的查看端点，如下图所示

![](/images/14529493984204.jpg)

记录下 OAUTH 2.0 令牌端点的地址。

前面已经把位于 `https://cc15619.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/primertest-osDisk.7ec2e680-5a2f-462b-ba77-cd7b707389d4.vhd` 的镜像拷贝到了自己的账户中，接下来就可以开始创建了，首先从[这里](https://s3.amazonaws.com/15619public/webcontent/azure_demo_create_vm_from_ami.py)下载代码，其中的 `azure_api.create_vm_from_ami()` 函数在之后的 project 中会很有用。

也可以来参考 [Python SDK 文档](https://azure-sdk-for-python.readthedocs.org/en/latest/index.html)来找到所需的 API。

我们提供了一个 `demo()` 函数来展示如何使用，通过填写对应的参数来用以下代码运行 demo：

```bash
python azure_demo_create_vm_from_ami.py STORAGE_ACCOUNT_NAME SUBSCRIPTION_ID ENDPOINT_URI APPLICATION_ID APPLICATION_SECRET_KEY
```

这里出现了一些问题，后来发现是 python 的 requests 包版本过低导致的，使用 `pip install --upgrade requests` 解决。

运行的时候会有 InsecurePlatformWarning，这个是因为没有安装 `security` 包，用这条语句来安装 `$ pip install requests[security]` 或者直接安装 `$ pip install pyopenssl ndg-httpsclient pyasn1`。这之后 就会自动用加密的请求

![](/images/14529531648561.jpg)

完成之后就可以通过给出的 IP 地址来访问，用户名是 ubuntu，密码是 Cloud@123，然后就登录完成了

![c](/images/cc15.jpg)

记得在操作完成之后删除对应的资源。

### Azure Java API

为了介绍如何使用 Java API 来创建和管理 Azure 资源，这里通过一个例子来介绍如何通过一个预定义的镜像来创建虚拟机。

这个例子在之后的项目中非常有用。为了使用 Azure Java API 来创建虚拟机，需要完成以下几个步骤：

+ 安装 Java SDK
+ 通过 Java ID 来获取 AD 应用的 token
+ 把虚拟机镜像从课程账户拷贝到个人账户中
+ 利用提供的代码来部署一个虚拟机

我需要以下的 SDK

```
azure-mgmt-compute
azure-mgmt-utility
azure-mgmt-network
azure-mgmt-storage
```

按照 Java SDK 的意思是把这些 SDK 包含在项目中，方便起见，可以用`maven` 来编译和运行样例代码，步骤为：

1. 在电脑上安装 maven
2. 切换到 `azureDemo` 目录
3. 执行 `mvn compile` 与 `mvn exec:java -Dexec.mainClass="AzureVMApiDemo" [-Dexec.args="argument1 argument2 argument3 argument4 argument5"]`

和 Python 一样，我们需要获取到 `tenantID`, `applicationID`, `applicationKey` 并且把对应的镜像复制到自己的存储账户中。

如果之前已经创建过了，现在就可以直接下载代码来运行了。

因为我的机子上没有安装 maven，所以这里首先要进行安装。

先下载 [Maven](https://maven.apache.org/download.cgi)，然后解压到某个目录下，这里我解压到 `Users/dawang/apache-maven-3.3.9` 中。

然后打开终端，设置一下 Maven classpath

`$ vi ~/.bash_profile`

添加

```
 export M2_HOME=/Users/dawang/apache-maven-3.3.9
 export PATH=$PATH:$M2_HOME/bin
```

然后输入 `source ~/.bash_profile` 让命令生效，并用 `mvn -v` 来检验是否成功安装，如果出错，基本是因为没有设置 `JAVA_HOME` 环境变量所导致，重新编辑 `bash_profile` 增加

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_60.jdk/Contents/Home
```

重新输入 `source ~/.bash_profile` 让命令生效，就可以看到 maven 成功安装了，如下图：

![c](/images/cc16.jpg)

然后在[这里](https://s3.amazonaws.com/15619public/webcontent/azureDemo.tar.gz)下载代码，解压之后通过下面的命令来进行编译执行，注意添加对应的参数：

```bash
mvn compile && mvn exec:java -Dexec.mainClass="AzureVMApiDemo" -Dexec.args="RESOURCEGROUP STORAGEACCOUNT VHDNAME SUBSCRIPTIONID TENANTID APPLICATIONID APPLICATIONKEY"
```

其中

+ STORAGEACCOUNT, RESOURCEGROUP are corresponding to resources where you put the copied image;
+ VHDNAME is the copied disk image name under your account (it must end with a .vhd);
+ SUBSCRIPTIONID comes from your Azure account;
+ TENANTID, APPLICATIONID, APPLICATIONKEY are generated when you authenticated your service principal;(Please refer to previous sections)

疯狂下载之后开始配置，等待一段时间后会提示配置成功。这里需要注意 VHDNAME 里前面名字已经配置好了，直接输入文件名即可。

等一下就发现创建成功：

![c](/images/cc17.jpg)

完成之后就可以通过给出的 IP 地址来访问，用户名是 ubuntu，密码是 Cloud@123，然后就登录完成了，如下图：

![c](/images/cc18.jpg)

做完后记得删除对应的资源。


