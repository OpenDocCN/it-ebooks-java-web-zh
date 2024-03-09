# IX. How-to 指南

### How-to 指南

本章节将回答一些常见的"我该怎么做"类型的问题，这些问题在我们使用 Spring Boot 时经常遇到。这绝不是一个详尽的列表，但它覆盖了很多方面。

如果遇到一个特殊的我们没有覆盖的问题，你可能想去查看[stackoverflow.com](http://stackoverflow.com/tags/spring-boot)，看是否有人已经给出了答案;这也是一个很好的提新问题的地方（请使用`spring-boot`标签）。

我们也乐意扩展本章节;如果想添加一个'how-to'，你可以给我们发一个[pull 请求](http://github.com/spring-projects/spring-boot/tree/master)。

# 62\. Spring Boot 应用

### 62\. Spring Boot 应用

# 62.1\. 解决自动配置问题

### 62.1\. 解决自动配置问题

Spring Boot 自动配置总是尝试尽最大努力去做正确的事，但有时候会失败并且很难说出失败原因。

在每个 Spring Boot ApplicationContext 中都存在一个相当有用的 ConditionEvaluationReport。如果开启`DEBUG`日志输出，你将会看到它。如果你使用`spring-boot-actuator`，则会有一个 autoconfig 的端点，它将以 JSON 形式渲染该报告。可以使用它调试应用程序，并能查看 Spring Boot 运行时都添加了哪些特性（及哪些没添加）。

通过查看源码和 javadoc 可以获取更多问题的答案。以下是一些经验：

*   查找名为`*AutoConfiguration`的类并阅读源码，特别是`@Conditional*`注解，这可以帮你找出它们启用哪些特性及何时启用。 将`--debug`添加到命令行或添加系统属性`-Ddebug`可以在控制台查看日志，该日志会记录你的应用中所有自动配置的决策。在一个运行的 Actuator app 中，通过查看 autoconfig 端点（`/autoconfig`或等效的 JMX）可以获取相同信息。
*   查找是`@ConfigurationProperties`的类（比如[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)）并看下有哪些可用的外部配置选项。`@ConfigurationProperties`类有一个用于充当外部配置前缀的 name 属性，因此`ServerProperties`的值为`prefix="server"`，它的配置属性有`server.port`，`server.address`等。在运行的 Actuator 应用中可以查看 configprops 端点。
*   查看使用 RelaxedEnvironment 明确地将配置从 Environment 暴露出去。它经常会使用一个前缀。
*   查看`@Value`注解，它直接绑定到 Environment。相比 RelaxedEnvironment，这种方式稍微缺乏灵活性，但它也允许松散的绑定，特别是 OS 环境变量（所以`CAPITALS_AND_UNDERSCORES`是`period.separated`的同义词）。
*   查看`@ConditionalOnExpression`注解，它根据 SpEL 表达式的结果来开启或关闭特性，通常使用解析自 Environment 的占位符进行计算。

# 62.2\. 启动前自定义 Environment 或 ApplicationContext

### 62.2\. 启动前自定义 Environment 或 ApplicationContext

每个 SpringApplication 都有 ApplicationListeners 和 ApplicationContextInitializers，用于自定义上下文（context）或环境(environment)。Spring Boot 从`META-INF/spring.factories`下加载很多这样的内部使用的自定义。有很多方法可以注册其他的自定义：

*   以编程方式为每个应用注册自定义，通过在 SpringApplication 运行前调用它的`addListeners`和`addInitializers`方法来实现。
*   以声明方式为每个应用注册自定义，通过设置`context.initializer.classes`或`context.listener.classes`来实现。
*   以声明方式为所有应用注册自定义，通过添加一个`META-INF/spring.factories`并打包成一个 jar 文件（该应用将它作为一个库）来实现。

SpringApplication 会给监听器（即使是在上下文被创建之前就存在的）发送一些特定的 ApplicationEvents，然后也会注册监听 ApplicationContext 发布的事件的监听器。查看 Spring Boot 特性章节中的 Section 22.4, “Application events and listeners” 可以获取一个完整列表。

# 62.4\. 创建一个非 web（non-web）应用

### 62.4\. 创建一个非 web（non-web）应用

不是所有的 Spring 应用都必须是 web 应用（或 web 服务）。如果你想在 main 方法中执行一些代码，但需要启动一个 Spring 应用去设置需要的底层设施，那使用 Spring Boot 的`SpringApplication`特性可以很容易实现。`SpringApplication`会根据它是否需要一个 web 应用来改变它的`ApplicationContext`类。首先你需要做的是去掉 servlet API 依赖，如果不能这样做（比如，基于相同的代码运行两个应用），那你可以明确地调用`SpringApplication.setWebEnvironment(false)`或设置`applicationContextClass`属性（通过 Java API 或使用外部配置）。你想运行的，作为业务逻辑的应用代码可以实现为一个`CommandLineRunner`，并将上下文降级为一个`@Bean`定义。

# 63\. 属性&配置

### 63\. 属性&配置

# 63.1\. 外部化 SpringApplication 配置

### 63.1\. 外部化 SpringApplication 配置

SpringApplication 已经被属性化（主要是 setters），所以你可以在创建应用时使用它的 Java API 修改它的行为。或者你可以使用 properties 文件中的`spring.main.*`来外部化（在应用代码外配置）这些配置。比如，在`application.properties`中可能会有以下内容：

```java
spring.main.web_environment=false
spring.main.show_banner=false 
```

然后 Spring Boot 在启动时将不会显示 banner，并且该应用也不是一个 web 应用。

# 63.2\. 改变应用程序外部配置文件的位置

### 63.2\. 改变应用程序外部配置文件的位置

默认情况下，来自不同源的属性以一个定义好的顺序添加到 Spring 的`Environment`中（查看'Sprin Boot 特性'章节的[Chapter 23, Externalized Configuration](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config)获取精确的顺序）。

为应用程序源添加`@PropertySource`注解是一种很好的添加和修改源顺序的方法。传递给`SpringApplication`静态便利设施（convenience）方法的类和使用`setSources()`添加的类都会被检查，以查看它们是否有`@PropertySources`，如果有，这些属性会被尽可能早的添加到`Environment`里，以确保`ApplicationContext`生命周期的所有阶段都能使用。以这种方式添加的属性优先于任何使用默认位置添加的属性，但低于系统属性，环境变量或命令行参数。

你也可以提供系统属性（或环境变量）来改变该行为：

*   `spring.config.name`（`SPRING_CONFIG_NAME`）是根文件名，默认为`application`。
*   `spring.config.location`（`SPRING_CONFIG_LOCATION`）是要加载的文件（例如，一个 classpath 资源或一个 URL）。Spring Boot 为该文档设置一个单独的`Environment`属性，它可以被系统属性，环境变量或命令行参数覆盖。

不管你在 environment 设置什么，Spring Boot 都将加载上面讨论过的`application.properties`。如果使用 YAML，那具有'.yml'扩展的文件默认也会被添加到该列表。

详情参考[ConfigFileApplicationListener](http://github.com/spring-projects/spring-boot/tree/master/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java)

# 63.3\. 使用'short'命令行参数

### 63.3\. 使用'short'命令行参数

有些人喜欢使用（例如）`--port=9000`代替`--server.port=9000`来设置命令行配置属性。你可以通过在 application.properties 中使用占位符来启用该功能，比如：

```java
server.port=${port:8080} 
```

**注**：如果你继承自`spring-boot-starter-parent` POM，为了防止和 Spring-style 的占位符产生冲突，`maven-resources-plugins`默认的过滤令牌（filter token）已经从`${*}`变为`@`（即`@maven.token@`代替了`${maven.token}`）。如果已经直接启用 maven 对 application.properties 的过滤，你可能也想使用[其他的分隔符](http://maven.apache.org/plugins/maven-resources-plugin/resources-mojo.html#delimiters)替换默认的过滤令牌。

**注**：在这种特殊的情况下，端口绑定能够在一个 PaaS 环境下工作，比如 Heroku 和 Cloud Foundry，因为在这两个平台中`PORT`环境变量是自动设置的，并且 Spring 能够绑定`Environment`属性的大写同义词。

# 63.4\. 使用 YAML 配置外部属性

### 63.4\. 使用 YAML 配置外部属性

YAML 是 JSON 的一个超集，可以非常方便的将外部配置以层次结构形式存储起来。比如：

```java
spring:
    application:
        name: cruncher
    datasource:
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost/test
server:
    port: 9000 
```

创建一个 application.yml 文件，将它放到 classpath 的根目录下，并添加 snakeyaml 依赖（Maven 坐标为`org.yaml:snakeyaml`，如果你使用`spring-boot-starter`那就已经被包含了）。一个 YAML 文件会被解析为一个 Java `Map<String,Object>`（和一个 JSON 对象类似），Spring Boot 会平伸该 map，这样它就只有 1 级深度，并且有 period-separated 的 keys，跟人们在 Java 中经常使用的 Properties 文件非常类似。 上面的 YAML 示例对应于下面的 application.properties 文件：

```java
spring.application.name=cruncher
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost/test
server.port=9000 
```

查看'Spring Boot 特性'章节的[Section 23.6, “Using YAML instead of Properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-yaml)可以获取更多关于 YAML 的信息。

# 63.5\. 设置生效的 Spring profiles

### 63.5\. 设置生效的 Spring profiles

Spring `Environment`有一个 API 可以设置生效的 profiles，但通常你会设置一个系统 profile（`spring.profiles.active`）或一个 OS 环境变量（`SPRING_PROFILES_ACTIVE`）。比如，使用一个`-D`参数启动应用程序（记着把它放到 main 类或 jar 文件之前）：

```java
$ java -jar -Dspring.profiles.active=production demo-0.0.1-SNAPSHOT.jar 
```

在 Spring Boot 中，你也可以在 application.properties 里设置生效的 profile，例如：

```java
spring.profiles.active=production 
```

通过这种方式设置的值会被系统属性或环境变量替换，但不会被`SpringApplicationBuilder.profiles()`方法替换。因此，后面的 Java API 可用来在不改变默认设置的情况下增加 profiles。

想要获取更多信息可查看'Spring Boot 特性'章节的[Chapter 24, Profiles](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-profiles)。

# 63.6\. 根据环境改变配置

### 63.6\. 根据环境改变配置

一个 YAML 文件实际上是一系列以`---`线分割的文档，每个文档都被单独解析为一个平坦的（flattened）map。

如果一个 YAML 文档包含一个`spring.profiles`关键字，那 profiles 的值（以逗号分割的 profiles 列表）将被传入 Spring 的`Environment.acceptsProfiles()`方法，并且如果这些 profiles 的任何一个被激活，对应的文档被包含到最终的合并中（否则不会）。

示例：

```java
server:
    port: 9000
---

spring:
    profiles: development
server:
    port: 9001

---

spring:
    profiles: production
server:
    port: 0 
```

在这个示例中，默认的端口是 9000，但如果 Spring profile 'development'生效则该端口是 9001，如果'production'生效则它是 0。

YAML 文档以它们遇到的顺序合并（所以后面的值会覆盖前面的值）。

想要使用 profiles 文件完成同样的操作，你可以使用`application-${profile}.properties`指定特殊的，profile 相关的值。

# 63.7\. 发现外部属性的内置选项

### 63.7\. 发现外部属性的内置选项

Spring Boot 在运行时将来自 application.properties（或.yml）的外部属性绑定进一个应用中。在一个地方不可能存在详尽的所有支持属性的列表（技术上也是不可能的），因为你的 classpath 下的其他 jar 文件也能够贡献。

每个运行中且有 Actuator 特性的应用都会有一个`configprops`端点，它能够展示所有边界和可通过`@ConfigurationProperties`绑定的属性。

附录中包含一个[application.properties](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#common-application-properties)示例，它列举了 Spring Boot 支持的大多数常用属性。获取权威列表可搜索`@ConfigurationProperties`和`@Value`的源码，还有不经常使用的`RelaxedEnvironment`。

# 64\. 内嵌的 servlet 容器

### 64\. 内嵌的 servlet 容器

# 64.1\. 为应用添加 Servlet，Filter 或 ServletContextListener

### 64.1\. 为应用添加 Servlet，Filter 或 ServletContextListener

Servlet 规范支持的 Servlet，Filter，ServletContextListener 和其他监听器可以作为`@Bean`定义添加到你的应用中。需要格外小心的是，它们不会引起太多的其他 beans 的热初始化，因为在应用生命周期的早期它们已经被安装到容器里了（比如，让它们依赖你的 DataSource 或 JPA 配置就不是一个好主意）。你可以通过延迟初始化它们到第一次使用而不是初始化时来突破该限制。

在 Filters 和 Servlets 的情况下，你也可以通过添加一个`FilterRegistrationBean`或`ServletRegistrationBean`代替或以及底层的组件来添加映射（mappings）和初始化参数。

# 64.2\. 改变 HTTP 端口

### 64.2\. 改变 HTTP 端口

在一个单独的应用中，主 HTTP 端口默认为 8080，但可以使用`server.port`设置（比如，在 application.properties 中或作为一个系统属性）。由于`Environment`值的宽松绑定，你也可以使用`SERVER_PORT`（比如，作为一个 OS 环境变）。

为了完全关闭 HTTP 端点，但仍创建一个 WebApplicationContext，你可以设置`server.port=-1`（测试时可能有用）。

想获取更多详情可查看'Spring Boot 特性'章节的[Section 26.3.3, “Customizing embedded servlet containers”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-customizing-embedded-containers)，或[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)源码。

# 64.3\. 使用随机未分配的 HTTP 端口

### 64.3\. 使用随机未分配的 HTTP 端口

想扫描一个未使用的端口（为了防止冲突使用 OS 本地端口）可以使用`server.port=0`。

# 64.4\. 发现运行时的 HTTP 端口

### 64.4\. 发现运行时的 HTTP 端口

你可以通过日志输出或它的 EmbeddedServletContainer 的 EmbeddedWebApplicationContext 获取服务器正在运行的端口。获取和确认服务器已经初始化的最好方式是添加一个`ApplicationListener<EmbeddedServletContainerInitializedEvent>`类型的`@Bean`，然后当事件发布时将容器 pull 出来。

使用`@WebIntegrationTests`的一个有用实践是设置`server.port=0`，然后使用`@Value`注入实际的（'local'）端口。例如：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = SampleDataJpaApplication.class)
@WebIntegrationTest("server.port:0")
public class CityRepositoryIntegrationTests {

    @Autowired
    EmbeddedWebApplicationContext server;

    @Value("${local.server.port}")
    int port;

    // ...

} 
```

# 64.5\. 配置 SSL

### 64.5\. 配置 SSL

SSL 能够以声明方式进行配置，一般通过在 application.properties 或 application.yml 设置各种各样的`server.ssl.*`属性。例如：

```java
server.port = 8443
server.ssl.key-store = classpath:keystore.jks
server.ssl.key-store-password = secret
server.ssl.key-password = another-secret 
```

获取所有支持的配置详情可查看[Ssl](http://github.com/spring-projects/spring-boot/tree/master/spring-boot/src/main/java/org/springframework/boot/context/embedded/Ssl.java)。

**注**：Tomcat 要求 key 存储（如果你正在使用一个可信存储）能够直接在文件系统上访问，即它不能从一个 jar 文件内读取。Jetty 和 Undertow 没有该限制。

使用类似于以上示例的配置意味着该应用将不在支持端口为 8080 的普通 HTTP 连接。Spring Boot 不支持通过 application.properties 同时配置 HTTP 连接器和 HTTPS 连接器。如果你两个都想要，那就需要以编程的方式配置它们中的一个。推荐使用 application.properties 配置 HTTPS，因为 HTTP 连接器是两个中最容易以编程方式进行配置的。获取示例可查看[spring-boot-sample-tomcat-multi-connectors](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors)示例项目。

# 64.6\. 配置 Tomcat

### 64.6\. 配置 Tomcat

通常你可以遵循[Section 63.7, “Discover built-in options for external properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-discover-build-in-options-for-external-properties)关于`@ConfigurationProperties`（这里主要的是`ServerProperties`）的建议，但也看下`EmbeddedServletContainerCustomizer`和各种你可以添加的 Tomcat-specific 的`*Customizers`。

Tomcat APIs 相当丰富，一旦获取到`TomcatEmbeddedServletContainerFactory`，你就能够以多种方式修改它。或核心选择是添加你自己的`TomcatEmbeddedServletContainerFactory`。

# 64.7\. 启用 Tomcat 的多连接器（Multiple Connectors）

### 64.7\. 启用 Tomcat 的多连接器（Multiple Connectors）

你可以将一个`org.apache.catalina.connector.Connector`添加到`TomcatEmbeddedServletContainerFactory`，这就能够允许多连接器，比如 HTTP 和 HTTPS 连接器：

```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
    tomcat.addAdditionalTomcatConnectors(createSslConnector());
    return tomcat;
}

private Connector createSslConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
    try {
        File keystore = new ClassPathResource("keystore").getFile();
        File truststore = new ClassPathResource("keystore").getFile();
        connector.setScheme("https");
        connector.setSecure(true);
        connector.setPort(8443);
        protocol.setSSLEnabled(true);
        protocol.setKeystoreFile(keystore.getAbsolutePath());
        protocol.setKeystorePass("changeit");
        protocol.setTruststoreFile(truststore.getAbsolutePath());
        protocol.setTruststorePass("changeit");
        protocol.setKeyAlias("apitester");
        return connector;
    }
    catch (IOException ex) {
        throw new IllegalStateException("can't access keystore: [" + "keystore"
                + "] or truststore: [" + "keystore" + "]", ex);
    }
} 
```

# 64.8\. 在前端代理服务器后使用 Tomcat

### 64.8\. 在前端代理服务器后使用 Tomcat

Spring Boot 将自动配置 Tomcat 的`RemoteIpValve`，如果你启用它的话。这允许你透明地使用标准的`x-forwarded-for`和`x-forwarded-proto`头，很多前端代理服务器都会添加这些头信息（headers）。通过将这些属性中的一个或全部设置为非空的内容来开启该功能（它们是大多数代理约定的值，如果你只设置其中的一个，则另一个也会被自动设置）。

```java
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto 
```

如果你的代理使用不同的头部（headers），你可以通过向 application.properties 添加一些条目来自定义该值的配置，比如：

```java
server.tomcat.remote_ip_header=x-your-remote-ip-header
server.tomcat.protocol_header=x-your-protocol-header 
```

该值也可以配置为一个默认的，能够匹配信任的内部代理的正则表达式。默认情况下，受信任的 IP 包括 10/8, 192.168/16, 169.254/16 和 127/8。可以通过向 application.properties 添加一个条目来自定义该值的配置，比如：

```java
server.tomcat.internal_proxies=192\\.168\\.\\d{1,3}\\.\\d{1,3} 
```

**注**：只有在你使用一个 properties 文件作为配置的时候才需要双反斜杠。如果你使用 YAML，单个反斜杠就足够了，`192\.168\.\d{1,3}\.\d{1,3}`和上面的等价。

另外，通过在一个`TomcatEmbeddedServletContainerFactory` bean 中配置和添加`RemoteIpValve`，你就可以完全控制它的设置了。

# 64.9\. 使用 Jetty 替代 Tomcat

### 64.9\. 使用 Jetty 替代 Tomcat

Spring Boot starters（特别是 spring-boot-starter-web）默认都是使用 Tomcat 作为内嵌容器的。你需要排除那些 Tomcat 的依赖并包含 Jetty 的依赖。为了让这种处理尽可能简单，Spring Boot 将 Tomcat 和 Jetty 的依赖捆绑在一起，然后提供单独的 starters。

Maven 示例：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency> 
```

Gradle 示例：

```java
configurations {
    compile.exclude module: "spring-boot-starter-tomcat"
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.3.0.BUILD-SNAPSHOT")
    compile("org.springframework.boot:spring-boot-starter-jetty:1.3.0.BUILD-SNAPSHOT")
    // ...
} 
```

# 64.10\. 配置 Jetty

### 64.10\. 配置 Jetty

通常你可以遵循[Section 63.7, “Discover built-in options for external properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-discover-build-in-options-for-external-properties)关于`@ConfigurationProperties`（此处主要是 ServerProperties）的建议，但也要看下`EmbeddedServletContainerCustomizer`。Jetty API 相当丰富，一旦获取到`JettyEmbeddedServletContainerFactory`，你就可以使用很多方式修改它。或更彻底地就是添加你自己的`JettyEmbeddedServletContainerFactory`。

# 64.11\. 使用 Undertow 替代 Tomcat

### 64.11\. 使用 Undertow 替代 Tomcat

使用 Undertow 替代 Tomcat 和[使用 Jetty 替代 Tomcat](https://github.com/qibaoguang/Spring-Boot-Reference-Guide/edit/master/How-to_%20guides.md)非常类似。你需要排除 Tomat 依赖，并包含 Undertow starter。

Maven 示例：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency> 
```

Gradle 示例：

```java
configurations {
    compile.exclude module: "spring-boot-starter-tomcat"
}

dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web:1.3.0.BUILD-SNAPSHOT")
    compile 'org.springframework.boot:spring-boot-starter-undertow:1.3.0.BUILD-SNAPSHOT")
    // ...
} 
```

# 64.12\. 配置 Undertow

### 64.12\. 配置 Undertow

通常你可以遵循[Section 63.7, “Discover built-in options for external properties”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-discover-build-in-options-for-external-properties)关于`@ConfigurationProperties`（此处主要是 ServerProperties 和 ServerProperties.Undertow），但也要看下`EmbeddedServletContainerCustomizer`。一旦获取到`UndertowEmbeddedServletContainerFactory`，你就可以使用一个`UndertowBuilderCustomizer`修改 Undertow 的配置以满足你的需求。或更彻底地就是添加你自己的`UndertowEmbeddedServletContainerFactory`。

# 64.13\. 启用 Undertow 的多监听器

### 64.13\. 启用 Undertow 的多监听器（Multiple Listeners）

往`UndertowEmbeddedServletContainerFactory`添加一个`UndertowBuilderCustomizer`，然后添加一个监听者到`Builder`：

```java
@Bean
public UndertowEmbeddedServletContainerFactory embeddedServletContainerFactory() {
    UndertowEmbeddedServletContainerFactory factory = new UndertowEmbeddedServletContainerFactory();
    factory.addBuilderCustomizers(new UndertowBuilderCustomizer() {

        @Override
        public void customize(Builder builder) {
            builder.addHttpListener(8080, "0.0.0.0");
        }

    });
    return factory;
} 
```

# 64.14\. 使用 Tomcat7

### 64.14\. 使用 Tomcat7

Tomcat7 可用于 Spring Boot，但默认使用的是 Tomcat8。如果不能使用 Tomcat8（例如，你使用的是 Java1.6），你需要改变 classpath 去引用 Tomcat7。

# 64.14.1\. 通过 Maven 使用 Tomcat7

### 64.14.1\. 通过 Maven 使用 Tomcat7

如果正在使用 starter pom 和 parent，你只需要改变 Tomcat 的 version 属性，比如，对于一个简单的 webapp 或 service：

```java
<properties>
    <tomcat.version>7.0.59</tomcat.version>
</properties>
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    ...
</dependencies> 
```

# 64.14.2\. 通过 Gradle 使用 Tomcat7

### 64.14.2\. 通过 Gradle 使用 Tomcat7

你可以通过设置`tomcat.version`属性改变 Tomcat 的版本：

```java
ext['tomcat.version'] = '7.0.59'
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web'
} 
```

# 64.15\. 使用 Jetty8

### 64.15\. 使用 Jetty8

Jetty8 可用于 Spring Boot，但默认使用的是 Jetty9。如果不能使用 Jetty9（例如，因为你使用的是 Java1.6），你只需改变 classpath 去引用 Jetty8。你也需要排除 Jetty 的 WebSocket 相关的依赖。

# 64.15.1\. 通过 Maven 使用 Jetty8

### 64.15.1\. 通过 Maven 使用 Jetty8

如果正在使用 starter pom 和 parent，你只需添加 Jetty starter，去掉 WebSocket 依赖，并改变 version 属性，比如，对于一个简单的 webapp 或 service：

```java
<properties>
    <jetty.version>8.1.15.v20140411</jetty.version>
    <jetty-jsp.version>2.2.0.v201112011158</jetty-jsp.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.eclipse.jetty.websocket</groupId>
                <artifactId>*</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies> 
```

# 64.15.2\. 通过 Gradle 使用 Jetty8

### 64.15.2\. 通过 Gradle 使用 Jetty8

你可以设置`jetty.version`属性并排除相关的 WebSocket 依赖，比如对于一个简单的 webapp 或 service：

```java
ext['jetty.version'] = '8.1.15.v20140411'
dependencies {
    compile ('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    compile ('org.springframework.boot:spring-boot-starter-jetty') {
        exclude group: 'org.eclipse.jetty.websocket'
    }
} 
```

# 64.16\. 使用@ServerEndpoint 创建 WebSocket 端点

### 64.16\. 使用@ServerEndpoint 创建 WebSocket 端点

如果想在一个使用内嵌容器的 Spring Boot 应用中使用@ServerEndpoint，你需要声明一个单独的 ServerEndpointExporter @Bean：

```java
@Bean
public ServerEndpointExporter serverEndpointExporter() {
    return new ServerEndpointExporter();
} 
```

该 bean 将用底层的 WebSocket 容器注册任何的被`@ServerEndpoint`注解的 beans。当部署到一个单独的 servlet 容器时，该角色将被一个 servlet 容器初始化方法履行，ServerEndpointExporter bean 也就不是必需的了。

# 64.17\. 启用 HTTP 响应压缩

### 64.17\. 启用 HTTP 响应压缩

Spring Boot 提供两种启用 HTTP 压缩的机制;一种是 Tomcat 特有的，另一种是使用一个 filter，可以配合 Jetty，Tomcat 和 Undertow。

# 64.17.1\. 启用 Tomcat 的 HTTP 响应压缩

### 64.17.1\. 启用 Tomcat 的 HTTP 响应压缩

Tomcat 对 HTTP 响应压缩提供内建支持。默认是禁用的，但可以通过 application.properties 轻松的启用：

```java
server.tomcat.compression: on 
```

当设置为`on`时，Tomcat 将压缩响应的长度至少为 2048 字节。你可以配置一个整型值来设置该限制而不只是`on`，比如：

```java
server.tomcat.compression: 4096 
```

默认情况下，Tomcat 只压缩某些 MIME 类型的响应（text/html，text/xml 和 text/plain）。你可以使用`server.tomcat.compressableMimeTypes`属性进行自定义，比如：

```java
server.tomcat.compressableMimeTypes=application/json,application/xml 
```

# 64.17.2\. 使用 GzipFilter 开启 HTTP 响应压缩

### 64.17.2\. 使用 GzipFilter 开启 HTTP 响应压缩

如果你正在使用 Jetty 或 Undertow，或想要更精确的控制 HTTP 响应压缩，Spring Boot 为 Jetty 的 GzipFilter 提供自动配置。虽然该过滤器是 Jetty 的一部分，但它也兼容 Tomcat 和 Undertow。想要启用该过滤器，只需简单的为你的应用添加`org.eclipse.jetty:jetty-servlets`依赖。

GzipFilter 可以使用`spring.http.gzip.*`属性进行配置。具体参考[GzipFilterProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/GzipFilterProperties.java)。

# 65\. Spring MVC

### 65\. Spring MVC

# 65.1\. 编写一个 JSON REST 服务

### 65.1\. 编写一个 JSON REST 服务

在 Spring Boot 应用中，任何 Spring `@RestController`默认应该渲染为 JSON 响应，只要 classpath 下存在 Jackson2。例如：

```java
@RestController
public class MyController {

    @RequestMapping("/thing")
    public MyThing thing() {
            return new MyThing();
    }

} 
```

只要 MyThing 能够通过 Jackson2 序列化（比如，一个标准的 POJO 或 Groovy 对象），[localhost:8080/thing](http://localhost:8080/thing)默认响应一个 JSON 表示。有时在一个浏览器中你可能看到 XML 响应因为浏览器倾向于发送 XML 响应头。

# 65.2\. 编写一个 XML REST 服务

### 65.2\. 编写一个 XML REST 服务

如果 classpath 下存在 Jackson XML 扩展（jackson-dataformat-xml），它会被用来渲染 XML 响应，示例和 JSON 的非常相似。想要使用它，只需为你的项目添加以下的依赖：

```java
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency> 
```

你可能也想添加对 Woodstox 的依赖。它比 JDK 提供的默认 Stax 实现快很多，并且支持良好的格式化输出，提高了 namespace 处理能力：

```java
<dependency>
    <groupId>org.codehaus.woodstox</groupId>
    <artifactId>woodstox-core-asl</artifactId>
</dependency> 
```

如果 Jackson 的 XML 扩展不可用，Spring Boot 将使用 JAXB（JDK 默认提供），不过你需要为 MyThing 添加额外的注解`@XmlRootElement`：

```java
@XmlRootElement
public class MyThing {
    private String name;
    // .. getters and setters
} 
```

想要服务器渲染 XML 而不是 JSON，你可能需要发送一个`Accept: text/xml`头部（或使用浏览器）。

# 65.3\. 自定义 Jackson ObjectMapper

### 65.3\. 自定义 Jackson ObjectMapper

在一个 HTTP 交互中，Spring MVC（客户端和服务端）使用 HttpMessageConverters 协商内容转换。如果 classpath 下存在 Jackson，你就已经获取到 Jackson2ObjectMapperBuilder 提供的默认转换器。

创建的 ObjectMapper（或用于 Jackson XML 转换的 XmlMapper）实例默认有以下自定义属性：

*   `MapperFeature.DEFAULT_VIEW_INCLUSION`禁用
*   `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`禁用

Spring Boot 也有一些简化自定义该行为的特性。

你可以使用当前的 environment 配置 ObjectMapper 和 XmlMapper 实例。Jackson 提供一个扩展套件，可以用来简单的关闭或开启一些特性，你可以用它们配置 Jackson 处理的不同方面。这些特性在 Jackson 中使用 5 个枚举进行描述的，并被映射到 environment 的属性上：

| Jackson 枚举 | Environment 属性 |
| --- | --- |
| `com.fasterxml.jackson.databind.DeserializationFeature` | `spring.jackson.deserialization.<feature_name class="hljs-pi">=true</feature_name> | false` |
| `com.fasterxml.jackson.core.JsonGenerator.Feature` | `spring.jackson.generator.<feature_name class="hljs-pi">=true</feature_name> | false` |
| `com.fasterxml.jackson.databind.MapperFeature` | `spring.jackson.mapper.<feature_name class="hljs-pi">=true</feature_name> | false` |
| `com.fasterxml.jackson.core.JsonParser.Feature` | `spring.jackson.parser.<feature_name class="hljs-pi">=true</feature_name> | false` |
| `com.fasterxml.jackson.databind.SerializationFeature` | `spring.jackson.serialization.<feature_name class="hljs-pi">=true</feature_name> | false` |

例如，设置`spring.jackson.serialization.indent_output=true`可以开启漂亮打印。注意，由于[松绑定](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-relaxed-binding)的使用，`indent_output`不必匹配对应的枚举常量`INDENT_OUTPUT`。

如果想彻底替换默认的 ObjectMapper，你需要定义一个该类型的`@Bean`并将它标记为`@Primary`。

定义一个 Jackson2ObjectMapperBuilder 类型的`@Bean`将允许你自定义默认的 ObjectMapper 和 XmlMapper（分别用于 MappingJackson2HttpMessageConverter 和 MappingJackson2XmlHttpMessageConverter）。

另一种自定义 Jackson 的方法是向你的上下文添加`com.fasterxml.jackson.databind.Module`类型的 beans。它们会被注册入每个 ObjectMapper 类型的 bean，当为你的应用添加新特性时，这就提供了一种全局机制来贡献自定义模块。

最后，如果你提供任何 MappingJackson2HttpMessageConverter 类型的`@Beans`，那它们将替换 MVC 配置中的默认值。同时，也提供一个 HttpMessageConverters 类型的 bean，它有一些有用的方法可以获取默认的和用户增强的 message 转换器。

想要获取更多细节可查看[Section 65.4, “Customize the @ResponseBody rendering”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-customize-the-responsebody-rendering)和[WebMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)源码。

# 65.4\. 自定义@ResponseBody 渲染

### 65.4\. 自定义@ResponseBody 渲染

Spring 使用 HttpMessageConverters 渲染`@ResponseBody`（或来自`@RestController`的响应）。你可以通过在 Spring Boot 上下文中添加该类型的 beans 来贡献其他的转换器。如果你添加的 bean 类型默认已经包含了（像用于 JSON 转换的 MappingJackson2HttpMessageConverter），那它将替换默认的。Spring Boot 提供一个方便的 HttpMessageConverters 类型的 bean，它有一些有用的方法可以访问默认的和用户增强的 message 转换器（有用，比如你想要手动将它们注入到一个自定义的`RestTemplate`）。

在通常的 MVC 用例中，任何你提供的 WebMvcConfigurerAdapter beans 通过覆盖 configureMessageConverters 方法也能贡献转换器，但不同于通常的 MVC，你可以只提供你需要的转换器（因为 Spring Boot 使用相同的机制来贡献它默认的转换器）。最终，如果你通过提供自己的`@EnableWebMvc`注解覆盖 Spring Boot 默认的 MVC 配置，那你就可以完全控制，并使用来自 WebMvcConfigurationSupport 的 getMessageConverters 手动做任何事。

具体参考[WebMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)源码。

# 65.5\. 处理 Multipart 文件上传

### 65.5\. 处理 Multipart 文件上传

Spring Boot 采用 Servlet 3 `javax.servlet.http.Part` API 来支持文件上传。默认情况下，Spring Boot 配置 Spring MVC 在单个请求中每个文件最大 1Mb，最多 10Mb 的文件数据。你可以覆盖那些值，也可以设置临时文件存储的位置（比如，存储到`/tmp`文件夹下）及传递数据刷新到磁盘的阀值（通过使用 MultipartProperties 类暴露的属性）。如果你需要设置文件不受限制，例如，可以设置`multipart.maxFileSize`属性值为`-1`。

当你想要接收部分（multipart）编码文件数据作为 Spring MVC 控制器（controller）处理方法中被`@RequestParam`注解的 MultipartFile 类型的参数时，multipart 支持就非常有用了。

具体参考[MultipartAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/MultipartAutoConfiguration.java)源码。

# 65.6\. 关闭 Spring MVC DispatcherServlet

### 65.6\. 关闭 Spring MVC DispatcherServlet

Spring Boot 想要服务来自应用程序 root `/`下的所有内容。如果你想将自己的 servlet 映射到该目录下也是可以的，但当然你可能失去一些 Boot MVC 特性。为了添加你自己的 servlet，并将它映射到 root 资源，你只需声明一个 Servlet 类型的`@Bean`，并给它特定的 bean 名称`dispatcherServlet`（如果只想关闭但不替换它，你可以使用该名称创建不同类型的 bean）。

# 65.7\. 关闭默认的 MVC 配置

### 65.7\. 关闭默认的 MVC 配置

完全控制 MVC 配置的最简单方式是提供你自己的被`@EnableWebMvc`注解的`@Configuration`。这样所有的 MVC 配置都逃不出你的掌心。

# 65.8\. 自定义 ViewResolvers

### 65.8\. 自定义 ViewResolvers

ViewResolver 是 Spring MVC 的核心组件，它负责转换`@Controller`中的视图名称到实际的 View 实现。注意 ViewResolvers 主要用在 UI 应用中，而不是 REST 风格的服务（View 不是用来渲染`@ResponseBody`的）。Spring 有很多你可以选择的 ViewResolver 实现，并且 Spring 自己对如何选择相应实现也没发表意见。另一方面，Spring Boot 会根据 classpath 上的依赖和应用上下文为你安装一或两个 ViewResolver 实现。DispatcherServlet 使用所有在应用上下文中找到的解析器（resolvers），并依次尝试每一个直到它获取到结果，所以如果你正在添加自己的解析器，那就要小心顺序和你的解析器添加的位置。

WebMvcAutoConfiguration 将会为你的上下文添加以下 ViewResolvers：

*   bean id 为`defaultViewResolver`的 InternalResourceViewResolver。这个会定位可以使用 DefaultServlet 渲染的物理资源（比如，静态资源和 JSP 页面）。它在视图（view name）上应用了一个前缀和后缀（默认都为空，但你可以通过`spring.view.prefix`和`spring.view.suffix`外部配置设置），然后查找在 servlet 上下文中具有该路径的物理资源。可以通过提供相同类型的 bean 覆盖它。
*   id 为`beanNameViewResolver`的 BeanNameViewResolver。这是视图解析器链的一个非常有用的成员，它可以在 View 被解析时收集任何具有相同名称的 beans。
*   id 为`viewResolver`的 ContentNegotiatingViewResolver 只会在实际 View 类型的 beans 出现时添加。这是一个'主'解析器，它的职责会代理给其他解析器，它会尝试找到客户端发送的一个匹配'Accept'的 HTTP 头部。这有一篇有用的，关于你需要更多了解的[ContentNegotiatingViewResolver](https://spring.io/blog/2013/06/03/content-negotiation-using-views)的博客，也要具体查看下源码。通过定义一个名叫'viewResolver'的 bean，你可以关闭自动配置的 ContentNegotiatingViewResolver。
*   如果使用 Thymeleaf，你将有一个 id 为`thymeleafViewResolver`的 ThymeleafViewResolver。它会通过加前缀和后缀的视图名来查找资源（外部配置为`spring.thymeleaf.prefix`和`spring.thymeleaf.suffix`，对应的默认为'classpath:/templates/'和'.html'）。你可以通过提供相同名称的 bean 来覆盖它。
*   如果使用 FreeMarker，你将有一个 id 为`freeMarkerViewResolver`的 FreeMarkerViewResolver。它会使用加前缀和后缀（外部配置为`spring.freemarker.prefix`和`spring.freemarker.suffix`，对应的默认值为空和'.ftl'）的视图名从加载路径（外部配置为`spring.freemarker.templateLoaderPath`，默认为'classpath:/templates/'）下查找资源。你可以通过提供一个相同名称的 bean 来覆盖它。
*   如果使用 Groovy 模板（实际上只要你把 groovy-templates 添加到 classpath 下），你将有一个 id 为`groovyTemplateViewResolver`的 Groovy TemplateViewResolver。它会使用加前缀和后缀（外部属性为`spring.groovy.template.prefix`和`spring.groovy.template.suffix`，对应的默认值为'classpath:/templates/'和'.tpl'）的视图名从加载路径下查找资源。你可以通过提供一个相同名称的 bean 来覆盖它。
*   如果使用 Velocity，你将有一个 id 为`velocityViewResolver`的 VelocityViewResolver。它会使用加前缀和后缀（外部属性为`spring.velocity.prefix`和`spring.velocity.suffix`，对应的默认值为空和'.vm'）的视图名从加载路径（外部属性为`spring.velocity.resourceLoaderPath`，默认为'classpath:/templates/'）下查找资源。你可以通过提供一个相同名称的 bean 来覆盖它。

具体参考： [WebMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)，[ThymeleafAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)，[FreeMarkerAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)，[GroovyTemplateAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)，[VelocityAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)。

# 66\. 日志

### 66\. 日志

Spring Boot 除了 commons-logging API 外没有其他强制性的日志依赖，你有很多可选的日志实现。想要使用[Logback](http://logback.qos.ch/)，你需要包含它，及一些对 classpath 下 commons-logging 的绑定。最简单的方式是通过依赖`spring-boot-starter-logging`的 starter pom。对于一个 web 应用程序，你只需添加`spring-boot-starter-web`依赖，因为它依赖于 logging starter。例如，使用 Maven：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency> 
```

Spring Boot 有一个 LoggingSystem 抽象，用于尝试通过 classpath 上下文配置日志系统。如果 Logback 可用，则首选它。如果你唯一需要做的就是设置不同日志的级别，那可以通过在 application.properties 中使用`logging.level`前缀实现，比如：

```java
logging.level.org.springframework.web: DEBUG
logging.level.org.hibernate: ERROR 
```

你也可以使用`logging.file`设置日志文件的位置（除控制台之外，默认会输出到控制台）。

想要对日志系统进行更细粒度的配置，你需要使用正在说的 LoggingSystem 支持的原生配置格式。默认情况下，Spring Boot 从系统的默认位置加载原生配置（比如对于 Logback 为`classpath:logback.xml`），但你可以使用`logging.config`属性设置配置文件的位置。

# 66.1\. 配置 Logback

### 66.1\. 配置 Logback

如果你将一个 logback.xml 放到 classpath 根目录下，那它将会被从这加载。Spring Boot 提供一个默认的基本配置，如果你只是设置日志级别，那你可以包含它，比如：

```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <logger name="org.springframework.web" level="DEBUG"/>
</configuration> 
```

如果查看 spring-boot jar 包中的默认 logback.xml，你将会看到 LoggingSystem 为你创建的很多有用的系统属性，比如：

*   ${PID}，当前进程 id
*   ${LOG_FILE}，如果在 Boot 外部配置中设置了`logging.file`
*   ${LOG_PATH}，如果设置了`logging.path`（表示日志文件产生的目录）

Spring Boot 也提供使用自定义的 Logback 转换器在控制台上输出一些漂亮的彩色 ANSI 日志信息（不是日志文件）。具体参考默认的`base.xml`配置。

如果 Groovy 在 classpath 下，你也可以使用 logback.groovy 配置 Logback。

# 66.2\. 配置 Log4j

### 66.2\. 配置 Log4j

Spring Boot 也支持[Log4j](http://logging.apache.org/log4j/1.2)或[Log4j 2](http://logging.apache.org/log4j/2.x)作为日志配置，但只有在它们中的某个在 classpath 下存在的情况。如果你正在使用 starter poms 进行依赖装配，这意味着你需要排除 Logback，然后包含你选择的 Log4j 版本。如果你不使用 starter poms，那除了你选择的 Log4j 版本外还要提供 commons-logging（至少）。

最简单的方式可能就是通过 starter poms，尽管它需要排除一些依赖，比如，在 Maven 中：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j</artifactId>
</dependency> 
```

想要使用 Log4j 2，只需要依赖`spring-boot-starter-log4j2`而不是`spring-boot-starter-log4j`。

**注**：使用 Log4j 各版本的 starters 都会收集好依赖以满足 common logging 的要求（比如，Tomcat 中使用`java.util.logging`，但使用 Log4j 或 Log4j 2 作为输出）。具体查看 Actuator Log4j 或 Log4j 2 的示例，了解如何将它用于实战。

# 66.2.1\. 使用 YAML 或 JSON 配置 Log4j2

### 66.2.1\. 使用 YAML 或 JSON 配置 Log4j2

除了它的默认 XML 配置格式，Log4j 2 也支持 YAML 和 JSON 配置文件。想要使用其他配置文件格式来配置 Log4j 2，你需要添加合适的依赖到 classpath。为了使用 YAML，你需要添加`com.fasterxml.jackson.dataformat:jackson-dataformat-yaml`依赖，Log4j 2 将查找名称为`log4j2.yaml`或`log4j2.yml`的配置文件。为了使用 JSON，你需要添加`com.fasterxml.jackson.core:jackson-databind`依赖，Log4j 2 将查找名称为`log4j2.json`或`log4j2.jsn`的配置文件

# 67\. 数据访问

### 67\. 数据访问

# 67.1\. 配置一个数据源

### 67.1\. 配置一个数据源

想要覆盖默认的设置只需要定义一个你自己的 DataSource 类型的`@Bean`。Spring Boot 提供一个工具构建类 DataSourceBuilder，可用来创建一个标准的 DataSource（如果它处于 classpath 下），或者仅创建你自己的 DataSource，然后将它和在[Section 23.7.1, “Third-party configuration”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-3rd-party-configuration)解释的一系列 Environment 属性绑定。

比如：

```java
@Bean
@ConfigurationProperties(prefix="datasource.mine")
public DataSource dataSource() {
    return new FancyDataSource();
} 
```

```java
datasource.mine.jdbcUrl=jdbc:h2:mem:mydb
datasource.mine.user=sa
datasource.mine.poolSize=30 
```

具体参考'Spring Boot 特性'章节中的[Section 28.1, “Configure a DataSource”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-configure-datasource)和[DataSourceAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.java)类源码。

# 67.2\. 配置两个数据源

### 67.2\. 配置两个数据源

创建多个数据源和创建第一个工作都是一样的。如果使用针对 JDBC 或 JPA 的默认自动配置，你可能想要将其中一个设置为`@Primary`（然后它就能被任何`@Autowired`注入获取）。

```java
@Bean
@Primary
@ConfigurationProperties(prefix="datasource.primary")
public DataSource primaryDataSource() {
    return DataSourceBuilder.create().build();
}

@Bean
@ConfigurationProperties(prefix="datasource.secondary")
public DataSource secondaryDataSource() {
    return DataSourceBuilder.create().build();
} 
```

# 67.3\. 使用 Spring Data 仓库

### 67.3\. 使用 Spring Data 仓库

Spring Data 可以为你的`@Repository`接口创建各种风格的实现。Spring Boot 会为你处理所有事情，只要那些`@Repositories`接口跟你的`@EnableAutoConfiguration`类处于相同的包（或子包）。

对于很多应用来说，你需要做的就是将正确的 Spring Data 依赖添加到 classpath 下（对于 JPA 有一个`spring-boot-starter-data-jpa`，对于 Mongodb 有一个`spring-boot-starter-data-mongodb`），创建一些 repository 接口来处理`@Entity`对象。具体参考[JPA sample](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-data-jpa)或[Mongodb sample](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-data-mongodb)。

Spring Boot 会基于它找到的`@EnableAutoConfiguration`来尝试猜测你的`@Repository`定义的位置。想要获取更多控制，可以使用`@EnableJpaRepositories`注解（来自 Spring Data JPA）。

# 67.4\. 从 Spring 配置分离@Entity 定义

### 67.4\. 从 Spring 配置分离`@Entity`定义

Spring Boot 会基于它找到的`@EnableAutoConfiguration`来尝试猜测你的`@Entity`定义的位置。想要获取更多控制，你可以使用`@EntityScan`注解，比如：

```java
@Configuration
@EnableAutoConfiguration
@EntityScan(basePackageClasses=City.class)
public class Application {

    //...

} 
```

# 67.5\. 配置 JPA 属性

### 67.5\. 配置 JPA 属性

Spring Data JPA 已经提供了一些独立的配置选项（比如，针对 SQL 日志），并且 Spring Boot 会暴露它们，针对 hibernate 的外部配置属性也更多些。最常见的选项如下：

```java
spring.jpa.hibernate.ddl-auto: create-drop
spring.jpa.hibernate.naming_strategy: org.hibernate.cfg.ImprovedNamingStrategy
spring.jpa.database: H2
spring.jpa.show-sql: true 
```

（由于宽松的数据绑定策略，连字符或下划线作为属性 keys 作用应该是等效的）`ddl-auto`配置是个特殊情况，它有不同的默认设置，这取决于你是否使用一个内嵌数据库（create-drop）。当本地 EntityManagerFactory 被创建时，所有`spring.jpa.properties.*`属性都被作为正常的 JPA 属性（去掉前缀）传递进去了。

具体参考[HibernateJpaAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.java)和[JpaBaseConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)。

# 67.6\. 使用自定义的 EntityManagerFactory

### 67.6\. 使用自定义的 EntityManagerFactory

为了完全控制 EntityManagerFactory 的配置，你需要添加一个名为`entityManagerFactory`的`@Bean`。Spring Boot 自动配置会根据是否存在该类型的 bean 来关闭它的实体管理器（entity manager）。

# 67.7\. 使用两个 EntityManagers

### 67.7\. 使用两个 EntityManagers

即使默认的 EntityManagerFactory 工作的很好，你也需要定义一个新的 EntityManagerFactory，因为一旦出现第二个该类型的 bean，默认的将会被关闭。为了轻松的实现该操作，你可以使用 Spring Boot 提供的 EntityManagerBuilder，或者如果你喜欢的话可以直接使用来自 Spring ORM 的 LocalContainerEntityManagerFactoryBean。

示例：

```java
// add two data sources configured as above

@Bean
public LocalContainerEntityManagerFactoryBean customerEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(customerDataSource())
            .packages(Customer.class)
            .persistenceUnit("customers")
            .build();
}

@Bean
public LocalContainerEntityManagerFactoryBean orderEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(orderDataSource())
            .packages(Order.class)
            .persistenceUnit("orders")
            .build();
} 
```

上面的配置靠自己基本可以运行。想要完成作品你也需要为两个 EntityManagers 配置 TransactionManagers。其中的一个会被 Spring Boot 默认的 JpaTransactionManager 获取，如果你将它标记为`@Primary`。另一个需要显式注入到一个新实例。或你可以使用一个 JTA 事物管理器生成它两个。

# 67.8\. 使用普通的 persistence.xml

### 67.8\. 使用普通的 persistence.xml

Spring 不要求使用 XML 配置 JPA 提供者（provider），并且 Spring Boot 假定你想要充分利用该特性。如果你倾向于使用`persistence.xml`，那你需要定义你自己的 id 为'entityManagerFactory'的 LocalEntityManagerFactoryBean 类型的`@Bean`，并在那设置持久化单元的名称。

默认设置可查看[JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)

# 67.9\. 使用 Spring Data JPA 和 Mongo 仓库

### 67.9\. 使用 Spring Data JPA 和 Mongo 仓库

Spring Data JPA 和 Spring Data Mongo 都能自动为你创建 Repository 实现。如果它们同时出现在 classpath 下，你可能需要添加额外的配置来告诉 Spring Boot 你想要哪个（或两个）为你创建仓库。最明确地方式是使用标准的 Spring Data `@Enable*Repositories`，然后告诉它你的 Repository 接口的位置（此处*即可以是 Jpa，也可以是 Mongo，或者两者都是）。

这里也有`spring.data.*.repositories.enabled`标志，可用来在外部配置中开启或关闭仓库的自动配置。这在你想关闭 Mongo 仓库，但仍旧使用自动配置的 MongoTemplate 时非常有用。

相同的障碍和特性也存在于其他自动配置的 Spring Data 仓库类型（Elasticsearch, Solr）。只需要改变对应注解的名称和标志。

# 67.10\. 将 Spring Data 仓库暴露为 REST 端点

### 67.10\. 将 Spring Data 仓库暴露为 REST 端点

Spring Data REST 能够将 Repository 的实现暴露为 REST 端点，只要该应用启用 Spring MVC。

Spring Boot 暴露一系列来自`spring.data.rest`命名空间的有用属性来定制化[RepositoryRestConfiguration](http://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/core/config/RepositoryRestConfiguration.html)。如果需要提供其他定制，你可以创建一个继承自 SpringBootRepositoryRestMvcConfiguration 的`@Configuration`类。该类功能和 RepositoryRestMvcConfiguration 相同，但允许你继续使用`spring.data.rest.*`属性。

# 68\. 数据库初始化

### 68\. 数据库初始化

一个数据库可以使用不同的方式进行初始化，这取决于你的技术栈。或者你可以手动完成该任务，只要数据库是单独的过程。

# 68.1\. 使用 JPA 初始化数据库

### 68.1\. 使用 JPA 初始化数据库

JPA 有个生成 DDL 的特性，这些可以设置为在数据库启动时运行。这可以通过两个外部属性进行控制：

*   `spring.jpa.generate-ddl`（boolean）控制该特性的关闭和开启，跟实现者没关系
*   `spring.jpa.hibernate.ddl-auto`（enum）是一个 Hibernate 特性，用于更细力度的控制该行为。更多详情参考以下内容。

# 68.2\. 使用 Hibernate 初始化数据库

### 68.2\. 使用 Hibernate 初始化数据库

你可以显式设置`spring.jpa.hibernate.ddl-auto`，标准的 Hibernate 属性值有`none`，`validate`，`update`，`create`，`create-drop`。Spring Boot 根据你的数据库是否为内嵌数据库来选择相应的默认值，如果是内嵌型的则默认值为`create-drop`，否则为`none`。通过查看 Connection 类型可以检查是否为内嵌型数据库，hsqldb，h2 和 derby 是内嵌的，其他都不是。当从内存数据库迁移到一个真正的数据库时，你需要当心，在新的平台中不能对数据库表和数据是否存在进行臆断。你也需要显式设置`ddl-auto`，或使用其他机制初始化数据库。

此外，启动时处于 classpath 根目录下的 import.sql 文件会被执行。这在 demos 或测试时很有用，但在生产环境中你可能不期望这样。这是 Hibernate 的特性，和 Spring 没有一点关系。

# 68.3\. 使用 Spring JDBC 初始化数据库

### 68.3\. 使用 Spring JDBC 初始化数据库

Spring JDBC 有一个 DataSource 初始化特性。Spring Boot 默认启用了该特性，并从标准的位置 schema.sql 和 data.sql（位于 classpath 根目录）加载 SQL。此外，Spring Boot 将加载`schema-${platform}.sql`和`data-${platform}.sql`文件（如果存在），在这里 platform 是`spring.datasource.platform`的值，比如，你可以将它设置为数据库的供应商名称（hsqldb, h2, oracle, mysql, postgresql 等）。Spring Boot 默认启用 Spring JDBC 初始化快速失败特性，所以如果脚本导致异常产生，那应用程序将启动失败。脚本的位置可以通过设置`spring.datasource.schema`和`spring.datasource.data`来改变，如果设置`spring.datasource.initialize=false`则哪个位置都不会被处理。

你可以设置`spring.datasource.continueOnError=true`禁用快速失败特性。一旦应用程序成熟并被部署了很多次，那该设置就很有用，因为脚本可以充当"可怜人的迁移"-例如，插入失败时意味着数据已经存在，也就没必要阻止应用继续运行。

如果你想要在一个 JPA 应用中使用 schema.sql，那如果 Hibernate 试图创建相同的表，`ddl-auto=create-drop`将导致错误产生。为了避免那些错误，可以将`ddl-auto`设置为“”（推荐）或“none”。不管是否使用`ddl-auto=create-drop`，你总可以使用 data.sql 初始化新数据。

# 68.4\. 初始化 Spring Batch 数据库

### 68.4\. 初始化 Spring Batch 数据库

如果你正在使用 Spring Batch，那么它会为大多数的流行数据库平台预装 SQL 初始化脚本。Spring Boot 会检测你的数据库类型，并默认执行那些脚本，在这种情况下将关闭快速失败特性（错误被记录但不会阻止应用启动）。这是因为那些脚本是可信任的，通常不会包含 bugs，所以错误会被忽略掉，并且对错误的忽略可以让脚本具有幂等性。你可以使用`spring.batch.initializer.enabled=false`显式关闭初始化功能。

# 68.5\. 使用一个高级别的数据迁移工具

### 68.5\. 使用一个高级别的数据迁移工具

Spring Boot 跟高级别的数据迁移工具[Flyway](http://flywaydb.org/)(基于 SQL)和[Liquibase](http://www.liquibase.org/)(XML)工作的很好。通常我们倾向于 Flyway，因为它一眼看去好像很容易，另外它通常不需要平台独立：一般一个或至多需要两个平台。

# 68.5.1\. 启动时执行 Flyway 数据库迁移

### 68.5.1\. 启动时执行 Flyway 数据库迁移

想要在启动时自动运行 Flyway 数据库迁移，需要将`org.flywaydb:flyway-core`添加到你的 classpath 下。

迁移是一些`V<VERSION>__<NAME>.sql`格式的脚本（`<VERSION>`是一个下划线分割的版本号，比如'1'或'2_1'）。默认情况下，它们存放在一个`classpath:db/migration`的文件夹中，但你可以使用`flyway.locations`（一个列表）来改变它。详情可参考 flyway-core 中的 Flyway 类，查看一些可用的配置，比如 schemas。Spring Boot 在[FlywayProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayProperties.java)中提供了一个小的属性集，可用于禁止迁移，或关闭位置检测。

默认情况下，Flyway 将自动注入（`@Primary`）DataSource 到你的上下文，并用它进行数据迁移。如果你想使用一个不同的 DataSource，你可以创建一个，并将它标记为`@FlywayDataSource`的`@Bean`-如果你这样做了，且想要两个数据源，记得创建另一个并将它标记为`@Primary`。或者你可以通过在外部配置文件中设置`flyway.[url,user,password]`来使用 Flyway 的原生 DataSource。

这是一个[Flyway 示例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-flyway)，你可以作为参考。

# 68.5.2\. 启动时执行 Liquibase 数据库迁移

### 68.5.2\. 启动时执行 Liquibase 数据库迁移

想要在启动时自动运行 Liquibase 数据库迁移，你需要将`org.liquibase:liquibase-core`添加到 classpath 下。

主改变日志（master change log）默认从`db/changelog/db.changelog-master.yaml`读取，但你可以使用`liquibase.change-log`进行设置。详情查看[LiquibaseProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/liquibase/LiquibaseProperties.java)以获取可用设置，比如上下文，默认的 schema 等。

这里有个[Liquibase 示例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-liquibase)可作为参考。

# 69\. 批处理应用

### 69\. 批处理应用

# 69.1\. 在启动时执行 Spring Batch 作业

### 69.1\. 在启动时执行 Spring Batch 作业

你可以在上下文的某个地方添加`@EnableBatchProcessing`来启用 Spring Batch 的自动配置功能。

默认情况下，在启动时它会执行应用的所有作业（Jobs），具体查看[JobLauncherCommandLineRunner](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/JobLauncherCommandLineRunner.java)。你可以通过指定`spring.batch.job.names`（多个作业名以逗号分割）来缩小到一个特定的作业或多个作业。

如果应用上下文包含一个 JobRegistry，那么处于`spring.batch.job.names`中的作业将会从 registry 中查找，而不是从上下文中自动装配。这是复杂系统中常见的一个模式，在这些系统中多个作业被定义在子上下文和注册中心。

具体参考[BatchAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java)和[@EnableBatchProcessing](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.java)。

# 70\. 执行器（Actuator）

### 70\. 执行器（Actuator）

# 70.1\. 改变 HTTP 端口或执行器端点的地址

### 70.1\. 改变 HTTP 端口或执行器端点的地址

在一个单独的应用中，执行器的 HTTP 端口默认和主 HTTP 端口相同。想要让应用监听不同的端口，你可以设置外部属性`management.port`。为了监听一个完全不同的网络地址（比如，你有一个用于管理的内部网络和一个用于用户应用程序的外部网络），你可以将`management.address`设置为一个可用的 IP 地址，然后将服务器绑定到该地址。

查看[ManagementServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/autoconfigure/ManagementServerProperties.java)源码和'Production-ready 特性'章节中的[Section 41.3, “Customizing the management server port”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready-customizing-management-server-port)来获取更多详情。

# 70.2\. 自定义'白标'（whitelabel，可以了解下相关理念）错误页面

### 70.2\. 自定义'白标'（whitelabel，可以了解下相关理念）错误页面

Spring Boot 安装了一个'whitelabel'错误页面，如果你遇到一个服务器错误（机器客户端消费的是 JSON，其他媒体类型则会看到一个具有正确错误码的合乎情理的响应），那就能在客户端浏览器中看到该页面。你可以设置`error.whitelabel.enabled=false`来关闭该功能，但通常你想要添加自己的错误页面来取代 whitelabel。确切地说，如何实现取决于你使用的模板技术。例如，你正在使用 Thymeleaf，你将添加一个 error.html 模板。如果你正在使用 FreeMarker，那你将添加一个 error.ftl 模板。通常，你需要的只是一个名称为 error 的 View，和/或一个处理`/error`路径的`@Controller`。除非你替换了一些默认配置，否则你将在你的 ApplicationContext 中找到一个 BeanNameViewResolver，所以一个 id 为 error 的`@Bean`可能是完成该操作的一个简单方式。详情参考[ErrorMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ErrorMvcAutoConfiguration.java)。

查看[Error Handling](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-error-handling)章节，了解下如何将处理器（handlers）注册到 servlet 容器中。

# 71\. 安全

### 71\. 安全

# 71.1\. 关闭 Spring Boot 安全配置

### 71.1\. 关闭 Spring Boot 安全配置

不管你在应用的什么地方定义了一个使用`@EnableWebSecurity`注解的`@Configuration`，它将会关闭 Spring Boot 中的默认 webapp 安全设置。想要调整默认值，你可以尝试设置`security.*`属性（具体查看[SecurityProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/SecurityProperties.java)和[常见应用属性](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#common-application-properties-security)的 SECURITY 章节）。

# 71.2\. 改变 AuthenticationManager 并添加用户账号

### 71.2\. 改变 AuthenticationManager 并添加用户账号

如果你提供了一个 AuthenticationManager 类型的`@Bean`，那么默认的就不会被创建了，所以你可以获得 Spring Security 可用的全部特性（比如，[不同的认证选项](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication)）。

Spring Security 也提供了一个方便的 AuthenticationManagerBuilder，可用于构建具有常见选项的 AuthenticationManager。在一个 webapp 中，推荐将它注入到 WebSecurityConfigurerAdapter 的一个 void 方法中，比如：

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
            auth.inMemoryAuthentication()
                .withUser("barry").password("password").roles("USER"); // ... etc.
    }

    // ... other stuff for application security
} 
```

