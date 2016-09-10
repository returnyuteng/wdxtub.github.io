title: 云计算 Twitter 语料分析 3 Vert.x 配置部署
categories:
- Technique
toc: true
date: 2016-03-01 15:42:05
tags:
- CMU
- 云计算
- 服务
- Vertx
---

我们需要比较两个不同的 web 框架的性能，于是也选择了当下比较热门的 Vert.x 框架来做比较，网上的中文资源还是比较少的，这里同样记录一下如何在 EC2 上搭建和部署 Vert.x。

<!-- more -->

---

## 环境配置

先启动一个标准的 Ubuntu 镜像（因为有 apt-get 安装软件比较方便）

我们需要安装 java 和 maven，并配置好对应的路径，具体参考下面的命令，这里和之前不同的是需要安装 Java 8，以及 maven 3：

```bash
# 安装 java,maven
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo add-apt-repository "deb http://ppa.launchpad.net/natecarlson/maven3/ubuntu precise main"
sudo apt-get update
sudo apt-get install maven3

# 设置默认 jdk
sudo update-alternatives --config java

# 配置 Java Home 编辑 ~/.bashrc
JAVA_HOME=/usr/
export JAVA_HOME
PATH=$PATH:$JAVA_HOME
export PATH
```

## Hello World

同样用 maven 来创建项目，这次我们直接手动现在本地建立如下所示的文件层级：

![项目目录](/images/14568662369892.jpg)

然后我们修改 `pom.xml` 文件，具体如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>housailei.vertx</groupId>
  <artifactId>vertx-server</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>io.vertx</groupId>
      <artifactId>vertx-core</artifactId>
      <version>3.0.0</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.3</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>

        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-shade-plugin</artifactId>
          <version>2.3</version>
          <executions>
            <execution>
              <phase>package</phase>
              <goals>
                <goal>shade</goal>
              </goals>
              <configuration>
                <transformers>
                  <transformer
                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                    <manifestEntries>
                      <Main-Class>io.vertx.core.Starter</Main-Class>
                      <Main-Verticle>housailei.vertx.App</Main-Verticle>
                    </manifestEntries>
                  </transformer>
                </transformers>
                <artifactSet/>
                <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-fat.jar</outputFile>
              </configuration>
            </execution>
          </executions>
        </plugin>
    </plugins>
  </build>
</project>
```

然后我们在 `src/main/java/housailei/vertx/` 下创建一个 `App.java` 文件：

```java
package housailei.vertx;

import io.vertx.core.AbstractVerticle;
import io.vertx.core.Future;

public class App extends AbstractVerticle {

  @Override
  public void start(Future<Void> fut) {
    vertx
        .createHttpServer()
        .requestHandler(r -> {
          r.response().end("<h1>Hello from my first " +
              "Vert.x 3 application! config by dawang</h1>");
        })
        .listen(8080, result -> {
          if (result.succeeded()) {
            fut.complete();
          } else {
            fut.fail(result.cause());
          }
        });
  }
}
```

然后我们把代码上传回服务器上：`scp -i ../group.pem -r ./* ubuntu@dns.amazonaws.com:~/vertx-server/`

执行的话稍微麻烦一点

```bash
mvn3 clean package
java -jar target/vertx-server-1.0-SNAPSHOT-fat.jar
```

正常运行如下图所示：

![正常运行截图](/images/14568725986794.jpg)

## 实现 REST

前面的代码虽然可以工作，我们没办法设定不同的 api，也没办法做更进一步的处理，所以我们现在来更进一步，实现一个 RESTful 的简易 API。

我们先要在 `pom.xml` 文件中添加依赖

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-web</artifactId>
  <version>3.2.1</version>
</dependency>
```

然后对应修改 `start` 方法，如下：

```java
public void start(Future<Void> fut) {

   // Create a router object.
   Router router = Router.router(vertx);

   // Bind "/" to our hello message - so we are still compatible.
   router.route("/").handler(routingContext -> {
   HttpServerResponse response = routingContext.response();
   response
      .putHeader("content-type", "text/html")
      .end("<h1>Hello from my first " +
         "Vert.x 3 application! config by dawang</h1>");
   });


   // Create the HTTP server and pass the "accept" method to the request handler.
   vertx.createHttpServer().requestHandler(router::accept)
       .listen(
       // Retrieve the port from the configuration,
       // default to 8080.
       config().getInteger("http.port", 8080), result -> {
           if (result.succeeded()) {
               fut.complete();
           } else {
               fut.fail(result.cause());
           }
       });
}
```

在开始start方法里创建了一个 `Router` 对象。router 是 Vert.x Web 的基础，负责分发 HTTP 请求到 handler（处理器），在Vert.x Web中还有两个很重要的概念。

+ Route - 定义请求的分发
+ Handler - 这是实际处理请求并且返回结果的地方。Handlers可以被链接起来使用。

如果明白了这3个概念（Router、Routes、Handlers），就能明白 Vert.x Web 了。

重新运行一次，可以看到结果如下：

![再次配置成功](/images/14568771965878.jpg)

然后我们多定义几个接口，并对应不同的方法来实现，同样是在 `start` 方法中，添加两条 router 规则：

```java
router.get("/api/hou").handler(this::HouHandler);
router.get("/api/sai").handler(this::SaiHandler);
```

然后创建对应的方法（就不用都写在一个函数里了）：

```java
private void HouHandler(RoutingContext routingContext){
   routingContext.response()
       .putHeader("content-type", "application/json; charset=utf-8")
       .end("This is Hou HOu HOU!!! API!");
}

private void SaiHandler(RoutingContext routingContext){
   routingContext.response()
       .putHeader("content-type", "application/json; charset=utf-8")
       .end("This is Sai SAi SAI!!! API!");
}
```

我们再测试一下，可以看到 api 已经启用了：

![Hou Content](/images/14568776771132.jpg)

![Sai Content](/images/14568776972444.jpg)

有了这些，我们就可以自己来进行操作了，虽然可能代码丑一些，不过易上手，容易改。

## 参考资料

+ [官方文档](http://vertx.io/docs/)
+ [Vertx Core 手册](http://vertx.io/docs/vertx-core/java/)
+ [Vertx Core API 文档](http://vertx.io/docs/apidocs/)
+ [Vertx Web 手册](http://vertx.io/docs/vertx-web/java/)
+ [Vertx Web API 文档](http://vertx.io/docs/apidocs/)

非常有用的新手入门教程

+ [My first Vert.x 3 Application](http://vertx.io/blog/my-first-vert-x-3-application/index.html)
+ [Vert.x Application Configuration](http://vertx.io/blog/vert-x-application-configuration/)
+ [Some Rest with Vert.x](http://vertx.io/blog/some-rest-with-vert-x/)
+ [Unit and Integration Tests](http://vertx.io/blog/unit-and-integration-tests/)
+ [中文机器翻译版本](https://github.com/quanke/vertx3_study_demo)


