title: 云计算 Twitter 语料分析 2 Undertow 配置部署
categories:
- Technique
toc: true
date: 2016-03-01 09:41:11
tags:
- CMU
- 云计算
- 服务
- Undertow
---

因为项目有一定的性能要求，所以我们选择 Undertow 这个微框架来降低框架本身带来的性能影响，但是因为比较小众，所以网上很多资料都不全，这里记录下具体在 EC2 上如何配置和部署 Undertow。

<!-- more -->

---

## 环境配置

先启动一个标准的 Ubuntu 镜像（因为有 apt-get 安装软件比较方便）

我们需要安装 java 和 maven，并配置好对应的路径，具体参考下面的命令：

```bash
# 安装 java,maven
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer
# 可能需要先删除原来的 maven2
sudo apt-get remove maven2
sudo apt-get update
sudo apt-get install maven # 安装 maven3，2 在之后会有 bug

# 设置默认 jdk
sudo update-alternatives --config java

# 配置 Java Home 编辑 ~/.bashrc
JAVA_HOME=/usr/lib/jvm/java-7-oracle
export JAVA_HOME
PATH=$PATH:$JAVA_HOME
export PATH
```

如果一切正常的话，使用 `java -version` 可以看到：

![java -version](/images/14568475090854.jpg)

## Hello World

我们创建一个项目来搭建服务器，因为 undertow 是使用 maven 来管理包和依赖的，所以我们也直接用 maven 来创建项目

```bash
# 创建项目，注意设置包名和项目名称
mvn archetype:generate -DgroupId=housailei.undertow -DartifactId=p1_front -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DartifactId=undertow-server
```

为了编辑方便，我们把项目复制到本地

`scp -i group.pem -r ubuntu@dns.amazonaws.com:~/undertow-server/* ./`

 编辑完成可以用下面的命令上传回去（注意所在文件夹，我这里新建了一个文件夹用来存放源代码，密钥放在上一层）

`scp -i ../group.pem -r ./* ubuntu@dns.compute-1.amazonaws.com:~/undertow-server/ `

![目录层级](/images/14568478192103.jpg)

我们需要对 `App.java` 和 `pom.xml` 做一些修改

```java
package housailei.undertow;
import io.undertow.Undertow;
import io.undertow.server.*;
import io.undertow.util.Headers;

public class App
{
    public static void main( String[] args )
    {
        // 设置成 0.0.0.0 开放访问以便测试
        Undertow server = Undertow.builder().addHttpListener(8080, "0.0.0.0")
            .setHandler(new HttpHandler() {
           public void handleRequest(final HttpServerExchange exchange)
                       throws Exception {
                   exchange.getResponseHeaders().put(Headers.CONTENT_TYPE,
                           "text/plain");
               exchange.getResponseSender().send("Hello World! This is wdxtub.");
               }
           }).build();
        server.start();
    }
}
```

对 pom 文件的修改主要就是加上各类依赖，已经添加构建插件，我们这里选用了最新的 undertow，具体需要添加以下两个部分：

```xml
<dependency>
 <groupId>io.undertow</groupId>
 <artifactId>undertow-core</artifactId>
 <version>1.3.18.Final</version>
</dependency>

<dependency>
   <groupId>io.undertow</groupId>
   <artifactId>undertow-servlet</artifactId>
   <version>1.3.18.Final</version>
</dependency>
```

和 

```xml
<build>
 <plugins>
    <plugin>
       <groupId>org.codehaus.mojo</groupId>
       <artifactId>exec-maven-plugin</artifactId>
       <version>1.2.1</version>
       <executions>
          <execution>
             <goals>
                <goal>java</goal>
             </goals>
          </execution>
       </executions>
       <configuration>
          <mainClass>housailei.undertow.App</mainClass>
       </configuration>
    </plugin>
 </plugins>
</build>
```

然后上传回 EC2 实例，就可以用以下代码执行：

```bash
mvn compile && mvn exec:java
```

服务器正常开启之后，我们就可以在浏览器中访问了：

![访问网站](/images/14568488280327.jpg)


## 添加 Servlet

现在我们的服务器基本除了展示个页面没办法做任何事情，我们需要能让服务器运行 servlet 才行（最新版本的 undertow 会有一些小问题，会具体标记出来）

我们先写两个简单的 servlet，具体代码如下：

```java
// 第一个 servlet
public class HouServlet extends HttpServlet {

    private String message;
    public void init() throws ServletException{
        message = "Hou HOu HOU!!!! Servlet!!";
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        out.println("<h1>" + message + "</h1>");
    }

    public void destroy() {

    }
}

// 第二个 servlet
public class SaiServlet extends HttpServlet {

    private String message;
    public void init() throws ServletException{
        message = "Sai SAi SAI!!!! Servlet!!";
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        out.println("<h1>" + message + "</h1>");
    }

    public void destroy() {

    }
}
```

然后修改 `App.java` 把这两个 servlet 载入进去

```java
public class App
{
    public static final String MYAPP = "/hsl";

    public static void main( String[] args ){
        try {
            // 官方例子中使用的 addServlets 方法不可用
            // 这里我用了 addServlet 方法
            DeploymentInfo servletBuilder = Servlets.deployment()
                .setClassLoader(App.class.getClassLoader())
                .setContextPath(MYAPP)
                .setDeploymentName("test.war")
                .addServlet(
                        Servlets.servlet("HouServlet", HouServlet.class)
                                .addMapping("/hou"))
                .addServlet(
                        Servlets.servlet("SaiServlet", SaiServlet.class)
                                .addMapping("/sai"));

            DeploymentManager manager = Servlets.defaultContainer().addDeployment(servletBuilder);
            manager.deploy();

            HttpHandler servletHandler = manager.start();
            PathHandler path = Handlers
                .path(Handlers.redirect(MYAPP))
                .addPrefixPath(MYAPP, servletHandler);

            Undertow server = Undertow.builder()
                .addHttpListener(8080, "0.0.0.0")
                .setHandler(path)
                .build();
            server.start();

        } catch (ServletException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```

接着还是传到服务器上并 `mvn compile && mvn exec:java`，就可以看到结果

![SaiServlet 结果](/images/14568573432825.jpg)

![HouServlet 结果](/images/14568573815710.jpg)

之后的任务就可以在 Servlet 的 `doGet` 方法中对应写代码完成了。

后面应该会写一些脚本把配置的工作自动化，因为每次新建 EC2 都得重新配置还是挺麻烦的。

## 参考资料

+ [官方文档](http://undertow.io/undertow-docs/undertow-docs-1.3.0/index.html#introduction)
+ [官方 JavaDoc](http://undertow.io/javadoc/1.3.x/index.html)
+ [官方样例代码](https://github.com/undertow-io/undertow/tree/master/examples/src/main/java/io/undertow/examples)