如果把它放到一个内部类或一个单独的类中，你将得到最好的结果（也就是不跟很多其他`@Beans`混合在一起将允许你改变实例化的顺序）。[secure web sample](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-secure)是一个有用的参考模板。

如果你遇到了实例化问题（比如，使用 JDBC 或 JPA 进行用户详细信息的存储），那将 AuthenticationManagerBuilder 回调提取到一个 GlobalAuthenticationConfigurerAdapter（放到 init()方法内以防其他地方也需要 authentication manager）可能是个不错的选择，比如：

```java
@Configuration
public class AuthenticationManagerConfiguration extends

    GlobalAuthenticationConfigurerAdapter {
    @Override
    public void init(AuthenticationManagerBuilder auth) {
        auth.inMemoryAuthentication() // ... etc.
    }

} 
```

# 71.3\. 当前端使用代理服务器时，启用 HTTPS

### 71.3\. 当前端使用代理服务器时，启用 HTTPS

对于任何应用来说，确保所有的主端点（URL）都只在 HTTPS 下可用是个重要的苦差事。如果你使用 Tomcat 作为 servlet 容器，那 Spring Boot 如果发现一些环境设置的话，它将自动添加 Tomcat 自己的 RemoteIpValve，你也可以依赖于 HttpServletRequest 来报告是否请求是安全的（即使代理服务器的 downstream 处理真实的 SSL 终端）。这个标准行为取决于某些请求头是否出现（`x-forwarded-for`和`x-forwarded-proto`），这些请求头的名称都是约定好的，所以对于大多数前端和代理都是有效的。

