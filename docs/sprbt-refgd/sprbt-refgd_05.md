# V. Spring Boot 执行器: Production-ready 特性

### Spring Boot 执行器：Production-ready 特性

Spring Boot 包含很多其他的特性，它们可以帮你监控和管理发布到生产环境的应用。你可以选择使用 HTTP 端点，JMX 或远程 shell（SSH 或 Telnet）来管理和监控应用。审计（Auditing），健康（health）和数据采集（metrics gathering）会自动应用到你的应用。

# 39\. 开启 production-ready 特性

### 39\. 开启 production-ready 特性

[spring-boot-actuator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator)模块提供了 Spring Boot 所有的 production-ready 特性。启用该特性的最简单方式就是添加对 spring-boot-starter-actuator ‘Starter POM’的依赖。

**执行器（Actuator）的定义**：执行器是一个制造业术语，指的是用于移动或控制东西的一个机械装置。一个很小的改变就能让执行器产生大量的运动。

基于 Maven 的项目想要添加执行器只需添加下面的'starter'依赖：

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies> 
```

对于 Gradle，使用下面的声明：

```java
dependencies {
    compile("org.springframework.boot:spring-boot-starter-actuator")
} 
```

# 40\. 端点

### 40\. 端点

执行器端点允许你监控应用及与应用进行交互。Spring Boot 包含很多内置的端点，你也可以添加自己的。例如，health 端点提供了应用的基本健康信息。

端点暴露的方式取决于你采用的技术类型。大部分应用选择 HTTP 监控，端点的 ID 映射到一个 URL。例如，默认情况下，health 端点将被映射到/health。

下面的端点都是可用的：

| ID | 描述　 | 敏感（Sensitive） |
| --- | --- | --- |
| autoconfig | 显示一个 auto-configuration 的报告，该报告展示所有 auto-configuration 候选者及它们被应用或未被应用的原因 | true |
| beans | 显示一个应用中所有 Spring Beans 的完整列表 | true |
| configprops | 显示一个所有@ConfigurationProperties 的整理列表 | true |
| dump | 执行一个线程转储 | true |
| env | 暴露来自 Spring　ConfigurableEnvironment 的属性 | true |
| health | 展示应用的健康信息（当使用一个未认证连接访问时显示一个简单的'status'，使用认证连接访问则显示全部信息详情） | false |
| info | 显示任意的应用信息 | false |
| metrics | 展示当前应用的'指标'信息 | true |
| mappings | 显示一个所有@RequestMapping 路径的整理列表 | true |
| shutdown | 允许应用以优雅的方式关闭（默认情况下不启用） | true |
| trace | 显示 trace 信息（默认为最新的一些 HTTP 请求） | true |

**注**：根据一个端点暴露的方式，sensitive 参数可能会被用做一个安全提示。例如，在使用 HTTP 访问 sensitive 端点时需要提供用户名/密码（如果没有启用 web 安全，可能会简化为禁止访问该端点）。

# 40.1\. 自定义端点

### 40.1\. 自定义端点

使用 Spring 属性可以自定义端点。你可以设置端点是否开启（enabled），是否敏感（sensitive），甚至它的 id。例如，下面的 application.properties 改变了敏感性和 beans 端点的 id，也启用了 shutdown。

```java
endpoints.beans.id=springbeans
endpoints.beans.sensitive=false
endpoints.shutdown.enabled=true 
```

**注**：前缀`endpoints + . + name`被用来唯一的标识被配置的端点。

默认情况下，除了 shutdown 外的所有端点都是启用的。如果希望指定选择端点的启用，你可以使用 endpoints.enabled 属性。例如，下面的配置禁用了除 info 外的所有端点：

```java
endpoints.enabled=false
endpoints.info.enabled=true 
```

# 40.2\. 健康信息

### 40.2\. 健康信息

健康信息可以用来检查应用的运行状态。它经常被监控软件用来提醒人们生产系统是否停止。health 端点暴露的默认信息取决于端点是如何被访问的。对于一个非安全，未认证的连接只返回一个简单的'status'信息。对于一个安全或认证过的连接其他详细信息也会展示（具体参考 Section 41.6, “HTTP Health endpoint access restrictions” ）。

健康信息是从你的 ApplicationContext 中定义的所有[HealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) beans 收集过来的。Spring Boot 包含很多 auto-configured 的 HealthIndicators，你也可以写自己的。

# 40.3\. 安全与 HealthIndicators

### 40.3\. 安全与 HealthIndicators

HealthIndicators 返回的信息常常性质上有点敏感。例如，你可能不想将数据库服务器的详情发布到外面。因此，在使用一个未认证的 HTTP 连接时，默认只会暴露健康状态（health status）。如果想将所有的健康信息暴露出去，你可以把 endpoints.health.sensitive 设置为 false。

为防止'拒绝服务'攻击，Health 响应会被缓存。你可以使用`endpoints.health.time-to-live`属性改变默认的缓存时间（1000 毫秒）。

# 40.3.1\. 自动配置的 HealthIndicators

### 40.3.1\. 自动配置的 HealthIndicators

下面的 HealthIndicators 会被 Spring Boot 自动配置（在合适的时候）：

| 名称 | 描述 |
| --- | --- |
| [DiskSpaceHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/DiskSpaceHealthIndicator.java) | 低磁盘空间检测 |
| [DataSourceHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/DataSourceHealthIndicator.java) | 检查是否能从 DataSource 获取连接 |
| [MongoHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/MongoHealthIndicator.java) | 检查一个 Mongo 数据库是否可用（up） |
| [RabbitHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/RabbitHealthIndicator.java) | 检查一个 Rabbit 服务器是否可用（up） |
| [RedisHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/RedisHealthIndicator.java) | 检查一个 Redis 服务器是否可用（up） |
| [SolrHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/SolrHealthIndicator.java) | 检查一个 Solr 服务器是否可用（up） |

# 40.3.2\. 编写自定义 HealthIndicators

### 40.3.2\. 编写自定义 HealthIndicators

想提供自定义健康信息，你可以注册实现了[HealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java)接口的 Spring beans。你需要提供一个 health()方法的实现，并返回一个 Health 响应。Health 响应需要包含一个 status 和可选的用于展示的详情。

```java
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealth implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

} 
```

除了 Spring Boot 预定义的[Status](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java)类型，Health 也可以返回一个代表新的系统状态的自定义 Status。在这种情况下，需要提供一个[HealthAggregator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthAggregator.java)接口的自定义实现，或使用 management.health.status.order 属性配置默认的实现。

例如，假设一个新的，代码为 FATAL 的 Status 被用于你的一个 HealthIndicator 实现中。为了配置严重程度，你需要将下面的配置添加到 application 属性文件中：

```java
management.health.status.order: DOWN, OUT_OF_SERVICE, UNKNOWN, UP 
```

如果使用 HTTP 访问 health 端点，你可能想要注册自定义的 status，并使用 HealthMvcEndpoint 进行映射。例如，你可以将 FATAL 映射为 HttpStatus.SERVICE_UNAVAILABLE。

# 40.4\. 自定义应用 info 信息

### 40.4\. 自定义应用 info 信息

通过设置 Spring 属性 info.*，你可以定义 info 端点暴露的数据。所有在 info 关键字下的 Environment 属性都将被自动暴露。例如，你可以将下面的配置添加到 application.properties：

```java
info.app.name=MyService
info.app.description=My awesome service
info.app.version=1.0.0 
```

# 40.4.1\. 在构建时期自动扩展 info 属性

### 40.4.1\. 在构建时期自动扩展 info 属性

你可以使用已经存在的构建配置自动扩展 info 属性，而不是对在项目构建配置中存在的属性进行硬编码。这在 Maven 和 Gradle 都是可能的。

**使用 Maven 自动扩展属性**

对于 Maven 项目，你可以使用资源过滤来自动扩展 info 属性。如果使用 spring-boot-starter-parent，你可以通过`@..@`占位符引用 Maven 的'project properties'。

```java
project.artifactId=myproject
project.name=Demo
project.version=X.X.X.X
project.description=Demo project for info endpoint
info.build.artifact=@project.artifactId@
info.build.name=@project.name@
info.build.description=@project.description@
info.build.version=@project.version@ 
```

**注**：在上面的示例中，我们使用 project.*来设置一些值以防止由于某些原因 Maven 的资源过滤没有开启。Maven 目标`spring-boot:run`直接将`src/main/resources`添加到 classpath 下（出于热加载的目的）。这就绕过了资源过滤和自动扩展属性的特性。你可以使用`exec:java`替换该目标或自定义插件的配置，具体参考[plugin usage page](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/usage.html)。

如果你不使用 starter parent，在你的 pom.xml 你需要添加（处于<build class="hljs-pi">元素内）：</build>

```java
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources> 
```

和（处于<plugins class="hljs-pi">内）：</plugins>

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <delimiters>
            <delimiter>@</delimiter>
        </delimiters>
    </configuration>
</plugin> 
```

