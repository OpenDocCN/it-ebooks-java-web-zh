# VI. 部署到云端

### 部署到云端

对于大多数流行云 PaaS（平台即服务）提供商，Spring Boot 的可执行 jars 就是为它们准备的。这些提供商往往要求你带上自己的容器；它们管理应用的进程（不特别针对 Java 应用程序），所以它们需要一些中间层来将你的应用适配到云概念中的一个运行进程。

两个流行的云提供商，Heroku 和 Cloud Foundry，采取一个打包（'buildpack'）方法。为了启动你的应用程序，不管需要什么，buildpack 都会将它们打包到你的部署代码：它可能是一个 JDK 和一个 java 调用，也可能是一个内嵌的 webserver，或者是一个成熟的应用服务器。buildpack 是可插拔的，但你最好尽可能少的对它进行自定义设置。这可以减少不受你控制的功能范围，最小化部署和生产环境的发散。

理想情况下，你的应用就像一个 Spring Boot 可执行 jar，所有运行需要的东西都打包到它内部。

# 49\. Cloud Foundry

### 49\. Cloud Foundry

如果不指定其他打包方式，Cloud Foundry 会启用它提供的默认打包方式。Cloud Foundry 的[Java buildpack](https://github.com/cloudfoundry/java-buildpack)对 Spring 应用有出色的支持，包括 Spring Boot。你可以部署独立的可执行 jar 应用，也可以部署传统的.war 形式的应用。

一旦你构建了应用（比如，使用`mvn clean package`）并[安装](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html)了 cf[命令行工具](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html)，你可以使用下面的`cf push`命令（将路径指向你编译后的.jar）来部署应用。在发布一个应用前，确保你已登陆 cf 命令行客户端。

```java
$ cf push acloudyspringtime -p target/demo-0.0.1-SNAPSHOT.jar 
```

查看`cf push`[文档](http://docs.cloudfoundry.org/devguide/installcf/whats-new-v6.html#push)获取更多可选项。如果相同目录下存在[manifest.yml](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)，Cloud Foundry 会使用它。

就此，cf 开始上传你的应用：

```java
Uploading acloudyspringtime... OK
Preparing to start acloudyspringtime... OK
-----> Downloaded app package (8.9M)
-----> Java Buildpack source: system
-----> Downloading Open JDK 1.7.0_51 from .../x86_64/openjdk-1.7.0_51.tar.gz (1.8s)
       Expanding Open JDK to .java-buildpack/open_jdk (1.2s)
-----> Downloading Spring Auto Reconfiguration from  0.8.7 .../auto-reconfiguration-0.8.7.jar (0.1s)
-----> Uploading droplet (44M)
Checking status of app 'acloudyspringtime'...
  0 of 1 instances running (1 starting)
  ...
  0 of 1 instances running (1 down)
  ...
  0 of 1 instances running (1 starting)
  ...
  1 of 1 instances running (1 running)

App started 
```

恭喜！应用现在处于运行状态！

检验部署应用的状态是很简单的：

```java
$ cf apps
Getting applications in ...
OK

name                 requested state   instances   memory   disk   urls
...
acloudyspringtime    started           1/1         512M     1G     acloudyspringtime.cfapps.io
... 
```

一旦 Cloud Foundry 意识到你的应用已经部署，你就可以点击给定的应用 URI，此处是[acloudyspringtime.cfapps.io/](http://acloudyspringtime.cfapps.io/)。

# 49.1\. 绑定服务

### 49.1\. 绑定服务

默认情况下，运行应用的元数据和服务连接信息被暴露为应用的环境变量（比如，$VCAP_SERVICES）。采用这种架构的原因是因为 Cloud Foundry 多语言特性（任何语言和平台都支持作为 buildpack）。进程级别的环境变量是语言无关（language agnostic）的。

环境变量并不总是有利于设计最简单的 API，所以 Spring Boot 自动提取它们，然后将这些数据导入能够通过 Spring `Environment`抽象访问的属性里：

```java
@Component
class MyBean implements EnvironmentAware {

    private String instanceId;

    @Override
    public void setEnvironment(Environment environment) {
        this.instanceId = environment.getProperty("vcap.application.instance_id");
    }

    // ...

} 
```

所有的 Cloud Foundry 属性都以 vcap 作为前缀。你可以使用 vcap 属性获取应用信息（比如应用的公共 URL）和服务信息（比如数据库证书）。具体参考 VcapApplicationListener Javadoc。

**注**：[Spring Cloud Connectors](http://cloud.spring.io/spring-cloud-connectors/)项目很适合比如配置数据源的任务。Spring Boot 提供自动配置支持和一个`spring-boot-starter-cloud-connectors` starter POM。

# 50\. Heroku

### 50\. Heroku

Heroku 是另外一个流行的 Paas 平台。想要自定义 Heroku 的构建过程，你可以提供一个`Procfile`，它提供部署一个应用所需的指令。Heroku 为 Java 应用分配一个端口，确保能够路由到外部 URI。

你必须配置你的应用监听正确的端口。下面是用于我们的 starter REST 应用的 Procfile：

```java
web: java -Dserver.port=$PORT -jar target/demo-0.0.1-SNAPSHOT.jar 
```

Spring Boot 将`-D`参数作为属性，通过一个 Spring 的 Environment 实例访问。`server.port`配置属性适合于内嵌的 Tomcat，Jetty 或 Undertow 实例启用时使用。`$PORT`环境变量被分配给 Heroku Paas 使用。

Heroku 默认使用 Java 1.6。只要你的 Maven 或 Gradle 构建时使用相同的版本就没问题（Maven 用户可以设置`java.version`属性）。如果你想使用 JDK 1.7，在你的 pom.xml 和 Procfile 临近处创建一个 system.properties 文件。在该文件中添加以下设置：

```java
java.runtime.version=1.7 
```

这就是你需要做的一切。对于 Heroku 部署来说，经常做的工作就是使用`git push`将代码推送到生产环境。

```java
$ git push heroku master

Initializing repository, done.
Counting objects: 95, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (78/78), done.
Writing objects: 100% (95/95), 8.66 MiB | 606.00 KiB/s, done.
Total 95 (delta 31), reused 0 (delta 0)

-----> Java app detected
-----> Installing OpenJDK 1.7... done
-----> Installing Maven 3.2.3... done
-----> Installing settings.xml... done
-----> executing /app/tmp/cache/.maven/bin/mvn -B
       -Duser.home=/tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229
       -Dmaven.repo.local=/app/tmp/cache/.m2/repository
       -s /app/tmp/cache/.m2/settings.xml -DskipTests=true clean install

       [INFO] Scanning for projects...
       Downloading: http://repo.spring.io/...
       Downloaded: http://repo.spring.io/... (818 B at 1.8 KB/sec)
        ....
       Downloaded: http://s3pository.heroku.com/jvm/... (152 KB at 595.3 KB/sec)
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/target/...
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/pom.xml ...
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 59.358s
       [INFO] Finished at: Fri Mar 07 07:28:25 UTC 2014
       [INFO] Final Memory: 20M/493M
       [INFO] ------------------------------------------------------------------------

-----> Discovering process types
       Procfile declares types -> web

-----> Compressing... done, 70.4MB
-----> Launching... done, v6
       http://agile-sierra-1405.herokuapp.com/ deployed to Heroku

To git@heroku.com:agile-sierra-1405.git
 * [new branch]      master -> master 
```

现在你的应用已经启动并运行在 Heroku。

# 51\. Openshift

### 51\. Openshift

[Openshift](https://www.openshift.com/)是 RedHat 公共（和企业）PaaS 解决方案。和 Heroku 相似，它也是通过运行被 git 提交触发的脚本来工作的，所以你可以使用任何你喜欢的方式编写 Spring Boot 应用启动脚本，只要 Java 运行时环境可用（这是在 Openshift 上可以要求的一个标准特性）。为了实现这样的效果，你可以使用[DIY Cartridge](https://www.openshift.com/developers/do-it-yourself)，并在`.openshift/action_scripts`下 hooks 你的仓库：

基本模式如下：

1.确保 Java 和构建工具已被远程安装，比如使用一个`pre_build` hook（默认会安装 Java 和 Maven，不会安装 Gradle）。

2.使用一个`build` hook 去构建你的 jar（使用 Maven 或 Gradle），比如

```java
#!/bin/bash
cd $OPENSHIFT_REPO_DIR
mvn package -s .openshift/settings.xml -DskipTests=true 
```

3.添加一个调用`java -jar …`的`start` hook

```java
#!/bin/bash
cd $OPENSHIFT_REPO_DIR
nohup java -jar target/*.jar --server.port=${OPENSHIFT_DIY_PORT} --server.address=${OPENSHIFT_DIY_IP} & 
```

4.使用一个`stop` hook

```java
#!/bin/bash
source $OPENSHIFT_CARTRIDGE_SDK_BASH
PID=$(ps -ef | grep java.*\.jar | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    client_result "Application is already stopped"
else
    kill $PID
fi 
```

5.将内嵌的服务绑定到平台提供的在 application.properties 定义的环境变量，比如

```java
spring.datasource.url: jdbc:mysql://${OPENSHIFT_MYSQL_DB_HOST}:${OPENSHIFT_MYSQL_DB_PORT}/${OPENSHIFT_APP_NAME}
spring.datasource.username: ${OPENSHIFT_MYSQL_DB_USERNAME}
spring.datasource.password: ${OPENSHIFT_MYSQL_DB_PASSWORD} 
```

在 Openshift 的网站上有一篇[running Gradle in Openshift](https://www.openshift.com/blogs/run-gradle-builds-on-openshift)博客，如果想使用 gradle 构建运行的应用可以参考它。由于一个[Gradle bug](http://issues.gradle.org/browse/GRADLE-2871)，你不能使用高于 1.6 版本的 Gradle。

# 52\. Google App Engine

### 52\. Google App Engine

Google App Engine 跟 Servlet 2.5 API 是有联系的，所以在不修改的情况系你是不能部署一个 Spring 应用的。具体查看本指南的 Servlet 2.5 章节 Container.md)。

# 53\. 接下来阅读什么

### 53\. 接下来阅读什么