你可以向 application.properties 添加以下设置里开启该功能，比如：

```java
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto 
```

（这些属性出现一个就会开启该功能，或者你可以通过添加一个 TomcatEmbeddedServletContainerFactory bean 自己添加 RemoteIpValve）

Spring Security 也可以配置成针对所以或某些请求需要一个安全渠道（channel）。想要在一个 Spring Boot 应用中开启它，你只需将 application.properties 中的`security.require_ssl`设置为`true`即可。

# 72\. 热交换

### 72\. 热交换

# 72.1\. 重新加载静态内容

### 72.1\. 重新加载静态内容

Spring Boot 有很多用于热加载的选项。使用 IDE 开发是一个不错的方式，特别是需要调试的时候（所有的现代 IDEs 都允许重新加载静态资源，通常也支持对变更的 Java 类进行热交换）。[Maven 和 Gradle 插件](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins)也支持命令行下的静态文件热加载。如果你使用其他高级工具编写 css/js，并使用外部的 css/js 编译器，那你就可以充分利用该功能。

# 72.2\. 在不重启容器的情况下重新加载 Thymeleaf 模板

### 72.2\. 在不重启容器的情况下重新加载 Thymeleaf 模板

如果你正在使用 Thymeleaf，那就将`spring.thymeleaf.cache`设置为 false。查看[ThymeleafAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)可以获取其他 Thymeleaf 自定义选项。