**使用 Gradle 自动扩展属性**

通过配置 Java 插件的 processResources 任务，你也可以自动使用来自 Gradle 项目的属性扩展 info 属性。

```java
processResources {
    expand(project.properties)
} 
```

然后你可以通过占位符引用 Gradle 项目的属性：

```java
info.build.name=${name}
info.build.description=${description}
info.build.version=${version} 
```

# 40.4.2\. Git 提交信息

### 40.4.2\. Git 提交信息

info 端点的另一个有用特性是，当项目构建完成后，它可以发布关于你的 git 源码仓库状态的信息。如果在你的 jar 中包含一个 git.properties 文件，git.branch 和 git.commit 属性将被加载。

对于 Maven 用户，`spring-boot-starter-parent` POM 包含一个能够产生 git.properties 文件的预配置插件。只需要简单的将下面的声明添加到你的 POM 中：

```java
<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>
        </plugin>
    </plugins>
</build> 
```

对于 Gradle 用户可以使用一个相似的插件[gradle-git](https://github.com/ajoberstar/gradle-git)，尽管为了产生属性文件可能需要稍微多点工作。

# 41\. 基于 HTTP 的监控和管理

### 41\. 基于 HTTP 的监控和管理

如果你正在开发一个 Spring MVC 应用，Spring Boot 执行器自动将所有启用的端点通过 HTTP 暴露出去。默认约定使用端点的 id 作为 URL 路径，例如，health 暴露为/health。

# 41.1\. 保护敏感端点

### 41.1\. 保护敏感端点

如果你的项目中添加的有 Spring Security，所有通过 HTTP 暴露的敏感端点都会受到保护。默认情况下会使用基本认证（basic authentication，用户名为 user，密码为应用启动时在控制台打印的密码）。

你可以使用 Spring 属性改变用户名，密码和访问端点需要的安全角色。例如，你可能会在 application.properties 中添加下列配置：

```java
security.user.name=admin
security.user.password=secret
management.security.role=SUPERUSER 
```

**注**：如果你不使用 Spring Security，那你的 HTTP 端点就被公开暴露，你应该慎重考虑启用哪些端点。具体参考 Section 40.1, “Customizing endpoints”。

# 41.2\. 自定义管理服务器的上下文路径

### 41.2\. 自定义管理服务器的上下文路径

有时候将所有的管理端口划分到一个路径下是有用的。例如，你的应用可能已经将`/info`作为他用。你可以用`management.contextPath`属性为管理端口设置一个前缀：

```java
management.context-path=/manage 
```

上面的 application.properties 示例将把端口从`/{id}`改为`/manage/{id}`（比如，/manage/info）。

# 41.3\. 自定义管理服务器的端口

### 41.3\. 自定义管理服务器的端口

对于基于云的部署，使用默认的 HTTP 端口暴露管理端点（endpoints）是明智的选择。然而，如果你的应用是在自己的数据中心运行，那你可能倾向于使用一个不同的 HTTP 端口来暴露端点。

`management.port`属性可以用来改变 HTTP 端口：

```java
management.port=8081 
```

由于你的管理端口经常被防火墙保护，不对外暴露也就不需要保护管理端点，即使你的主要应用是安全的。在这种情况下，classpath 下会存在 Spring Security 库，你可以设置下面的属性来禁用安全管理策略（management security）：

```java
management.security.enabled=false 
```

（如果 classpath 下不存在 Spring Security，那也就不需要显示的以这种方式来禁用安全管理策略，它甚至可能会破坏应用程序。）

# 41.4\. 自定义管理服务器的地址

### 41.4\. 自定义管理服务器的地址

你可以通过设置`management.address`属性来定义管理端点可以使用的地址。这在你只想监听内部或面向生产环境的网络，或只监听来自 localhost 的连接时非常有用。

下面的 application.properties 示例不允许远程管理连接：

```java
management.port=8081
management.address=127.0.0.1 
```

# 41.5\. 禁用 HTTP 端点

### 41.5\. 禁用 HTTP 端点

如果不想通过 HTTP 暴露端点，你可以将管理端口设置为-1： `management.port=-1`

# 41.6\. HTTP Health 端点访问限制

### 41.6\. HTTP Health 端点访问限制

通过 health 端点暴露的信息根据是否为匿名访问而不同。默认情况下，当匿名访问时，任何有关服务器的健康详情都被隐藏了，该端点只简单的指示服务器是运行（up）还是停止（down）。此外，当匿名访问时，响应会被缓存一个可配置的时间段以防止端点被用于'拒绝服务'攻击。`endpoints.health.time-to-live`属性被用来配置缓存时间（单位为毫秒），默认为 1000 毫秒，也就是 1 秒。

上述的限制可以被禁止，从而允许匿名用户完全访问 health 端点。想达到这个效果，可以将`endpoints.health.sensitive`设为`false`。

# 42\. 基于 JMX 的监控和管理

### 42\. 基于 JMX 的监控和管理

Java 管理扩展（JMX）提供了一种标准的监控和管理应用的机制。默认情况下，Spring Boot 在`org.springframework.boot`域下将管理端点暴露为 JMX MBeans。

# 42.1\. 自定义 MBean 名称

### 42.1\. 自定义 MBean 名称

MBean 的名称通常产生于端点的 id。例如，health 端点被暴露为`org.springframework.boot/Endpoint/HealthEndpoint`。

如果你的应用包含多个 Spring ApplicationContext，你会发现存在名称冲突。为了解决这个问题，你可以将`endpoints.jmx.uniqueNames`设置为 true，这样 MBean 的名称总是唯一的。

你也可以自定义 JMX 域，所有的端点都在该域下暴露。这里有个 application.properties 示例： ```java endpoints.jmx.domain=myapp endpoints.jmx.uniqueNames=true

# 42.2\. 禁用 JMX 端点

### 42.2\. 禁用 JMX 端点

如果不想通过 JMX 暴露端点，你可以将`spring.jmx.enabled`属性设置为 false：

```
spring.jmx.enabled=false 
```java

# 42.3\. 使用 Jolokia 通过 HTTP 实现 JMX 远程管理

### 42.3\. 使用 Jolokia 通过 HTTP 实现 JMX 远程管理

Jolokia 是一个 JMX-HTTP 桥，它提供了一种访问 JMX beans 的替代方法。想要使用 Jolokia，只需添加`org.jolokia:jolokia-core`的依赖。例如，使用 Maven 需要添加下面的配置：

```
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
 </dependency> 
```java

在你的管理 HTTP 服务器上可以通过`/jolokia`访问 Jolokia。

# 42.3.1\. 自定义 Jolokia

### 42.3.1\. 自定义 Jolokia

Jolokia 有很多配置，传统上一般使用 servlet 参数进行设置。使用 Spring Boot，你可以在 application.properties 中通过把参数加上`jolokia.config.`前缀来设置：

```
jolokia.config.debug=true 
```java

# 42.3.2\. 禁用 Jolokia

### 42.3.2\. 禁用 Jolokia

如果你正在使用 Jolokia，但不想让 Spring Boot 配置它，只需要简单的将`endpoints.jolokia.enabled`属性设置为 false：

```
endpoints.jolokia.enabled=false 
```java

# 43\. 使用远程 shell 来进行监控和管理

### 43\. 使用远程 shell 来进行监控和管理

Spring Boot 支持集成一个称为'CRaSH'的 Java shell。你可以在 CRaSH 中使用 ssh 或 telnet 命令连接到运行的应用。为了启用远程 shell 支持，你只需添加`spring-boot-starter-remote-shell`的依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-remote-shell</artifactId>
 </dependency> 
```java

**注**：如果想使用 telnet 访问，你还需添加对`org.crsh:crsh.shell.telnet`的依赖。

# 43.1\. 连接远程 shell

### 43.1\. 连接远程 shell

默认情况下，远程 shell 监听端口 2000 以等待连接。默认用户名为`user`，密码为随机生成的，并且在输出日志中会显示。如果应用使用 Spring Security，该 shell 默认使用相同的配置。如果不是，将使用一个简单的认证策略，你可能会看到类似这样的信息：

```
Using default password for shell access: ec03e16c-4cf4-49ee-b745-7c8255c1dd7e 
```java

Linux 和 OSX 用户可以使用`ssh`连接远程 shell，Windows 用户可以下载并安装[PuTTY](http://www.putty.org/)。

```
$ ssh -p 2000 user@localhost

user@localhost's password:
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.3.0.BUILD-SNAPSHOT) on myhost 
```java

输入 help 可以获取一系列命令的帮助。Spring boot 提供`metrics`，`beans`，`autoconfig`和`endpoint`命令。

# 43.1.1\. 远程 shell 证书

### 43.1.1\. 远程 shell 证书

你可以使用`shell.auth.simple.user.name`和`shell.auth.simple.user.password`属性配置自定义的连接证书。也可以使用 Spring Security 的 AuthenticationManager 处理登录职责。具体参考 Javadoc[CrshAutoConfiguration](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/actuate/autoconfigure/CrshAutoConfiguration.html)和[ShellProperties](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/actuate/autoconfigure/ShellProperties.html)。

# 43.2\. 扩展远程 shell

### 43.2\. 扩展远程 shell

有很多有趣的方式可以用来扩展远程 shell。

# 43.2.1\. 远程 shell 命令

### 43.2.1\. 远程 shell 命令

你可以使用 Groovy 或 Java 编写其他的 shell 命令（具体参考 CRaSH 文档）。默认情况下，Spring Boot 会搜索以下路径的命令：

*   `classpath*:/commands/**`
*   `classpath*:/crash/commands/**`

**注**：可以通过`shell.commandPathPatterns`属性改变搜索路径。

下面是一个从`src/main/resources/commands/hello.groovy`加载的'hello world'命令：

```
package commands

import org.crsh.cli.Usage
import org.crsh.cli.Command

class hello {

    @Usage("Say Hello")
    @Command
    def main(InvocationContext context) {
        return "Hello"
    }

} 
```java

Spring Boot 将一些额外属性添加到了 InvocationContext，你可以在命令中访问它们：

| 属性名称 | 描述 |
| --- | --- |
| spring.boot.version | Spring Boot 的版本 |
| spring.version | Spring 框架的核心版本 |
| spring.beanfactory | 获取 Spring 的 BeanFactory |
| spring.environment | 获取 Spring 的 Environment |

# 43.2.2\. 远程 shell 插件

### 43.2.2\. 远程 shell 插件

除了创建新命令，也可以扩展 CRaSH shell 的其他特性。所有继承`org.crsh.plugin.CRaSHPlugin`的 Spring Beans 将自动注册到 shell。

具体查看[CRaSH 参考文档](http://www.crashub.org/)。

# 44\. 度量指标（Metrics）

### 44\. 度量指标（Metrics）

Spring Boot 执行器包括一个支持'gauge'和'counter'级别的度量指标服务。'gauge'记录一个单一值；'counter'记录一个增量（增加或减少）。同时，Spring Boot 提供一个[PublicMetrics](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/endpoint/PublicMetrics.java)接口，你可以实现它，从而暴露以上两种机制不能记录的指标。具体参考[SystemPublicMetrics](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/endpoint/SystemPublicMetrics.java)。

所有 HTTP 请求的指标都被自动记录，所以如果点击`metrics`端点，你可能会看到类似以下的响应：

```
{
    "counter.status.200.root": 20,
    "counter.status.200.metrics": 3,
    "counter.status.200.star-star": 5,
    "counter.status.401.root": 4,
    "gauge.response.star-star": 6,
    "gauge.response.root": 2,
    "gauge.response.metrics": 3,
    "classes": 5808,
    "classes.loaded": 5808,
    "classes.unloaded": 0,
    "heap": 3728384,
    "heap.committed": 986624,
    "heap.init": 262144,
    "heap.used": 52765,
    "mem": 986624,
    "mem.free": 933858,
    "processors": 8,
    "threads": 15,
    "threads.daemon": 11,
    "threads.peak": 15,
    "uptime": 494836,
    "instance.uptime": 489782,
    "datasource.primary.active": 5,
    "datasource.primary.usage": 0.25
} 
```java

此处我们可以看到基本的`memory`，`heap`，`class loading`，`processor`和`thread pool`信息，连同一些 HTTP 指标。在该实例中，`root`('/')，`/metrics` URLs 分别返回 20 次，3 次`HTTP 200`响应。同时可以看到`root` URL 返回了 4 次`HTTP 401`（unauthorized）响应。双 asterix（star-star）来自于被 Spring MVC `/**`匹配到的一个请求（通常为一个静态资源）。

`gauge`级别展示了一个请求的最后响应时间。所以，`root`的最后请求被响应耗时 2 毫秒，`/metrics`耗时 3 毫秒。

# 44.1\. 系统指标

### 44.1\. 系统指标

Spring Boot 暴露以下系统指标：

*   系统内存总量（mem），单位:Kb
*   空闲内存数量（mem.free），单位:Kb
*   处理器数量（processors）
*   系统正常运行时间（uptime），单位:毫秒
*   应用上下文（就是一个应用实例）正常运行时间（instance.uptime），单位:毫秒
*   系统平均负载（systemload.average）
*   堆信息（heap，heap.committed，heap.init，heap.used），单位:Kb
*   线程信息（threads，thread.peak，thead.daemon）
*   类加载信息（classes，classes.loaded，classes.unloaded）
*   垃圾收集信息（gc.xxx.count, gc.xxx.time）

# 44.2\. 数据源指标

### 44.2\. 数据源指标

Spring Boot 会为你应用中定义的支持的 DataSource 暴露以下指标：

*   最大连接数（datasource.xxx.max）
*   最小连接数（datasource.xxx.min）
*   活动连接数（datasource.xxx.active）
*   连接池的使用情况（datasource.xxx.usage）

所有的数据源指标共用`datasoure.`前缀。该前缀对每个数据源都非常合适：

*   如果是主数据源（唯一可用的数据源或存在的数据源中被@Primary 标记的）前缀为 datasource.primary
*   如果数据源 bean 名称以 dataSource 结尾，那前缀就是 bean 的名称去掉 dataSource 的部分（例如，batchDataSource 的前缀是 datasource.batch）
*   其他情况使用 bean 的名称作为前缀

通过注册一个自定义版本的 DataSourcePublicMetrics bean，你可以覆盖部分或全部的默认行为。默认情况下，Spring Boot 提供支持所有数据源的元数据；如果你喜欢的数据源恰好不被支持，你可以添加另外的 DataSourcePoolMetadataProvider beans。具体参考 DataSourcePoolMetadataProvidersConfiguration。

# 44.3\. Tomcat session 指标

### 44.3\. Tomcat session 指标

如果你使用 Tomcat 作为内嵌的 servlet 容器，session 指标将被自动暴露出去。 `httpsessions.active`和`httpsessions.max`提供了活动的和最大的 session 数量。

# 44.4\. 记录自己的指标

### 44.4\. 记录自己的指标

想要记录你自己的指标，只需将[CounterService](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/CounterService.java)或[GaugeService](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/GaugeService.java)注入到你的 bean 中。CounterService 暴露 increment，decrement 和 reset 方法；GaugeService 提供一个 submit 方法。

下面是一个简单的示例，它记录了方法调用的次数：

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.metrics.CounterService;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final CounterService counterService;

    @Autowired
    public MyService(CounterService counterService) {
        this.counterService = counterService;
    }

    public void exampleMethod() {
        this.counterService.increment("services.system.myservice.invoked");
    }

} 
```java

**注**：你可以将任何的字符串用作指标的名称，但最好遵循所选存储或图技术的指南。[Matt Aimonetti’s Blog](http://matt.aimonetti.net/posts/2013/06/26/practical-guide-to-graphite-monitoring/)中有一些好的关于图（Graphite）的指南。

# 44.5\. 添加你自己的公共指标

### 44.5\. 添加你自己的公共指标

想要添加额外的，每次指标端点被调用时都会重新计算的度量指标，只需简单的注册其他的 PublicMetrics 实现 bean(s)。默认情况下，端点会聚合所有这样的 beans，通过定义自己的 MetricsEndpoint 可以轻易改变这种情况。

# 44.6\. 指标仓库

### 44.6\. 指标仓库

通过绑定一个[MetricRepository](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/repository/MetricRepository.java)来实现指标服务。`MetricRepository`负责存储和追溯指标信息。Spring Boot 提供一个`InMemoryMetricRepository`和一个`RedisMetricRepository`（默认使用 in-memory 仓库），不过你可以编写自己的`MetricRepository`。`MetricRepository`接口实际是`MetricReader`接口和`MetricWriter`接口的上层组合。具体参考[Javadoc](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/api/org/springframework/boot/actuate/metrics/repository/MetricRepository.html)

没有什么能阻止你直接将`MetricRepository`的数据导入应用中的后端存储，但我们建议你使用默认的`InMemoryMetricRepository`（如果担心堆使用情况，你可以使用自定义的 Map 实例），然后通过一个 scheduled export job 填充后端仓库（意思是先将数据保存到内存中，然后通过异步 job 将数据持久化到数据库，可以提高系统性能）。通过这种方式，你可以将指标数据缓存到内存中，然后通过低频率或批量导出来减少网络拥堵。Spring Boot 提供一个`Exporter`接口及一些帮你开始的基本实现。

# 44.7\. Dropwizard 指标

### 44.7\. Dropwizard 指标

[Dropwizard ‘Metrics’库](https://dropwizard.github.io/metrics/)的用户会发现 Spring Boot 指标被发布到了`com.codahale.metrics.MetricRegistry`。当你声明对`io.dropwizard.metrics:metrics-core`库的依赖时会创建一个默认的`com.codahale.metrics.MetricRegistry` Spring bean；如果需要自定义，你可以注册自己的@Bean 实例。来自于`MetricRegistry`的指标也是自动通过`/metrics`端点暴露的。

用户可以通过使用合适类型的指标名称作为前缀来创建 Dropwizard 指标（比如，`histogram.*`, `meter.*`）。

# 44.8\. 消息渠道集成

### 44.8\. 消息渠道集成

如果你的 classpath 下存在'Spring Messaging' jar，一个名为`metricsChannel`的`MessageChannel`将被自动创建（除非已经存在一个）。此外，所有的指标更新事件作为'messages'发布到该渠道上。订阅该渠道的客户端可以进行额外的分析或行动。

# 45\. 审计

### 45\. 审计

Spring Boot 执行器具有一个灵活的审计框架，一旦 Spring Security 处于活动状态（默认抛出'authentication success'，'failure'和'access denied'异常），它就会发布事件。这对于报告非常有用，同时可以基于认证失败实现一个锁定策略。

你也可以使用审计服务处理自己的业务事件。为此，你可以将存在的`AuditEventRepository`注入到自己的组件，并直接使用它，或者只是简单地通过 Spring `ApplicationEventPublisher`发布`AuditApplicationEvent`（使用`ApplicationEventPublisherAware`）。

# 46\. 追踪（Tracing）

### 46\. 追踪（Tracing）

对于所有的 HTTP 请求 Spring Boot 自动启用追踪。你可以查看`trace`端点，并获取最近一些请求的基本信息：

```
[{
    "timestamp": 1394343677415,
    "info": {
        "method": "GET",
        "path": "/trace",
        "headers": {
            "request": {
                "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
                "Connection": "keep-alive",
                "Accept-Encoding": "gzip, deflate",
                "User-Agent": "Mozilla/5.0 Gecko/Firefox",
                "Accept-Language": "en-US,en;q=0.5",
                "Cookie": "_ga=GA1.1.827067509.1390890128; ..."
                "Authorization": "Basic ...",
                "Host": "localhost:8080"
            },
            "response": {
                "Strict-Transport-Security": "max-age=31536000 ; includeSubDomains",
                "X-Application-Context": "application:8080",
                "Content-Type": "application/json;charset=UTF-8",
                "status": "200"
            }
        }
    }
},{
    "timestamp": 1394343684465,
    ...
}] 
```java

# 46.1\. 自定义追踪

### 46.1\. 自定义追踪

如果需要追踪其他的事件，你可以将一个[TraceRepository](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/trace/TraceRepository.java)注入到你的 Spring Beans 中。`add`方法接收一个将被转化为 JSON 的`Map`结构，该数据将被记录下来。

默认情况下，使用的`InMemoryTraceRepository`将存储最新的 100 个事件。如果需要扩展该容量，你可以定义自己的`InMemoryTraceRepository`实例。如果需要，你可以创建自己的替代`TraceRepository`实现。

# 47\. 进程监控

### 47\. 进程监控

在 Spring Boot 执行器中，你可以找到几个创建有利于进程监控的文件的类：

*   `ApplicationPidFileWriter`创建一个包含应用 PID 的文件（默认位于应用目录，文件名为 application.pid）
*   `EmbeddedServerPortFileWriter`创建一个或多个包含内嵌服务器端口的文件（默认位于应用目录，文件名为 application.port）

默认情况下，这些 writers 没有被激活，但你可以使用下面描述的任何方式来启用它们。

# 47.1\. 扩展属性

### 47.1\. 扩展属性

你需要激活`META-INF/spring.factories`文件里的 listener(s)：

```
org.springframework.context.ApplicationListener=\
org.springframework.boot.actuate.system.ApplicationPidFileWriter,
org.springframework.boot.actuate.system.EmbeddedServerPortFileWriter 
```

# 47.2\. 以编程方式

### 47.2\. 以编程方式

你也可以通过调用`SpringApplication.addListeners(…)`方法来激活一个监听器，并传递相应的`Writer`对象。该方法允许你通过`Writer`构造器自定义文件名和路径。