# 72.3\. 在不重启容器的情况下重新加载 FreeMarker 模板

### 72.3\. 在不重启容器的情况下重新加载 FreeMarker 模板

如果你正在使用 FreeMarker，那就将`spring.freemarker.cache`设置为 false。查看[FreeMarkerAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java) 可以获取其他 FreeMarker 自定义选项。

# 72.4\. 在不重启容器的情况下重新加载 Groovy 模板

### 72.4\. 在不重启容器的情况下重新加载 Groovy 模板

如果你正在使用 Groovy 模板，那就将`spring.groovy.template.cache`设置为 false。查看[GroovyTemplateAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java)可以获取其他 Groovy 自定义选项。

# 72.5\. 在不重启容器的情况下重新加载 Velocity 模板

### 72.5\. 在不重启容器的情况下重新加载 Velocity 模板

如果你正在使用 Velocity，那就将`spring.velocity.cache`设置为 false。查看[VelocityAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/velocity/VelocityAutoConfiguration.java)可以获取其他 Velocity 自定义选项。

# 72.6\. 在不重启容器的情况下重新加载 Java 类

### 72.6\. 在不重启容器的情况下重新加载 Java 类

现代 IDEs（Eclipse, IDEA 等）都支持字节码的热交换，所以如果你做了一个没有影响类或方法签名的改变，它会利索地重新加载并没有任何影响。

[Spring Loaded](https://github.com/spring-projects/spring-loaded)在这方面走的更远，它能够重新加载方法签名改变的类定义。如果对它进行一些自定义配置可以强制 ApplicationContext 刷新自己（但没有通用的机制来确保这对一个运行中的应用总是安全的，所以它可能只是一个开发时间的技巧）。

# 72.6.1\. 使用 Maven 配置 Spring Loaded

### 72.6.1\. 使用 Maven 配置 Spring Loaded

为了在 Maven 命令行下使用 Spring Loaded，你只需将它作为一个依赖添加到 Spring Boot 插件声明中即可，比如：

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.0.RELEASE</version>
        </dependency>
    </dependencies>
</plugin> 
```

正常情况下，这在 Eclipse 和 IntelliJ 中工作的相当漂亮，只要它们有相应的，和 Maven 默认一致的构建配置（Eclipse m2e 对此支持的更好，开箱即用）。

# 72.6.2\. 使用 Gradle 和 IntelliJ 配置 Spring Loaded

### 72.6.2\. 使用 Gradle 和 IntelliJ 配置 Spring Loaded

如果想将 Spring Loaded 和 Gradle，IntelliJ 结合起来，那你需要付出代价。默认情况下，IntelliJ 将类编译到一个跟 Gradle 不同的位置，这会导致 Spring Loaded 监控失败。

为了正确配置 IntelliJ，你可以使用`idea` Gradle 插件：

```java
buildscript {
    repositories { jcenter() }
    dependencies {
        classpath "org.springframework.boot:spring-boot-gradle-plugin:1.3.0.BUILD-SNAPSHOT"
        classpath 'org.springframework:springloaded:1.2.0.RELEASE'
    }
}

apply plugin: 'idea'

idea {
    module {
        inheritOutputDirs = false
        outputDir = file("$buildDir/classes/main/")
    }
}

// ... 
```

**注**：IntelliJ 必须配置跟命令行 Gradle 任务相同的 Java 版本，并且 springloaded 必须作为一个 buildscript 依赖被包含进去。

此外，你也可以启用 Intellij 内部的`Make Project Automatically`，这样不管什么时候只要文件被保存都会自动编译你的代码。

# 73\. 构建

### 73\. 构建

# 73.1\. 使用 Maven 自定义依赖版本

### 73.1\. 使用 Maven 自定义依赖版本

如果你使用 Maven 进行一个直接或间接继承`spring-boot-dependencies`（比如`spring-boot-starter-parent`）的构建，并想覆盖一个特定的第三方依赖，那你可以添加合适的`<properties>`元素。浏览[spring-boot-dependencies](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-dependencies/pom.xml) POM 可以获取一个全面的属性列表。例如，想要选择一个不同的 slf4j 版本，你可以添加以下内容：

```java
<properties>
    <slf4j.version>1.7.5<slf4j.version>
</properties> 
```

**注**：这只在你的 Maven 项目继承（直接或间接）自`spring-boot-dependencies`才有用。如果你使用`<scope>import</scope>`，将`spring-boot-dependencies`添加到自己的`dependencyManagement`片段，那你必须自己重新定义 artifact 而不是覆盖属性。

**注**：每个 Spring Boot 发布都是基于一些特定的第三方依赖集进行设计和测试的，覆盖版本可能导致兼容性问题。

# 73.2\. 使用 Maven 创建可执行 JAR

### 73.2\. 使用 Maven 创建可执行 JAR

`spring-boot-maven-plugin`能够用来创建可执行的'胖'JAR。如果你正在使用`spring-boot-starter-parent` POM，你可以简单地声明该插件，然后你的 jar 将被重新打包：

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build> 
```

如果没有使用 parent POM，你仍旧可以使用该插件。不过，你需要另外添加一个`<executions>`片段：

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>1.3.0.BUILD-SNAPSHOT</version>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build> 
```

查看[插件文档](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/usage.html)获取详细的用例。

# 73.3\. 创建其他的可执行 JAR

### 73.3\. 创建其他的可执行 JAR

如果你想将自己的项目以 library jar 的形式被其他项目依赖，并且需要它是一个可执行版本（例如 demo），你需要使用略微不同的方式来配置该构建。

对于 Maven 来说，正常的 JAR 插件和 Spring Boot 插件都有一个'classifier'，你可以添加它来创建另外的 JAR。示例如下（使用 Spring Boot Starter Parent 管理插件版本，其他配置采用默认设置）：

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </plugin>
    </plugins>
</build> 
```

上述配置会产生两个 jars，默认的一个和使用带有 classifier 'exec'的 Boot 插件构建的可执行的一个。

对于 Gradle 用户来说，步骤类似。示例如下：

```java
bootRepackage  {
    classifier = 'exec'
} 
```

# 73.4\. 在可执行 jar 运行时提取特定的版本

### 73.4\. 在可执行 jar 运行时提取特定的版本

在一个可执行 jar 中，为了运行，多数内嵌的库不需要拆包（unpacked），然而有一些库可能会遇到问题。例如，JRuby 包含它自己的内嵌 jar，它假定`jruby-complete.jar`本身总是能够直接作为文件访问的。

为了处理任何有问题的库，你可以标记那些特定的内嵌 jars，让它们在可执行 jar 第一次运行时自动解压到一个临时文件夹中。例如，为了将 JRuby 标记为使用 Maven 插件拆包，你需要添加如下的配置：

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <requiresUnpack>
                    <dependency>
                        <groupId>org.jruby</groupId>
                        <artifactId>jruby-complete</artifactId>
                    </dependency>
                </requiresUnpack>
            </configuration>
        </plugin>
    </plugins>
</build> 
```

使用 Gradle 完全上述操作：

```java
springBoot  {
    requiresUnpack = ['org.jruby:jruby-complete']
} 
```

# 73.6\. 远程调试一个使用 Maven 启动的 Spring Boot 项目

### 73.6\. 远程调试一个使用 Maven 启动的 Spring Boot 项目

想要为使用 Maven 启动的 Spring Boot 应用添加一个远程调试器，你可以使用[mave 插件](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/)的 jvmArguments 属性。详情参考[示例](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/examples/run-debug.html)。

# 73.7\. 远程调试一个使用 Gradle 启动的 Spring Boot 项目

### 73.7\. 远程调试一个使用 Gradle 启动的 Spring Boot 项目

想要为使用 Gradle 启动的 Spring Boot 应用添加一个远程调试器，你可以使用 build.gradle 的 applicationDefaultJvmArgs 属性或`--debug-jvm`命令行选项。

build.gradle：

```java
applicationDefaultJvmArgs = [
    "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005"
] 
```

命令行：

```java
$ gradle run --debug-jvm 
```

详情查看[Gradle 应用插件](http://www.gradle.org/docs/current/userguide/application_plugin.html)。

# 73.8\. 使用 Ant 构建可执行存档（archive）

### 73.8\. 使用 Ant 构建可执行存档（archive）

想要使用 Ant 进行构建，你需要抓取依赖，编译，然后像通常那样创建一个 jar 或 war 存档。为了让它可以执行：

1.  使用合适的启动器配置`Main-Class`，比如对于 jar 文件使用 JarLauncher，然后将其他需要的属性以 manifest 实体指定，主要是一个`Start-Class`。
2.  将运行时依赖添加到一个内嵌的'lib'目录（对于 jar），`provided`（内嵌容器）依赖添加到一个内嵌的`lib-provided`目录。记住***不要***压缩存档中的实体。
3.  在存档的根目录添加`spring-boot-loader`类（这样`Main-Class`就可用了）。

示例：

```java
<target name="build" depends="compile">
    <copy todir="target/classes/lib">
        <fileset dir="lib/runtime" />
    </copy>
    <jar destfile="target/spring-boot-sample-actuator-${spring-boot.version}.jar" compress="false">
        <fileset dir="target/classes" />
        <fileset dir="src/main/resources" />
        <zipfileset src="lib/loader/spring-boot-loader-jar-${spring-boot.version}.jar" />
        <manifest>
            <attribute name="Main-Class" value="org.springframework.boot.loader.JarLauncher" />
            <attribute name="Start-Class" value="${start-class}" />
        </manifest>
    </jar>
</target> 
```

该 Actuator 示例中有一个 build.xml 文件，可以使用以下命令来运行：

```java
$ ant -lib <path_to class="hljs-pi">/ivy-2.2.jar</path_to> 
```

在上述操作之后，你可以使用以下命令运行该应用：

```java
$ java -jar target/*.jar 
```

# 73.9\. 如何使用 Java6

### 73.9\. 如何使用 Java6

如果想在 Java6 环境中使用 Spring Boot，你需要改变一些配置。具体的变化取决于你应用的功能。

# 73.9.1\. 内嵌 Servlet 容器兼容性

### 73.9.1\. 内嵌 Servlet 容器兼容性

如果你在使用 Boot 的内嵌 Servlet 容器，你需要使用一个兼容 Java6 的容器。Tomcat 7 和 Jetty 8 都是 Java 6 兼容的。具体参考[Section 63.15, “Use Tomcat 7”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-tomcat-7)和[Section 63.16, “Use Jetty 8”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-jetty-8)。

# 73.9.2\. JTA API 兼容性

### 73.9.2\. JTA API 兼容性

Java 事务 API 自身并不要求 Java 7，而是官方的 API jar 包含的已构建类要求 Java 7。如果你正在使用 JTA，那么你需要使用能够在 Java 6 工作的构建版本替换官方的 JTA 1.2 API jar。为了完成该操作，你需要排除任何对`javax.transaction:javax.transaction-api`的传递依赖，并使用`org.jboss.spec.javax.transaction:jboss-transaction-api_1.2_spec:1.0.0.Final`依赖替换它们。

# 74\. 传统部署

### 74\. 传统部署

# 74.1\. 创建一个可部署的 war 文件

### 74.1\. 创建一个可部署的 war 文件

产生一个可部署 war 包的第一步是提供一个 SpringBootServletInitializer 子类，并覆盖它的 configure 方法。这充分利用了 Spring 框架对 Servlet 3.0 的支持，并允许你在应用通过 servlet 容器启动时配置它。通常，你只需把应用的主类改为继承 SpringBootServletInitializer 即可：

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

} 
```

下一步是更新你的构建配置，这样你的项目将产生一个 war 包而不是 jar 包。如果你使用 Maven，并使用`spring-boot-starter-parent`（为了配置 Maven 的 war 插件），所有你需要做的就是更改 pom.xml 的 packaging 为 war：

```java
<packaging>war</packaging> 
```

如果你使用 Gradle，你需要修改 build.gradle 来将 war 插件应用到项目上：

```java
apply plugin: 'war' 
```

该过程最后的一步是确保内嵌的 servlet 容器不能干扰 war 包将部署的 servlet 容器。为了达到这个目的，你需要将内嵌容器的依赖标记为 provided。

如果使用 Maven：

```java
<dependencies>
    <!-- … -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- … -->
</dependencies> 
```

如果使用 Gradle：

```java
dependencies {
    // …
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
    // …
} 
```

如果你使用[Spring Boot 构建工具](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins)，将内嵌容器依赖标记为 provided 将产生一个可执行 war 包，在`lib-provided`目录有该 war 包的 provided 依赖。这意味着，除了部署到 servlet 容器，你还可以通过使用命令行`java -jar`命令来运行应用。

**注**：查看 Spring Boot 基于以上配置的一个[Maven 示例应用](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-traditional/pom.xml)。

# 74.2\. 为老的 servlet 容器创建一个可部署的 war 文件

### 74.2\. 为老的 servlet 容器创建一个可部署的 war 文件

老的 Servlet 容器不支持在 Servlet 3.0 中使用的 ServletContextInitializer 启动处理。你仍旧可以在这些容器使用 Spring 和 Spring Boot，但你需要为应用添加一个 web.xml，并将它配置为通过一个 DispatcherServlet 加载一个 ApplicationContext。

# 74.3\. 将现有的应用转换为 Spring Boot

### 74.3\. 将现有的应用转换为 Spring Boot

对于一个非 web 项目，转换为 Spring Boot 应用很容易（抛弃创建 ApplicationContext 的代码，取而代之的是调用 SpringApplication 或 SpringApplicationBuilder）。Spring MVC web 应用通常先创建一个可部署的 war 应用，然后将它迁移为一个可执行的 war 或 jar。建议阅读[Getting Started Guide on Converting a jar to a war.](http://spring.io/guides/gs/convert-jar-to-war/)。

通过继承 SpringBootServletInitializer 创建一个可执行 war（比如，在一个名为 Application 的类中），然后添加 Spring Boot 的`@EnableAutoConfiguration`注解。示例：

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        // Customize the application or call application.sources(...) to add sources
        // Since our example is itself a @Configuration class we actually don't
        // need to override this method.
        return application;
    }

} 
```

记住不管你往 sources 放什么东西，它仅是一个 Spring ApplicationContext，正常情况下，任何生效的在这里也会起作用。有一些 beans 你可以先移除，然后让 Spring Boot 提供它的默认实现，不过有可能需要先完成一些事情。

静态资源可以移到 classpath 根目录下的`/public`（或`/static`，`/resources`，`/META-INF/resources`）。同样的方式也适合于`messages.properties`（Spring Boot 在 classpath 根目录下自动发现这些配置）。

美妙的（Vanilla usage of）Spring DispatcherServlet 和 Spring Security 不需要改变。如果你的应用有其他特性，比如使用其他 servlets 或 filters，那你可能需要添加一些配置到你的 Application 上下文中，按以下操作替换 web.xml 的那些元素：

*   在容器中安装一个 Servlet 或 ServletRegistrationBean 类型的`@Bean`，就好像 web.xml 中的`<servlet/>`和`<servlet-mapping/>`。
*   同样的添加一个 Filter 或 FilterRegistrationBean 类型的`@Bean`（类似于`<filter/>`和`<filter-mapping/>`）。
*   在 XML 文件中的 ApplicationContext 可以通过`@Import`添加到你的 Application 中。简单的情况下，大量使用注解配置可以在几行内定义`@Bean`定义。

一旦 war 可以使用，我们就通过添加一个 main 方法到 Application 来让它可以执行，比如：

```java
public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
} 
```

应用可以划分为多个类别：

*   没有 web.xml 的 Servlet 3.0+应用
*   有 web.xml 的应用
*   有上下文层次的应用
*   没有上下文层次的应用

所有这些都可以进行适当的转化，但每个可能需要稍微不同的技巧。

Servlet 3.0+的应用转化的相当简单，如果它们已经使用 Spring Servlet 3.0+初始化器辅助类。通常所有来自一个存在的 WebApplicationInitializer 的代码可以移到一个 SpringBootServletInitializer 中。如果一个存在的应用有多个 ApplicationContext（比如，如果它使用 AbstractDispatcherServletInitializer），那你可以将所有上下文源放进一个单一的 SpringApplication。你遇到的主要难题可能是如果那样不能工作，那你就要维护上下文层次。参考示例[entry on building a hierarchy](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-build-an-application-context-hierarchy)。一个存在的包含 web 相关特性的父上下文通常需要分解，这样所有的 ServletContextAware 组件都处于子上下文中。

对于还不是 Spring 应用的应用来说，上面的指南有助于你把应用转换为一个 Spring Boot 应用，但你也可以选择其他方式。

# 74.4\. 部署 WAR 到 Weblogic

### 74.4\. 部署 WAR 到 Weblogic

想要将 Spring Boot 应用部署到 Weblogic，你需要确保你的 servlet 初始化器直接实现 WebApplicationInitializer（即使你继承的基类已经实现了它）。

一个传统的 Weblogic 初始化器可能如下所示：

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.web.SpringBootServletInitializer;
import org.springframework.web.WebApplicationInitializer;

@SpringBootApplication
public class MyApplication extends SpringBootServletInitializer implements WebApplicationInitializer {

} 
```

如果使用 logback，你需要告诉 Weblogic 你倾向使用的打包版本而不是服务器预装的版本。你可以通过添加一个具有如下内容的`WEB-INF/weblogic.xml`实现该操作：

```java
<?xml version="1.0" encoding="UTF-8"?>
<wls:weblogic-web-app
    xmlns:wls="http://xmlns.oracle.com/weblogic/weblogic-web-app"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/ejb-jar_3_0.xsd
        http://xmlns.oracle.com/weblogic/weblogic-web-app
        http://xmlns.oracle.com/weblogic/weblogic-web-app/1.4/weblogic-web-app.xsd">
    <wls:container-descriptor>
        <wls:prefer-application-packages>
            <wls:package-name>org.slf4j</wls:package-name>
        </wls:prefer-application-packages>
    </wls:container-descriptor>
</wls:weblogic-web-app> 
```