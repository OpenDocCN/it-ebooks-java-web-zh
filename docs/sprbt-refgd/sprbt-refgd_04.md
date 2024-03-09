# IV. Spring Boot 特性

### Spring Boot 特性

# 22\. SpringApplication

### 22\. SpringApplication

SpringApplication 类提供了一种从 main()方法启动 Spring 应用的便捷方式。在很多情况下，你只需委托给 SpringApplication.run 这个静态方法：

```java
public static void main(String[] args){
    SpringApplication.run(MySpringConfiguration.class, args);
} 
```

当应用启动时，你应该会看到类似下面的东西（这是何方神兽？？）：

```java
.   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v1.2.2.BUILD-SNAPSHOT

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatEmbeddedServletContainerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658) 
```

默认情况下会显示 INFO 级别的日志信息，包括一些相关的启动详情，比如启动应用的用户等。

# 22.1\. 自定义 Banner

### 22.1\. 自定义 Banner

通过在 classpath 下添加一个 banner.txt 或设置 banner.location 来指定相应的文件可以改变启动过程中打印的 banner。如果这个文件有特殊的编码，你可以使用 banner.encoding 设置它（默认为 UTF-8）。

在 banner.txt 中可以使用如下的变量：

| 变量 | 描述 |
| --- | --- |
| ${application.version} | MANIFEST.MF 中声明的应用版本号，例如 1.0 |
| ${application.formatted-version} | MANIFEST.MF 中声明的被格式化后的应用版本号（被括号包裹且以 v 作为前缀），用于显示，例如(v1.0) |
| ${spring-boot.version} | 正在使用的 Spring Boot 版本号，例如 1.2.2.BUILD-SNAPSHOT |
| ${spring-boot.formatted-version} | 正在使用的 Spring Boot 被格式化后的版本号（被括号包裹且以 v 作为前缀）, 用于显示，例如(v1.2.2.BUILD-SNAPSHOT) |

**注**：如果想以编程的方式产生一个 banner，可以使用 SpringBootApplication.setBanner(…)方法。使用 org.springframework.boot.Banner 接口，实现你自己的 printBanner()方法。

# 22.2\. 自定义 SpringApplication

### 22.2\. 自定义 SpringApplication

如果默认的 SpringApplication 不符合你的口味，你可以创建一个本地的实例并自定义它。例如，关闭 banner 你可以这样写：

```java
public static void main(String[] args){
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setShowBanner(false);
    app.run(args);
} 
```

**注**：传递给 SpringApplication 的构造器参数是 spring beans 的配置源。在大多数情况下，这些将是@Configuration 类的引用，但它们也可能是 XML 配置或要扫描包的引用。

你也可以使用 application.properties 文件来配置 SpringApplication。具体参考 Externalized 配置。查看配置选项的完整列表，可参考[SpringApplication Javadoc](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/SpringApplication.html).

# 22.3\. 流畅的构建 API

### 22.3\. 流畅的构建 API

如果你需要创建一个分层的 ApplicationContext（多个具有父子关系的上下文），或你只是喜欢使用流畅的构建 API，你可以使用 SpringApplicationBuilder。SpringApplicationBuilder 允许你以链式方式调用多个方法，包括可以创建层次结构的 parent 和 child 方法。

```java
new SpringApplicationBuilder()
    .showBanner(false)
    .sources(Parent.class)
    .child(Application.class)
    .run(args); 
```

**注**：创建 ApplicationContext 层次时有些限制，比如，Web 组件(components)必须包含在子上下文(child context)中，且相同的 Environment 即用于父上下文也用于子上下文中。具体参考[SpringApplicationBuilder javadoc](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/builder/SpringApplicationBuilder.html)

# 22.4\. Application 事件和监听器

### 22.4\. Application 事件和监听器

除了常见的 Spring 框架事件，比如[ContextRefreshedEvent](http://docs.spring.io/spring/docs/4.1.4.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)，一个 SpringApplication 也发送一些额外的应用事件。一些事件实际上是在 ApplicationContext 被创建前触发的。

你可以使用多种方式注册事件监听器，最普通的是使用 SpringApplication.addListeners(…)方法。在你的应用运行时，应用事件会以下面的次序发送：

1.  在运行开始，但除了监听器注册和初始化以外的任何处理之前，会发送一个 ApplicationStartedEvent。
2.  在 Environment 将被用于已知的上下文，但在上下文被创建前，会发送一个 ApplicationEnvironmentPreparedEvent。
3.  在 refresh 开始前，但在 bean 定义已被加载后，会发送一个 ApplicationPreparedEvent。
4.  启动过程中如果出现异常，会发送一个 ApplicationFailedEvent。

**注**：你通常不需要使用应用程序事件，但知道它们的存在会很方便（在某些场合可能会使用到）。在 Spring 内部，Spring Boot 使用事件处理各种各样的任务。

# 22.5\. Web 环境

### 22.5\. Web 环境

一个 SpringApplication 将尝试为你创建正确类型的 ApplicationContext。在默认情况下，使用 AnnotationConfigApplicationContext 或 AnnotationConfigEmbeddedWebApplicationContext 取决于你正在开发的是否是 web 应用。

用于确定一个 web 环境的算法相当简单（基于是否存在某些类）。如果需要覆盖默认行为，你可以使用 setWebEnvironment(boolean webEnvironment)。通过调用 setApplicationContextClass(…)，你可以完全控制 ApplicationContext 的类型。

**注**：当 JUnit 测试里使用 SpringApplication 时，调用 setWebEnvironment(false)是可取的。

# 22.6\. 命令行启动器

### 22.6\. 命令行启动器

如果你想获取原始的命令行参数，或一旦 SpringApplication 启动，你需要运行一些特定的代码，你可以实现 CommandLineRunner 接口。在所有实现该接口的 Spring beans 上将调用 run(String… args)方法。

```java
import org.springframework.boot.*
import org.springframework.stereotype.*

@Component
public class MyBean implements CommandLineRunner {
    public void run(String... args) {
        // Do something...
    }
} 
```

如果一些 CommandLineRunner beans 被定义必须以特定的次序调用，你可以额外实现 org.springframework.core.Ordered 接口或使用 org.springframework.core.annotation.Order 注解。

# 22.7\. Application 退出

### 22.7\. Application 退出

每个 SpringApplication 在退出时为了确保 ApplicationContext 被优雅的关闭，将会注册一个 JVM 的 shutdown 钩子。所有标准的 Spring 生命周期回调（比如，DisposableBean 接口或@PreDestroy 注解）都能使用。

此外，如果 beans 想在应用结束时返回一个特定的退出码（exit code），可以实现 org.springframework.boot.ExitCodeGenerator 接口。

# 23.外化配置

### 23.外化配置

Spring Boot 允许外化（externalize）你的配置，这样你能够在不同的环境下使用相同的代码。你可以使用 properties 文件，YAML 文件，环境变量和命令行参数来外化配置。使用@Value 注解，可以直接将属性值注入到你的 beans 中，并通过 Spring 的 Environment 抽象或绑定到结构化对象来访问。

Spring Boot 使用一个非常特别的 PropertySource 次序来允许对值进行合理的覆盖，需要以下面的次序考虑属性：

1.  命令行参数
2.  来自于 java:comp/env 的 JNDI 属性
3.  Java 系统属性（System.getProperties()）
4.  操作系统环境变量
5.  只有在 random.*里包含的属性会产生一个 RandomValuePropertySource
6.  在打包的 jar 外的应用程序配置文件（application.properties，包含 YAML 和 profile 变量）
7.  在打包的 jar 内的应用程序配置文件（application.properties，包含 YAML 和 profile 变量）
8.  在@Configuration 类上的@PropertySource 注解
9.  默认属性（使用 SpringApplication.setDefaultProperties 指定）

下面是一个具体的示例（假设你开发一个使用 name 属性的@Component）：

```java
import org.springframework.stereotype.*
import org.springframework.beans.factory.annotation.*

@Component
public class MyBean {
    @Value("${name}")
    private String name;
    // ...
} 
```

你可以将一个 application.properties 文件捆绑到 jar 内，用来提供一个合理的默认 name 属性值。当运行在生产环境时，可以在 jar 外提供一个 application.properties 文件来覆盖 name 属性。对于一次性的测试，你可以使用特定的命令行开关启动（比如，java -jar app.jar --name="Spring"）。

# 23.1\. 配置随机值

### 23.1\. 配置随机值

RandomValuePropertySource 在注入随机值（比如，密钥或测试用例）时很有用。它能产生整数，longs 或字符串，比如：

```java
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]} 
```

random.int*语法是 OPEN value (,max) CLOSE，此处 OPEN，CLOSE 可以是任何字符，并且 value，max 是整数。如果提供 max，那么 value 是最小的值，max 是最大的值（不包含在内）。

# 23.2\. 访问命令行属性

### 23.2\. 访问命令行属性

默认情况下，SpringApplication 将任何可选的命令行参数（以'--'开头，比如，--server.port=9000）转化为 property，并将其添加到 Spring Environment 中。如上所述，命令行属性总是优先于其他属性源。

如果你不想将命令行属性添加到 Environment 里，你可以使用 SpringApplication.setAddCommandLineProperties(false)来禁止它们。

# 23.3\. Application 属性文件

### 23.3\. Application 属性文件

SpringApplication 将从以下位置加载 application.properties 文件，并把它们添加到 Spring Environment 中：

1.  当前目录下的一个/config 子目录
2.  当前目录
3.  一个 classpath 下的/config 包
4.  classpath 根路径（root）

这个列表是按优先级排序的（列表中位置高的将覆盖位置低的）。

**注**：你可以使用 YAML（'.yml'）文件替代'.properties'。

如果不喜欢将 application.properties 作为配置文件名，你可以通过指定 spring.config.name 环境属性来切换其他的名称。你也可以使用 spring.config.location 环境属性来引用一个明确的路径（目录位置或文件路径列表以逗号分割）。

```java
$ java -jar myproject.jar --spring.config.name=myproject
//or
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties 
```

如果 spring.config.location 包含目录（相对于文件），那它们应该以/结尾（在加载前，spring.config.name 产生的名称将被追加到后面）。不管 spring.config.location 是什么值，默认的搜索路径 classpath:,classpath:/config,file:,file:config/总会被使用。以这种方式，你可以在 application.properties 中为应用设置默认值，然后在运行的时候使用不同的文件覆盖它，同时保留默认配置。

**注**：如果你使用环境变量而不是系统配置，大多数操作系统不允许以句号分割（period-separated）的 key 名称，但你可以使用下划线（underscores）代替（比如，使用 SPRING_CONFIG_NAME 代替 spring.config.name）。如果你的应用运行在一个容器中，那么 JNDI 属性（java:comp/env）或 servlet 上下文初始化参数可以用来取代环境变量或系统属性，当然也可以使用环境变量或系统属性。

# 23.4\. 特定的 Profile 属性

### 23.4\. 特定的 Profile 属性

除了 application.properties 文件，特定配置属性也能通过命令惯例 application-{profile}.properties 来定义。特定 Profile 属性从跟标准 application.properties 相同的路径加载，并且特定 profile 文件会覆盖默认的配置。

# 23.5\. 属性占位符

### 23.5\. 属性占位符

当 application.properties 里的值被使用时，它们会被存在的 Environment 过滤，所以你能够引用先前定义的值（比如，系统属性）。

```java
app.name=MyApp
app.description=${app.name} is a Spring Boot application 
```

**注**：你也能使用相应的技巧为存在的 Spring Boot 属性创建'短'变量，具体参考 Section 63.3, “Use ‘short’ command line arguments”。

# 23.6\. 使用 YAML 代替 Properties

### 23.6\. 使用 YAML 代替 Properties

[YAML](http://yaml.org/)是 JSON 的一个超集，也是一种方便的定义层次配置数据的格式。无论你何时将[SnakeYAML](http://code.google.com/p/snakeyaml/) 库放到 classpath 下，SpringApplication 类都会自动支持 YAML 作为 properties 的替换。

**注**：如果你使用'starter POMs'，spring-boot-starter 会自动提供 SnakeYAML。

# 23.6.1\. 加载 YAML

### 23.6.1\. 加载 YAML

Spring 框架提供两个便利的类用于加载 YAML 文档，YamlPropertiesFactoryBean 会将 YAML 作为 Properties 来加载，YamlMapFactoryBean 会将 YAML 作为 Map 来加载。

示例：

```java
environments:
    dev:
        url: http://dev.bar.com
        name: Developer Setup
    prod:
        url: http://foo.bar.com
        name: My Cool App 
```

上面的 YAML 文档会被转化到下面的属性中：

```java
environments.dev.url=http://dev.bar.com
environments.dev.name=Developer Setup
environments.prod.url=http://foo.bar.com
environments.prod.name=My Cool App 
```

YAML 列表被表示成使用[index]间接引用作为属性 keys 的形式，例如下面的 YAML：

```java
my:
   servers:
       - dev.bar.com
       - foo.bar.com 
```

将会转化到下面的属性中:

```java
my.servers[0]=dev.bar.com
my.servers[1]=foo.bar.com 
```

使用 Spring DataBinder 工具绑定那样的属性（这是@ConfigurationProperties 做的事），你需要确定目标 bean 中有个 java.util.List 或 Set 类型的属性，并且需要提供一个 setter 或使用可变的值初始化它，比如，下面的代码将绑定上面的属性：

```java
@ConfigurationProperties(prefix="my")
public class Config {
    private List<String> servers = new ArrayList<String>();
    public List<String> getServers() {
        return this.servers;
    }
} 
```

# 23.6.2\. 在 Spring 环境中使用 YAML 暴露属性

### 23.6.2\. 在 Spring 环境中使用 YAML 暴露属性

YamlPropertySourceLoader 类能够用于将 YAML 作为一个 PropertySource 导出到 Sprig Environment。这允许你使用熟悉的@Value 注解和占位符语法访问 YAML 属性。

# 23.6.3\. Multi-profile YAML 文档

### 23.6.3\. Multi-profile YAML 文档

你可以在单个文件中定义多个特定配置（profile-specific）的 YAML 文档，并通过一个 spring.profiles key 标示应用的文档。例如：

```java
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production
server:
    address: 192.168.1.120 
```

在上面的例子中，如果 development 配置被激活，那 server.address 属性将是 127.0.0.1。如果 development 和 production 配置（profiles）没有启用，则该属性的值将是 192.168.1.100。

# 23.6.4\. YAML 缺点

### 23.6.4\. YAML 缺点

YAML 文件不能通过@PropertySource 注解加载。所以，在这种情况下，如果需要使用@PropertySource 注解的方式加载值，那就要使用 properties 文件。

# 23.7\. 类型安全的配置属性

### 23.7\. 类型安全的配置属性

使用@Value("${property}")注解注入配置属性有时可能比较笨重，特别是需要使用多个 properties 或你的数据本身有层次结构。为了控制和校验你的应用配置，Spring Boot 提供一个允许强类型 beans 的替代方法来使用 properties。

示例：

```java
@Component
@ConfigurationProperties(prefix="connection")
public class ConnectionSettings {
    private String username;
    private InetAddress remoteAddress;
    // ... getters and setters
} 
```

当@EnableConfigurationProperties 注解应用到你的@Configuration 时，任何被@ConfigurationProperties 注解的 beans 将自动被 Environment 属性配置。这种风格的配置特别适合与 SpringApplication 的外部 YAML 配置进行配合使用。

```java
# application.yml
connection:
    username: admin
    remoteAddress: 192.168.1.1
# additional configuration as required 
```

为了使用@ConfigurationProperties beans，你可以使用与其他任何 bean 相同的方式注入它们。

```java
@Service
public class MyService {
    @Autowired
    private ConnectionSettings connection;
     //...
    @PostConstruct
    public void openConnection() {
        Server server = new Server();
        this.connection.configure(server);
    }
} 
```

你可以通过在@EnableConfigurationProperties 注解中直接简单的列出属性类来快捷的注册@ConfigurationProperties bean 的定义。

```java
@Configuration
@EnableConfigurationProperties(ConnectionSettings.class)
public class MyConfiguration {
} 
```

**注**：使用@ConfigurationProperties 能够产生可被 IDEs 使用的元数据文件。具体参考 Appendix B, Configuration meta-data。

# 23.7.1\. 第三方配置

### 23.7.1\. 第三方配置

正如使用@ConfigurationProperties 注解一个类，你也可以在@Bean 方法上使用它。当你需要绑定属性到不受你控制的第三方组件时，这种方式非常有用。

为了从 Environment 属性配置一个 bean，将@ConfigurationProperties 添加到它的 bean 注册过程：

```java
@ConfigurationProperties(prefix = "foo")
@Bean
public FooComponent fooComponent() {
    ...
} 
```

和上面 ConnectionSettings 的示例方式相同，任何以 foo 为前缀的属性定义都会被映射到 FooComponent 上。

# 23.7.2\. 松散的绑定（Relaxed binding）

### 23.7.2\. 松散的绑定（Relaxed binding）

Spring Boot 使用一些宽松的规则用于绑定 Environment 属性到@ConfigurationProperties beans，所以 Environment 属性名和 bean 属性名不需要精确匹配。常见的示例中有用的包括虚线分割（比如，context--path 绑定到 contextPath）和将环境属性转为大写字母（比如，PORT 绑定 port）。

示例：

```java
@Component
@ConfigurationProperties(prefix="person")
public class ConnectionSettings {
    private String firstName;
} 
```

下面的属性名都能用于上面的@ConfigurationProperties 类：

| 属性 | 说明 |
| --- | --- |
| person.firstName | 标准驼峰规则 |
| person.first-name | 虚线表示，推荐用于.properties 和.yml 文件中 |
| PERSON_FIRST_NAME | 大写形式，使用系统环境变量时推荐 |

Spring 会尝试强制外部的应用属性在绑定到@ConfigurationProperties beans 时类型是正确的。如果需要自定义类型转换，你可以提供一个 ConversionService bean（bean id 为 conversionService）或自定义属性编辑器（通过一个 CustomEditorConfigurer bean）。

# 23.7.3\. @ConfigurationProperties 校验

### 23.7.3\. @ConfigurationProperties 校验

Spring Boot 将尝试校验外部的配置，默认使用 JSR-303（如果在 classpath 路径中）。你可以轻松的为你的@ConfigurationProperties 类添加 JSR-303 javax.validation 约束注解：

```java
@Component
@ConfigurationProperties(prefix="connection")
public class ConnectionSettings {
    @NotNull
    private InetAddress remoteAddress;
    // ... getters and setters
} 
```

你也可以通过创建一个叫做 configurationPropertiesValidator 的 bean 来添加自定义的 Spring Validator。

**注**：spring-boot-actuator 模块包含一个暴露所有@ConfigurationProperties beans 的端点。简单地将你的 web 浏览器指向/configprops 或使用等效的 JMX 端点。具体参考 Production ready features。

# 24\. Profiles

### 24\. Profiles

Spring Profiles 提供了一种隔离应用程序配置的方式，并让这些配置只能在特定的环境下生效。任何@Component 或@Configuration 都能被@Profile 标记，从而限制加载它的时机。

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {
    // ...
} 
```

以正常的 Spring 方式，你可以使用一个 spring.profiles.active 的 Environment 属性来指定哪个配置生效。你可以使用平常的任何方式来指定该属性，例如，可以将它包含到你的 application.properties 中：

```java
spring.profiles.active=dev,hsqldb 
```

或使用命令行开关：

```java
--spring.profiles.active=dev,hsqldb 
```

# 24.1\. 添加激活的配置(profiles)

### 24.1\. 添加激活的配置(profiles)

spring.profiles.active 属性和其他属性一样都遵循相同的排列规则，最高的 PropertySource 获胜。也就是说，你可以在 application.properties 中指定生效的配置，然后使用命令行开关替换它们。

有时，将特定的配置属性添加到生效的配置中而不是替换它们是有用的。spring.profiles.include 属性可以用来无条件的添加生效的配置。SpringApplication 的入口点也提供了一个用于设置额外配置的 Java API（比如，在那些通过 spring.profiles.active 属性生效的配置之上）：参考 setAdditionalProfiles()方法。

示例：当一个应用使用下面的属性，并用`--spring.profiles.active=prod`开关运行，那 proddb 和 prodmq 配置也会生效：

```java
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include: proddb,prodmq 
```

**注**：spring.profiles 属性可以定义到一个 YAML 文档中，用于决定什么时候该文档被包含进配置中。具体参考 Section 63.6, “Change configuration depending on the environment”

# 24.2.以编程方式设置 profiles

### 24.2.以编程方式设置 profiles

在应用运行前，你可以通过调用 SpringApplication.setAdditionalProfiles(…)方法，以编程的方式设置生效的配置。使用 Spring 的 ConfigurableEnvironment 接口激动配置也是可行的。

# 24.3\. Profile 特定配置文件

### 24.3\. Profile 特定配置文件

application.properties（或 application.yml）和通过@ConfigurationProperties 引用的文件这两种配置特定变种都被当作文件来加载的，具体参考 Section 23.3, “Profile specific properties”。

# 25\. 日志

### 25\. 日志

Spring Boot 内部日志系统使用的是[Commons Logging](http://commons.apache.org/logging)，但开放底层的日志实现。默认为会[Java Util Logging](http://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html), [Log4J](http://logging.apache.org/log4j/), [Log4J2](http://logging.apache.org/log4j/2.x/)和[Logback](http://logback.qos.ch/)提供配置。每种情况下都会预先配置使用控制台输出，也可以使用可选的文件输出。

默认情况下，如果你使用'Starter POMs'，那么就会使用 Logback 记录日志。为了确保那些使用 Java Util Logging, Commons Logging, Log4J 或 SLF4J 的依赖库能够正常工作，正确的 Logback 路由也被包含进来。

**注**：如果上面的列表看起来令人困惑，不要担心，Java 有很多可用的日志框架。通常，你不需要改变日志依赖，Spring Boot 默认的就能很好的工作。

# 25.1\. 日志格式

### 25.1\. 日志格式

Spring Boot 默认的日志输出格式如下：

```java
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*] 
```

输出的节点（items）如下：

1.  日期和时间 - 精确到毫秒，且易于排序。
2.  日志级别 - ERROR, WARN, INFO, DEBUG 或 TRACE。
3.  Process ID。
4.  一个用于区分实际日志信息开头的---分隔符。
5.  线程名 - 包括在方括号中（控制台输出可能会被截断）。
6.  日志名 - 通常是源 class 的类名（缩写）。
7.  日志信息。

# 25.2\. 控制台输出

### 25.2\. 控制台输出

默认的日志配置会在写日志消息时将它们回显到控制台。默认，ERROR, WARN 和 INFO 级别的消息会被记录。可以在启动应用时，通过`--debug`标识开启控制台的 DEBUG 级别日志记录。

```java
$ java -jar myapp.jar --debug 
```

如果你的终端支持 ANSI，为了增加可读性将会使用彩色的日志输出。你可以设置`spring.output.ansi.enabled`为一个[支持的值](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html)来覆盖自动检测。

# 25.3\. 文件输出

### 25.3\. 文件输出

默认情况下，Spring Boot 只会将日志记录到控制台而不会写进日志文件。如果除了输出到控制台你还想写入到日志文件，那你需要设置`logging.file`或`logging.path`属性（例如在你的 application.properties 中）。

下表显示如何组合使用`logging.*`：

| logging.file | logging.path | 示例 | 描述 |
| --- | --- | --- | --- |
| (none) | (none) |  | 只记录到控制台 |
| Specific file | (none) | my.log | 写到特定的日志文件里，名称可以是一个精确的位置或相对于当前目录 |
| (none) | Specific folder | /var/log | 写到特定文件夹下的 spring.log 里，名称可以是一个精确的位置或相对于当前目录 |

日志文件每达到 10M 就会被轮换（分割），和控制台一样，默认记录 ERROR, WARN 和 INFO 级别的信息。

# 25.4\. 日志级别

### 25.4\. 日志级别

所有支持的日志系统在 Spring 的 Environment（例如在 application.properties 里）都有通过'logging.level.*=LEVEL'（'LEVEL'是 TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF 中的一个）设置的日志级别。

示例：application.properties

```java
logging.level.org.springframework.web: DEBUG
logging.level.org.hibernate: ERROR 
```

# 25.5\. 自定义日志配置

### 25.5\. 自定义日志配置

通过将适当的库添加到 classpath，可以激活各种日志系统。然后在 classpath 的根目录(root)或通过 Spring Environment 的`logging.config`属性指定的位置提供一个合适的配置文件来达到进一步的定制（注意由于日志是在 ApplicationContext 被创建之前初始化的，所以不可能在 Spring 的@Configuration 文件中，通过@PropertySources 控制日志。系统属性和平常的 Spring Boot 外部配置文件能正常工作）。

根据你的日志系统，下面的文件会被加载：

| 日志系统 | 定制 |
| --- | --- |
| Logback | logback.xml |
| Log4j | log4j.properties 或 log4j.xml |
| Log4j2 | log4j2.xml |
| JDK (Java Util Logging) | logging.properties |

为了帮助定制一些其他的属性，从 Spring 的 Envrionment 转换到系统属性：

| Spring Environment | System Property | 评价 |
| --- | --- | --- |
| logging.file | LOG_FILE | 如果定义，在默认的日志配置中使用 |
| logging.path | LOG_PATH | 如果定义，在默认的日志配置中使用 |
| PID | PID | 当前的处理进程(process)ID（如果能够被发现且还没有作为操作系统环境变量被定义） |

所有支持的日志系统在解析它们的配置文件时都能查询系统属性。具体可以参考 spring-boot.jar 中的默认配置。

**注**：在运行可执行的 jar 时，Java Util Logging 有类加载问题，我们建议你尽可能避免使用它。

# 26\. 开发 Web 应用

### 26\. 开发 Web 应用

Spring Boot 非常适合开发 web 应用程序。你可以使用内嵌的 Tomcat，Jetty 或 Undertow 轻轻松松地创建一个 HTTP 服务器。大多数的 web 应用都使用 spring-boot-starter-web 模块进行快速搭建和运行。

# 26.1\. Spring Web MVC 框架

### 26.1\. Spring Web MVC 框架

Spring Web MVC 框架（通常简称为"Spring MVC"）是一个富"模型，视图，控制器"的 web 框架。 Spring MVC 允许你创建特定的@Controller 或@RestController beans 来处理传入的 HTTP 请求。 使用@RequestMapping 注解可以将控制器中的方法映射到相应的 HTTP 请求。

示例：

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
        // ...
    }
} 
```

# 26.1.1\. Spring MVC 自动配置

### 26.1.1\. Spring MVC 自动配置

Spring Boot 为 Spring MVC 提供适用于多数应用的自动配置功能。在 Spring 默认基础上，自动配置添加了以下特性：

1.  引入 ContentNegotiatingViewResolver 和 BeanNameViewResolver beans。
2.  对静态资源的支持，包括对 WebJars 的支持。
3.  自动注册 Converter，GenericConverter，Formatter beans。
4.  对 HttpMessageConverters 的支持。
5.  自动注册 MessageCodeResolver。
6.  对静态 index.html 的支持。
7.  对自定义 Favicon 的支持。

如果想全面控制 Spring MVC，你可以添加自己的@Configuration，并使用@EnableWebMvc 对其注解。如果想保留 Spring Boot MVC 的特性，并只是添加其他的[MVC 配置](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle#mvc)(拦截器，formatters，视图控制器等)，你可以添加自己的 WebMvcConfigurerAdapter 类型的@Bean（不使用@EnableWebMvc 注解）。

# 26.1.2\. HttpMessageConverters

### 26.1.2\. HttpMessageConverters

Spring MVC 使用 HttpMessageConverter 接口转换 HTTP 请求和响应。合理的缺省值被包含的恰到好处（out of the box），例如对象可以自动转换为 JSON（使用 Jackson 库）或 XML（如果 Jackson XML 扩展可用则使用它，否则使用 JAXB）。字符串默认使用 UTF-8 编码。

如果需要添加或自定义转换器，你可以使用 Spring Boot 的 HttpMessageConverters 类：

```java
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration
public class MyConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }
} 
```

任何在上下文中出现的 HttpMessageConverter bean 将会添加到 converters 列表，你可以通过这种方式覆盖默认的转换器（converters）。

# 26.1.3\. MessageCodesResolver

### 26.1.3\. MessageCodesResolver

Spring MVC 有一个策略，用于从绑定的 errors 产生用来渲染错误信息的错误码：MessageCodesResolver。如果设置`spring.mvc.message-codes-resolver.format`属性为`PREFIX_ERROR_CODE`或`POSTFIX_ERROR_CODE`（具体查看`DefaultMessageCodesResolver.Format`枚举值），Spring Boot 会为你创建一个 MessageCodesResolver。

# 26.1.4\. 静态内容

### 26.1.4\. 静态内容

默认情况下，Spring Boot 从 classpath 下一个叫/static（/public，/resources 或/META-INF/resources）的文件夹或从 ServletContext 根目录提供静态内容。这使用了 Spring MVC 的 ResourceHttpRequestHandler，所以你可以通过添加自己的 WebMvcConfigurerAdapter 并覆写 addResourceHandlers 方法来改变这个行为（加载静态文件）。

在一个单独的 web 应用中，容器默认的 servlet 是开启的，如果 Spring 决定不处理某些请求，默认的 servlet 作为一个回退（降级）将从 ServletContext 根目录加载内容。大多数时候，这不会发生（除非你修改默认的 MVC 配置），因为 Spring 总能够通过 DispatcherServlet 处理请求。

此外，上述标准的静态资源位置有个例外情况是[Webjars 内容](http://www.webjars.org/)。任何在/webjars/**路径下的资源都将从 jar 文件中提供，只要它们以 Webjars 的格式打包。

**注**：如果你的应用将被打包成 jar，那就不要使用 src/main/webapp 文件夹。尽管该文件夹是一个共同的标准，但它仅在打包成 war 的情况下起作用，并且如果产生一个 jar，多数构建工具都会静悄悄的忽略它。

# 26.1.5\. 模板引擎

### 26.1.5\. 模板引擎

正如 REST web 服务，你也可以使用 Spring MVC 提供动态 HTML 内容。Spring MVC 支持各种各样的模板技术，包括 Velocity, FreeMarker 和 JSPs。很多其他的模板引擎也提供它们自己的 Spring MVC 集成。

Spring Boot 为以下的模板引擎提供自动配置支持：

1.  [FreeMarker](http://freemarker.org/docs/)
2.  [Groovy](http://beta.groovy-lang.org/docs/groovy-2.3.0/html/documentation/markup-template-engine.html)
3.  [Thymeleaf](http://www.thymeleaf.org/)
4.  [Velocity](http://velocity.apache.org/)

**注**：如果可能的话，应该忽略 JSPs，因为在内嵌的 servlet 容器使用它们时存在一些已知的限制。

当你使用这些引擎的任何一种，并采用默认的配置，你的模板将会从 src/main/resources/templates 目录下自动加载。

**注**：IntelliJ IDEA 根据你运行应用的方式会对 classpath 进行不同的整理。在 IDE 里通过 main 方法运行你的应用跟从 Maven 或 Gradle 或打包好的 jar 中运行相比会导致不同的顺序。这可能导致 Spring Boot 不能从 classpath 下成功地找到模板。如果遇到这个问题，你可以在 IDE 里重新对 classpath 进行排序，将模块的类和资源放到第一位。或者，你可以配置模块的前缀为 classpath*:/templates/，这样会查找 classpath 下的所有模板目录。

# 26.1.6\. 错误处理

### 26.1.6\. 错误处理

Spring Boot 默认提供一个/error 映射用来以合适的方式处理所有的错误，并且它在 servlet 容器中注册了一个全局的 错误页面。对于机器客户端（相对于浏览器而言，浏览器偏重于人的行为），它会产生一个具有详细错误，HTTP 状态，异常信息的 JSON 响应。对于浏览器客户端，它会产生一个白色标签样式（whitelabel）的错误视图，该视图将以 HTML 格式显示同样的数据（可以添加一个解析为 erro 的 View 来自定义它）。为了完全替换默认的行为，你可以实现 ErrorController，并注册一个该类型的 bean 定义，或简单地添加一个 ErrorAttributes 类型的 bean 以使用现存的机制，只是替换显示的内容。

如果在某些条件下需要比较多的错误页面，内嵌的 servlet 容器提供了一个统一的 Java DSL（领域特定语言）来自定义错误处理。 示例：

```java
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer(){
    return new MyCustomizer();
}

// ...
private static class MyCustomizer implements EmbeddedServletContainerCustomizer {
    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }
} 
```

你也可以使用常规的 Spring MVC 特性来处理错误，比如[@ExceptionHandler 方法](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#mvc-exceptionhandlers)和[@ControllerAdvice](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-controller-advice)。ErrorController 将会捡起任何没有处理的异常。

N.B. 如果你为一个路径注册一个 ErrorPage，最终被一个过滤器（Filter）处理（对于一些非 Spring web 框架，像 Jersey 和 Wicket 这很常见），然后过滤器需要显式注册为一个 ERROR 分发器（dispatcher）。

```java
@Bean
public FilterRegistrationBean myFilter() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new MyFilter());
    ...
    registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
    return registration;
} 
```

**注**：默认的 FilterRegistrationBean 没有包含 ERROR 分发器类型。

# 26.1.7\. Spring HATEOAS

### 26.1.7\. Spring HATEOAS

如果你正在开发一个使用超媒体的 RESTful API，Spring Boot 将为 Spring HATEOAS 提供自动配置，这在多数应用中都工作良好。自动配置替换了对使用@EnableHypermediaSupport 的需求，并注册一定数量的 beans 来简化构建基于超媒体的应用，这些 beans 包括一个 LinkDiscoverer 和配置好的用于将响应正确编排为想要的表示的 ObjectMapper。ObjectMapper 可以根据 spring.jackson.*属性或一个存在的 Jackson2ObjectMapperBuilder bean 进行自定义。

通过使用@EnableHypermediaSupport，你可以控制 Spring HATEOAS 的配置。注意这会禁用上述的对 ObjectMapper 的自定义。

# 26.2\. JAX-RS 和 Jersey

### 26.2\. JAX-RS 和 Jersey

如果喜欢 JAX-RS 为 REST 端点提供的编程模型，你可以使用可用的实现替代 Spring MVC。如果在你的应用上下文中将 Jersey 1.x 和 Apache Celtix 的 Servlet 或 Filter 注册为一个@Bean，那它们工作的相当好。Jersey 2.x 有一些原生的 Spring 支持，所以我们会在 Spring Boot 为它提供自动配置支持，连同一个启动器（starter）。

想要开始使用 Jersey 2.x 只需要加入 spring-boot-starter-jersey 依赖，然后你需要一个 ResourceConfig 类型的@Bean，用于注册所有的端点（endpoints）。

```java
@Component
public class JerseyConfig extends ResourceConfig {
    public JerseyConfig() {
        register(Endpoint.class);
    }
} 
```

所有注册的端点都应该被@Components 和 HTTP 资源 annotations（比如@GET）注解。

```java
@Component
@Path("/hello")
public class Endpoint {
    @GET
    public String message() {
        return "Hello";
    }
} 
```

由于 Endpoint 是一个 Spring 组件（@Component），所以它的生命周期受 Spring 管理，并且你可以使用@Autowired 添加依赖及使用@Value 注入外部配置。Jersey servlet 将被注册，并默认映射到/*。你可以将@ApplicationPath 添加到 ResourceConfig 来改变该映射。

默认情况下，Jersey 将在一个 ServletRegistrationBean 类型的@Bean 中被设置成名称为 jerseyServletRegistration 的 Servlet。通过创建自己的相同名称的 bean，你可以禁止或覆盖这个 bean。你也可以通过设置`spring.jersey.type=filter`来使用一个 Filter 代替 Servlet（在这种情况下，被覆盖或替换的@Bean 是 jerseyFilterRegistration）。该 servlet 有@Order 属性，你可以通过`spring.jersey.filter.order`进行设置。不管是 Servlet 还是 Filter 注册都可以使用 spring.jersey.init.*定义一个属性集合作为初始化参数传递过去。

这里有一个[Jersey 示例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-jersey)，你可以查看如何设置相关事项。

# 26.3\. 内嵌 servlet 容器支持

### 26.3\. 内嵌 servlet 容器支持

Spring Boot 支持内嵌的 Tomcat, Jetty 和 Undertow 服务器。多数开发者只需要使用合适的'Starter POM'来获取一个完全配置好的实例即可。默认情况下，内嵌的服务器会在 8080 端口监听 HTTP 请求。

# 26.3.1\. Servlets 和 Filters

### 26.3.1\. Servlets 和 Filters

当使用内嵌的 servlet 容器时，你可以直接将 servlet 和 filter 注册为 Spring 的 beans。在配置期间，如果你想引用来自 application.properties 的值，这是非常方便的。默认情况下，如果上下文只包含单一的 Servlet，那它将被映射到根路径（/）。在多 Servlet beans 的情况下，bean 的名称将被用作路径的前缀。过滤器会被映射到/*。

如果基于约定（convention-based）的映射不够灵活，你可以使用 ServletRegistrationBean 和 FilterRegistrationBean 类实现完全的控制。如果你的 bean 实现了 ServletContextInitializer 接口，也可以直接注册它们。

# 26.3.2\. EmbeddedWebApplicationContext

### 26.3.2\. EmbeddedWebApplicationContext

Spring Boot 底层使用了一个新的 ApplicationContext 类型，用于对内嵌 servlet 容器的支持。EmbeddedWebApplicationContext 是一个特殊类型的 WebApplicationContext，它通过搜索一个单一的 EmbeddedServletContainerFactory bean 来启动自己。通常，TomcatEmbeddedServletContainerFactory，JettyEmbeddedServletContainerFactory 或 UndertowEmbeddedServletContainerFactory 将被自动配置。

**注**：你通常不需要知道这些实现类。大多数应用将被自动配置，并根据你的行为创建合适的 ApplicationContext 和 EmbeddedServletContainerFactory。

# 26.3.3\. 自定义内嵌 servlet 容器

### 26.3.3\. 自定义内嵌 servlet 容器

常见的 Servlet 容器设置可以通过 Spring Environment 属性进行配置。通常，你会把这些属性定义到 application.properties 文件中。 常见的服务器设置包括：

1.  server.port - 进来的 HTTP 请求的监听端口号
2.  server.address - 绑定的接口地址
3.  server.sessionTimeout - session 超时时间

具体参考[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)。

*   编程方式的自定义

如果需要以编程的方式配置内嵌的 servlet 容器，你可以注册一个实现 EmbeddedServletContainerCustomizer 接口的 Spring bean。EmbeddedServletContainerCustomizer 提供对 ConfigurableEmbeddedServletContainer 的访问，ConfigurableEmbeddedServletContainer 包含很多自定义的 setter 方法。

```java
import org.springframework.boot.context.embedded.*;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements EmbeddedServletContainerCustomizer {
    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.setPort(9000);
    }
} 
```

*   直接自定义 ConfigurableEmbeddedServletContainer

如果上面的自定义手法过于受限，你可以自己注册 TomcatEmbeddedServletContainerFactory，JettyEmbeddedServletContainerFactory 或 UndertowEmbeddedServletContainerFactory。

```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory factory = new TomcatEmbeddedServletContainerFactory();
    factory.setPort(9000);
    factory.setSessionTimeout(10, TimeUnit.MINUTES);
    factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html");
    return factory;
} 
```

很多可选的配置都提供了 setter 方法，也提供了一些受保护的钩子方法以满足你的某些特殊需求。具体参考相关文档。

# 26.3.4\. JSP 的限制

### 26.3.4\. JSP 的限制

在内嵌的 servlet 容器中运行一个 Spring Boot 应用时（并打包成一个可执行的存档 archive），容器对 JSP 的支持有一些限制。

1.  tomcat 只支持 war 的打包方式，不支持可执行的 jar。
2.  内嵌的 Jetty 目前不支持 JSPs。
3.  Undertow 不支持 JSPs。

这里有个[JSP 示例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-jsp)，你可以查看如何设置相关事项。

# 27\. 安全

### 27\. 安全

如果 Spring Security 在 classpath 下，那么 web 应用默认对所有的 HTTP 路径（也称为终点，端点，表示 API 的具体网址）使用'basic'认证。为了给 web 应用添加方法级别的保护，你可以添加@EnableGlobalMethodSecurity 并使用想要的设置。其他信息参考[Spring Security Reference](http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle#jc-method)。

默认的 AuthenticationManager 有一个单一的 user（'user'的用户名和随机密码会在应用启动时以 INFO 日志级别打印出来）。如下：

```java
Using default security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35 
```

**注**：如果你对日志配置进行微调，确保`org.springframework.boot.autoconfigure.security`类别能记录 INFO 信息，否则默认的密码不会被打印。

你可以通过提供`security.user.password`改变默认的密码。这些和其他有用的属性通过[SecurityProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/SecurityProperties.java)（以 security 为前缀的属性）被外部化了。

默认的安全配置（security configuration）是在 SecurityAutoConfiguration 和导入的类中实现的（SpringBootWebSecurityConfiguration 用于 web 安全，AuthenticationManagerConfiguration 用于与非 web 应用也相关的认证配置）。你可以添加一个@EnableWebSecurity bean 来彻底关掉 Spring Boot 的默认配置。为了对它进行自定义，你需要使用外部的属性配置和 WebSecurityConfigurerAdapter 类型的 beans（比如，添加基于表单的登陆）。在[Spring Boot 示例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/)里有一些安全相关的应用可以带你体验常见的用例。

在一个 web 应用中你能得到的基本特性如下：

1.  一个使用内存存储的 AuthenticationManager bean 和唯一的 user（查看 SecurityProperties.User 获取 user 的属性）。
2.  忽略（不保护）常见的静态资源路径（`/css/**, /js/**, /images/**`和 `**/favicon.ico`）。
3.  对其他的路径实施 HTTP Basic 安全保护。
4.  安全相关的事件会发布到 Spring 的 ApplicationEventPublisher（成功和失败的认证，拒绝访问）。
5.  Spring Security 提供的常见底层特性（HSTS, XSS, CSRF, 缓存）默认都被开启。

上述所有特性都能打开和关闭，或使用外部的配置进行修改（security.*）。为了覆盖访问规则（access rules）而不改变其他自动配置的特性，你可以添加一个使用@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)注解的 WebSecurityConfigurerAdapter 类型的@Bean。

如果 Actuator 也在使用，你会发现：

1.  即使应用路径不受保护，被管理的路径也会受到保护。
2.  安全相关的事件被转换为 AuditEvents（审计事件），并发布给 AuditService。
3.  默认的用户有 ADMIN 和 USER 的角色。

使用外部属性能够修改 Actuator（执行器）的安全特性（management.security.*）。为了覆盖应用程序的访问规则，你可以添加一个 WebSecurityConfigurerAdapter 类型的@Bean。同时，如果不想覆盖执行器的访问规则，你可以使用@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)注解该 bean，否则使用@Order(ManagementServerProperties.ACCESS_OVERRIDE_ORDER)注解该 bean。

# 28\. 使用 SQL 数据库

### 28\. 使用 SQL 数据库

Spring 框架为使用 SQL 数据库提供了广泛的支持。从使用 JdbcTemplate 直接访问 JDBC 到完全的对象关系映射技术，比如 Hibernate。Spring Data 提供一个额外的功能，直接从接口创建 Repository 实现，并使用约定从你的方法名生成查询。

# 28.1\. 配置 DataSource

### 28.1\. 配置 DataSource

Java 的 javax.sql.DataSource 接口提供了一个标准的使用数据库连接的方法。传统做法是，一个 DataSource 使用一个 URL 连同相应的证书去初始化一个数据库连接。

# 28.1.1\. 对内嵌数据库的支持

### 28.1.1\. 对内嵌数据库的支持

开发应用时使用内存数据库是很实用的。显而易见地，内存数据库不需要提供持久化存储。你不需要在应用启动时填充数据库，也不需要在应用结束时丢弃数据。

Spring Boot 可以自动配置的内嵌数据库包括[H2](http://www.h2database.com/), [HSQL](http://hsqldb.org/)和[Derby](http://db.apache.org/derby/)。你不需要提供任何连接 URLs，只需要简单的添加你想使用的内嵌数据库依赖。

示例：典型的 POM 依赖如下：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency> 
```

**注**：对于自动配置的内嵌数据库，你需要依赖 spring-jdbc。在示例中，它通过`spring-boot-starter-data-jpa`被传递地拉过来了。

# 28.1.2\. 连接到一个生产环境数据库

### 28.1.2\. 连接到一个生产环境数据库

在生产环境中，数据库连接可以使用 DataSource 池进行自动配置。下面是选取一个特定实现的算法：

*   由于 Tomcat 数据源连接池的性能和并发，在 tomcat 可用时，我们总是优先使用它。
*   如果 HikariCP 可用，我们将使用它。
*   如果 Commons DBCP 可用，我们将使用它，但在生产环境不推荐使用它。
*   最后，如果 Commons DBCP2 可用，我们将使用它。

如果你使用 spring-boot-starter-jdbc 或 spring-boot-starter-data-jpa 'starter POMs'，你将会自动获取对 tomcat-jdbc 的依赖。

**注**：其他的连接池可以手动配置。如果你定义自己的 DataSource bean，自动配置不会发生。

DataSource 配置通过外部配置文件的 spring.datasource.*属性控制。示例中，你可能会在 application.properties 中声明下面的片段：

```java
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver 
```

其他可选的配置可以查看[DataSourceProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java)。同时注意你可以通过 spring.datasource.*配置任何 DataSource 实现相关的特定属性：具体参考你使用的连接池实现的文档。

**注**：既然 Spring Boot 能够从大多数数据库的 url 上推断出 driver-class-name，那么你就不需要再指定它了。对于一个将要创建的 DataSource 连接池，我们需要能够验证 Driver 是否可用，所以我们会在做任何事情之前检查它。比如，如果你设置 spring.datasource.driverClassName=com.mysql.jdbc.Driver，然后这个类就会被加载。

# 28.1.3\. 连接到一个 JNDI 数据库

### 28.1.3\. 连接到一个 JNDI 数据库

如果正在将 Spring Boot 应用部署到一个应用服务器，你可能想要用应用服务器内建的特性来配置和管理你的 DataSource，并使用 JNDI 访问它。

spring.datasource.jndi-name 属性可以用来替代 spring.datasource.url，spring.datasource.username 和 spring.datasource.password 去从一个特定的 JNDI 路径访问 DataSource。比如，下面 application.properties 中的片段展示了如何获取 JBoss 定义的 DataSource：

```java
spring.datasource.jndi-name=java:jboss/datasources/customers 
```

# 28.2\. 使用 JdbcTemplate

### 28.2\. 使用 JdbcTemplate

Spring 的 JdbcTemplate 和 NamedParameterJdbcTemplate 类是被自动配置的，你可以在自己的 beans 中通过@Autowire 直接注入它们。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    // ...
} 
```

# 28.3\. JPA 和 Spring Data

### 28.3\. JPA 和 Spring Data

Java 持久化 API 是一个允许你将对象映射为关系数据库的标准技术。spring-boot-starter-data-jpa POM 提供了一种快速上手的方式。它提供下列关键的依赖：

*   Hibernate - 一个非常流行的 JPA 实现。
*   Spring Data JPA - 让实现基于 JPA 的 repositories 更容易。
*   Spring ORMs - Spring 框架的核心 ORM 支持。

**注**：我们不想在这涉及太多关于 JPA 或 Spring Data 的细节。你可以参考来自[spring.io](http://spring.io/)的指南[使用 JPA 获取数据](http://spring.io/guides/gs/accessing-data-jpa/)，并阅读[Spring Data JPA](http://projects.spring.io/spring-data-jpa/)和[Hibernate](http://hibernate.org/orm/documentation/)的参考文档。

# 28.3.1\. 实体类

### 28.3.1\. 实体类

传统上，JPA 实体类被定义到一个 persistence.xml 文件中。在 Spring Boot 中，这个文件不是必需的，并被'实体扫描'替代。默认情况下，在你主（main）配置类（被@EnableAutoConfiguration 或@SpringBootApplication 注解的类）下的所有包都将被查找。

任何被@Entity，@Embeddable 或@MappedSuperclass 注解的类都将被考虑。一个普通的实体类看起来像下面这样：

```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String state;

    // ... additional members, often include @OneToMany mappings

    protected City() {
        // no-args constructor required by JPA spec
        // this one is protected since it shouldn't be used directly
    }

    public City(String name, String state) {
        this.name = name;
        this.country = country;
    }

    public String getName() {
        return this.name;
    }

    public String getState() {
        return this.state;
    }
    // ... etc
} 
```

**注**：你可以使用@EntityScan 注解自定义实体扫描路径。具体参考 Section 67.4, “Separate @Entity definitions from Spring configuration”。

# 28.3.2\. Spring Data JPA 仓库

### 28.3.2\. Spring Data JPA 仓库

Spring Data JPA 仓库（repositories）是用来定义访问数据的接口。根据你的方法名，JPA 查询会被自动创建。比如，一个 CityRepository 接口可能声明一个 findAllByState(String state)方法，用来查找给定状态的所有城市。

对于比较复杂的查询，你可以使用 Spring Data 的[Query](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html)来注解你的方法。

Spring Data 仓库通常继承自[Repository](http://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html)或[CrudRepository](http://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)接口。如果你使用自动配置，包括在你的主配置类（被@EnableAutoConfiguration 或@SpringBootApplication 注解的类）的包下的仓库将会被搜索。

下面是一个传统的 Spring Data 仓库：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndCountryAllIgnoringCase(String name, String country);
} 
```

**注**：我们仅仅触及了 Spring Data JPA 的表面。具体查看它的[参考指南](http://projects.spring.io/spring-data-jpa/)。

# 28.3.3\. 创建和删除 JPA 数据库

### 28.3.3\. 创建和删除 JPA 数据库

默认情况下，只有在你使用内嵌数据库（H2, HSQL 或 Derby）时，JPA 数据库才会被自动创建。你可以使用 spring.jpa.*属性显示的设置 JPA。比如，为了创建和删除表你可以将下面的配置添加到 application.properties 中：

```java
spring.jpa.hibernate.ddl-auto=create-drop 
```

**注**：Hibernate 自己内部对创建，删除表支持（如果你恰好记得这回事更好）的属性是 hibernate.hbm2ddl.auto。使用 spring.jpa.properties.*（前缀在被添加到实体管理器之前会被剥离掉），你可以设置 Hibernate 本身的属性，比如 hibernate.hbm2ddl.auto。示例：`spring.jpa.properties.hibernate.globally_quoted_identifiers=true`将传递 hibernate.globally_quoted_identifiers 到 Hibernate 实体管理器。

默认情况下，DDL 执行（或验证）被延迟到 ApplicationContext 启动。这也有一个 spring.jpa.generate-ddl 标识，如果 Hibernate 自动配置被激活，那该标识就不会被使用，因为 ddl-auto 设置粒度更细。

# 29\. 使用 NoSQL 技术

### 29\. 使用 NoSQL 技术

Spring Data 提供其他项目，用来帮你使用各种各样的 NoSQL 技术，包括[MongoDB](http://projects.spring.io/spring-data-mongodb/), [Neo4J](http://projects.spring.io/spring-data-neo4j/), [Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch/), [Solr](http://projects.spring.io/spring-data-solr/), [Redis](http://projects.spring.io/spring-data-redis/), [Gemfire](http://projects.spring.io/spring-data-gemfire/), [Couchbase](http://projects.spring.io/spring-data-couchbase/)和[Cassandra](http://projects.spring.io/spring-data-cassandra/)。Spring Boot 为 Redis, MongoDB, Elasticsearch, Solr 和 Gemfire 提供自动配置。你可以充分利用其他项目，但你需要自己配置它们。具体查看[projects.spring.io/spring-data](http://projects.spring.io/spring-data/)中合适的参考文档。

# 29.1\. Redis

### 29.1\. Redis

[Redis](http://redis.io/)是一个缓存，消息中间件及具有丰富特性的键值存储系统。Spring Boot 为[Jedis](https://github.com/xetorthio/jedis/)客户端库和由[Spring Data Redis](https://github.com/spring-projects/spring-data-redis)提供的基于 Jedis 客户端的抽象提供自动配置。`spring-boot-starter-redis`'Starter POM'为收集依赖提供一种便利的方式。

# 29.1.1\. 连接 Redis

### 29.1.1\. 连接 Redis

你可以注入一个自动配置的 RedisConnectionFactory，StringRedisTemplate 或普通的跟其他 Spring Bean 相同的 RedisTemplate 实例。默认情况下，这个实例将尝试使用 localhost:6379 连接 Redis 服务器。

```java
@Component
public class MyBean {

    private StringRedisTemplate template;

    @Autowired
    public MyBean(StringRedisTemplate template) {
        this.template = template;
    }
    // ...
} 
```

如果你添加一个你自己的任何自动配置类型的@Bean，它将替换默认的（除了 RedisTemplate 的情况，它是根据 bean 的名称'redisTemplate'而不是它的类型进行排除的）。如果在 classpath 路径下存在 commons-pool2，默认你会获得一个连接池工厂。

# 29.2\. MongoDB

### 29.2\. MongoDB

[MongoDB](http://www.mongodb.com/)是一个开源的 NoSQL 文档数据库，它使用一个 JSON 格式的模式（schema）替换了传统的基于表的关系数据。Spring Boot 为使用 MongoDB 提供了很多便利，包括`spring-boot-starter-data-mongodb`'Starter POM'。

# 29.2.1\. 连接 MongoDB 数据库

### 29.2.1\. 连接 MongoDB 数据库

你可以注入一个自动配置的`org.springframework.data.mongodb.MongoDbFactory`来访问 Mongo 数据库。默认情况下，该实例将尝试使用 URL：`mongodb://localhost/test`连接一个 MongoDB 服务器。

```java
import org.springframework.data.mongodb.MongoDbFactory;
import com.mongodb.DB;

@Component
public class MyBean {

    private final MongoDbFactory mongo;

    @Autowired
    public MyBean(MongoDbFactory mongo) {
        this.mongo = mongo;
    }

    // ...
    public void example() {
        DB db = mongo.getDb();
        // ...
    }
} 
```

你可以通过设置`spring.data.mongodb.uri`来改变该 url，或指定一个 host/port。比如，你可能会在你的 application.properties 中设置如下的属性：

```java
spring.data.mongodb.host=mongoserver
spring.data.mongodb.port=27017 
```

**注**：如果没有指定`spring.data.mongodb.port`，那将使用默认的端口 27017。你可以简单的从上面的示例中删除这一行。如果不使用 Spring Data Mongo，你可以注入 com.mongodb.Mongo beans 而不是使用 MongoDbFactory。

如果想全面控制 MongoDB 连接的建立，你也可以声明自己的 MongoDbFactory 或 Mongo，@Beans。

# 29.2.2\. MongoDBTemplate

### 29.2.2\. MongoDBTemplate

Spring Data Mongo 提供了一个[MongoTemplate](http://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html)类，它的设计和 Spring 的 JdbcTemplate 很相似。正如 JdbcTemplate 一样，Spring Boot 会为你自动配置一个 bean，你只需简单的注入它即可：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final MongoTemplate mongoTemplate;

    @Autowired
    public MyBean(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }
    // ...
} 
```

具体参考 MongoOperations Javadoc。

# 29.2.3\. Spring Data MongoDB 仓库

### 29.2.3\. Spring Data MongoDB 仓库

Spring Data 的仓库包括对 MongoDB 的支持。正如上面讨论的 JPA 仓库，基本的原则是查询会自动基于你的方法名创建。

实际上，不管是 Spring Data JPA 还是 Spring Data MongoDB 都共享相同的基础设施。所以你可以使用上面的 JPA 示例，并假设那个 City 现在是一个 Mongo 数据类而不是 JPA　@Entity，它将以同样的方式工作。

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndCountryAllIgnoringCase(String name, String country);

} 
```

# 29.3\. Gemfire

### 29.3\. Gemfire

[Spring Data Gemfire](https://github.com/spring-projects/spring-data-gemfire)为使用[Pivotal Gemfire](http://www.pivotal.io/big-data/pivotal-gemfire#details)数据管理平台提供了方便的，Spring 友好的工具。Spring Boot 提供了一个用于聚集依赖的`spring-boot-starter-data-gemfire`'Starter POM'。目前不支持 Gemfire 的自动配置，但你可以使用一个[单一的注解](https://github.com/spring-projects/spring-data-gemfire/blob/master/src/main/java/org/springframework/data/gemfire/repository/config/EnableGemfireRepositories.java)使 Spring Data 仓库支持它。

# 29.4\. Solr

### 29.4\. Solr

[Apache Solr](http://lucene.apache.org/solr/)是一个搜索引擎。Spring Boot 为 solr 客户端库及[Spring Data Solr](https://github.com/spring-projects/spring-data-solr)提供的基于 solr 客户端库的抽象提供了基本的配置。Spring Boot 提供了一个用于聚集依赖的`spring-boot-starter-data-solr`'Starter POM'。

# 29.4.1\. 连接 Solr

### 29.4.1\. 连接 Solr

你可以像其他 Spring beans 一样注入一个自动配置的 SolrServer 实例。默认情况下，该实例将尝试使用`localhost:8983/solr`连接一个服务器。

```java
@Component
public class MyBean {

    private SolrServer solr;

    @Autowired
    public MyBean(SolrServer solr) {
        this.solr = solr;
    }
    // ...
} 
```

如果你添加一个自己的 SolrServer 类型的@Bean，它将会替换默认的。

# 29.4.2\. Spring Data Solr 仓库

### 29.4.2\. Spring Data Solr 仓库

Spring Data 的仓库包括了对 Apache Solr 的支持。正如上面讨论的 JPA 仓库，基本的原则是查询会自动基于你的方法名创建。

实际上，不管是 Spring Data JPA 还是 Spring Data Solr 都共享相同的基础设施。所以你可以使用上面的 JPA 示例，并假设那个 City 现在是一个@SolrDocument 类而不是 JPA　@Entity，它将以同样的方式工作。

**注**：具体参考[Spring Data Solr 文档](http://projects.spring.io/spring-data-solr/)。

# 29.5\. Elasticsearch

### 29.5\. Elasticsearch

[Elastic Search](http://www.elasticsearch.org/)是一个开源的，分布式，实时搜索和分析引擎。Spring Boot 为 Elasticsearch 及[Spring Data Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch)提供的基于它的抽象提供了基本的配置。Spring Boot 提供了一个用于聚集依赖的`spring-boot-starter-data-elasticsearch`'Starter POM'。

# 29.5.1\. 连接 Elasticsearch

### 29.5.1\. 连接 Elasticsearch

你可以像其他 Spring beans 那样注入一个自动配置的 ElasticsearchTemplate 或 Elasticsearch 客户端实例。默认情况下，该实例将尝试连接到一个本地内存服务器（在 Elasticsearch 项目中的一个 NodeClient），但你可以通过设置`spring.data.elasticsearch.clusterNodes`为一个以逗号分割的 host:port 列表来将其切换到一个远程服务器（比如，TransportClient）。

```java
@Component
public class MyBean {

    private ElasticsearchTemplate template;

    @Autowired
    public MyBean(ElasticsearchTemplate template) {
        this.template = template;
    }
    // ...
} 
```

如果你添加一个你自己的 ElasticsearchTemplate 类型的@Bean，它将替换默认的。

# 29.5.2\. Spring Data Elasticseach 仓库

### 29.5.2\. Spring Data Elasticseach 仓库

Spring Data 的仓库包括了对 Elasticsearch 的支持。正如上面讨论的 JPA 仓库，基本的原则是查询会自动基于你的方法名创建。

实际上，不管是 Spring Data JPA 还是 Spring Data　Elasticsearch 都共享相同的基础设施。所以你可以使用上面的 JPA 示例，并假设那个 City 现在是一个 Elasticsearch @Document 类而不是 JPA　@Entity，它将以同样的方式工作。

**注**：具体参考[Spring Data Elasticsearch 文档](http://docs.spring.io/spring-data/elasticsearch/docs/)。

# 30\. 消息

### 30\. 消息

Spring Framework 框架为集成消息系统提供了扩展（extensive）支持：从使用 JmsTemplate 简化 JMS　API，到实现一个完整异步消息接收的底层设施。

Spring AMQP 提供一个相似的用于'高级消息队列协议'的特征集，并且 Spring Boot 也为 RabbitTemplate 和 RabbitMQ 提供了自动配置选项。

Spring Websocket 提供原生的 STOMP 消息支持，并且 Spring Boot 通过 starters 和一些自动配置也提供了对它的支持。

# 30.1\. JMS

### 30.1\. JMS

javax.jms.ConnectionFactory 接口提供了一个标准的用于创建一个 javax.jms.Connection 的方法，javax.jms.Connection 用于和 JMS 代理（broker）交互。

尽管为了使用 JMS，Spring 需要一个 ConnectionFactory，但通常你不需要直接使用它，而是依赖于上层消息抽象（具体参考 Spring 框架的[相关章节](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#jms)）。Spring Boot 也会自动配置发送和接收消息需要的设施（infrastructure）。

# 30.1.1\. HornetQ 支持

### 30.1.1\. HornetQ 支持

如果在 classpath 下发现 HornetQ，Spring Boot 会自动配置 ConnectionFactory。如果需要代理，将会开启一个内嵌的，已经自动配置好的代理（除非显式设置 mode 属性）。支持的 modes 有：embedded（显式声明使用一个内嵌的代理，如果该代理在 classpath 下不可用将导致一个错误），native（使用 netty 传输协议连接代理）。当后者被配置，Spring Boot 配置一个连接到一个代理的 ConnectionFactory，该代理运行在使用默认配置的本地机器上。

**注**：如果使用 spring-boot-starter-hornetq，连接到一个已存在的 HornetQ 实例所需的依赖都会被提供，同时还有用于集成 JMS 的 Spring 基础设施。将 org.hornetq:hornetq-jms-server 添加到你的应用中，你就可以使用 embedded 模式。

HornetQ 配置被 spring.hornetq.*中的外部配置属性所控制。例如，你可能在 application.properties 声明以下片段：

```java
spring.hornetq.mode=native
spring.hornetq.host=192.168.1.210
spring.hornetq.port=9876 
```

当内嵌代理时，你可以选择是否启用持久化，并且列表中的目标都应该是可用的。这些可以通过一个以逗号分割的列表来指定一些默认的配置项，或定义 org.hornetq.jms.server.config.JMSQueueConfiguration 或 org.hornetq.jms.server.config.TopicConfiguration 类型的 bean(s)来配置更高级的队列和主题。具体参考[HornetQProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/hornetq/HornetQProperties.java)。

没有涉及 JNDI 查找，目标是通过名字解析的，名字即可以使用 HornetQ 配置中的 name 属性，也可以是配置中提供的 names。

# 30.1.2\. ActiveQ 支持

### 30.1.2\. ActiveQ 支持

如果发现 ActiveMQ 在 classpath 下可用，Spring Boot 会配置一个 ConnectionFactory。如果需要代理，将会开启一个内嵌的，已经自动配置好的代理（只要配置中没有指定代理 URL）。

ActiveMQ 配置是通过 spring.activemq.*中的外部配置来控制的。例如，你可能在 application.properties 中声明下面的片段：

```java
spring.activemq.broker-url=tcp://192.168.1.210:9876
spring.activemq.user=admin
spring.activemq.password=secret 
```

具体参考[ActiveMQProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java)。

默认情况下，如果目标还不存在，ActiveMQ 将创建一个，所以目标是通过它们提供的名称解析出来的。

# 30.1.3\. 使用 JNDI ConnectionFactory

### 30.1.3\. 使用 JNDI ConnectionFactory

如果你在一个应用服务器中运行你的应用，Spring Boot 将尝试使用 JNDI 定位一个 JMS ConnectionFactory。默认情况会检查 java:/JmsXA 和 java:/ XAConnectionFactory。如果需要的话，你可以使用 spring.jms.jndi-name 属性来指定一个替代位置。

```java
spring.jms.jndi-name=java:/MyConnectionFactory 
```

# 30.1.4\. 发送消息

### 30.1.4\. 发送消息

Spring 的 JmsTemplate 会被自动配置，你可以将它直接注入到你自己的 beans 中：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;
@Component
public class MyBean {
private final JmsTemplate jmsTemplate;
@Autowired
public MyBean(JmsTemplate jmsTemplate) {
this.jmsTemplate = jmsTemplate;
}
// ...
} 
```

**注**：[JmsMessagingTemplate](http://docs.spring.io/spring/docs/4.1.4.RELEASE/javadoc-api/org/springframework/jms/core/JmsMessagingTemplate.html)(Spring4.1 新增的)也可以使用相同的方式注入

# 30.1.5\. 接收消息

### 30.1.5\. 接收消息

当 JMS 基础设施能够使用时，任何 bean 都能够被@JmsListener 注解，以创建一个监听者端点。如果没有定义 JmsListenerContainerFactory，一个默认的将会被自动配置。下面的组件在 someQueue 目标上创建一个监听者端点。

```java
@Component
public class MyBean {
@JmsListener(destination = "someQueue")
public void processMessage(String content) {
// ...
}
} 
```

具体查看[@EnableJms javadoc](http://docs.spring.io/spring/docs/4.1.4.RELEASE/javadoc-api/org/springframework/jms/annotation/EnableJms.html)。

# 31\. 发送邮件

### 31\. 发送邮件

Spring 框架使用 JavaMailSender 接口为发送邮件提供了一个简单的抽象，并且 Spring Boot 也为它提供了自动配置和一个 starter 模块。 具体查看[JavaMailSender 参考文档](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#mail)。

如果 spring.mail.host 和相关的库（通过 spring-boot-starter-mail 定义）都存在，一个默认的 JavaMailSender 将被创建。该 sender 可以通过 spring.mail 命名空间下的配置项进一步自定义，具体参考[MailProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mail/MailProperties.java)。

# 32\. 使用 JTA 处理分布式事务

### 32\. 使用 JTA 处理分布式事务

Spring Boot 使用一个[Atomkos](http://www.atomikos.com/)或[Bitronix](http://docs.codehaus.org/display/BTM/Home)的内嵌事务管理器来支持跨多个 XA 资源的分布式 JTA 事务。当部署到一个恰当的 J2EE 应用服务器时也会支持 JTA 事务。

当发现一个 JTA 环境时，Spring Boot 将使用 Spring 的 JtaTransactionManager 来管理事务。自动配置的 JMS，DataSource 和 JPA　beans 将被升级以支持 XA 事务。你可以使用标准的 Spring idioms，比如@Transactional，来参与到一个分布式事务中。如果你处于 JTA 环境里，但仍旧想使用本地事务，你可以将 spring.jta.enabled 属性设置为 false 来禁用 JTA 自动配置功能。

# 32.1\. 使用一个 Atomikos 事务管理器

### 32.1\. 使用一个 Atomikos 事务管理器

Atomikos 是一个非常流行的开源事务管理器，它可以嵌入到你的 Spring Boot 应用中。你可以使用`spring-boot-starter-jta-atomikos`Starter POM 去获取正确的 Atomikos 库。Spring Boot 会自动配置 Atomikos，并将合适的 depends-on 应用到你的 Spring Beans 上，确保它们以正确的顺序启动和关闭。

默认情况下，Atomikos 事务日志将被记录在应用 home 目录（你的应用 jar 文件放置的目录）下的 transaction-logs 文件夹中。你可以在 application.properties 文件中通过设置 spring.jta.log-dir 属性来自定义该目录。以 spring.jta.开头的属性能用来自定义 Atomikos 的 UserTransactionServiceIml 实现。具体参考[AtomikosProperties javadoc](http://docs.spring.io/spring-boot/docs/1.2.2.BUILD-SNAPSHOT/api/org/springframework/boot/jta/atomikos/AtomikosProperties.html)。

**注**：为了确保多个事务管理器能够安全地和相应的资源管理器配合，每个 Atomikos 实例必须设置一个唯一的 ID。默认情况下，该 ID 是 Atomikos 实例运行的机器上的 IP 地址。为了确保生产环境中该 ID 的唯一性，你需要为应用的每个实例设置不同的 spring.jta.transaction-manager-id 属性值。

# 32.2\. 使用一个 Bitronix 事务管理器

### 32.2\. 使用一个 Bitronix 事务管理器

Bitronix 是另一个流行的开源 JTA 事务管理器实现。你可以使用`spring-boot-starter-jta-bitronix`starter POM 为项目添加合适的 Birtronix 依赖。和 Atomikos 类似，Spring Boot 将自动配置 Bitronix，并对 beans 进行后处理（post-process）以确保它们以正确的顺序启动和关闭。

默认情况下，Bitronix 事务日志将被记录到应用 home 目录下的 transaction-logs 文件夹中。通过设置 spring.jta.log-dir 属性，你可以自定义该目录。以 spring.jta.开头的属性将被绑定到 bitronix.tm.Configuration　bean，你可以通过这完成进一步的自定义。具体参考[Bitronix 文档](http://btm.codehaus.org/api/2.0.1/bitronix/tm/Configuration.html)。

**注**：为了确保多个事务管理器能够安全地和相应的资源管理器配合，每个 Bitronix 实例必须设置一个唯一的 ID。默认情况下，该 ID 是 Bitronix 实例运行的机器上的 IP 地址。为了确保生产环境中该 ID 的唯一性，你需要为应用的每个实例设置不同的 spring.jta.transaction-manager-id 属性值。

# 32.3\. 使用一个 J2EE 管理的事务管理器

### 32.3\. 使用一个 J2EE 管理的事务管理器

如果你将 Spring Boot 应用打包为一个 war 或 ear 文件，并将它部署到一个 J2EE 的应用服务器中，那你就能使用应用服务器内建的事务管理器。Spring Boot 将尝试通过查找常见的 JNDI 路径（java:comp/UserTransaction, java:comp/TransactionManager 等）来自动配置一个事务管理器。如果使用应用服务器提供的事务服务，你通常需要确保所有的资源都被应用服务器管理，并通过 JNDI 暴露出去。Spring Boot 通过查找 JNDI 路径 java:/JmsXA 或 java:/XAConnectionFactory 获取一个 ConnectionFactory 来自动配置 JMS，并且你可以使用 spring.datasource.jndi-name 属性配置你的 DataSource。

# 32.4\. 混合 XA 和 non-XA 的 JMS 连接

### 32.4\. 混合 XA 和 non-XA 的 JMS 连接

当使用 JTA 时，主要的 JMS ConnectionFactory bean 将是 XA　aware，并参与到分布式事务中。有些情况下，你可能需要使用 non-XA 的 ConnectionFactory 去处理一些 JMS 消息。例如，你的 JMS 处理逻辑可能比 XA 超时时间长。

如果想使用一个 non-XA 的 ConnectionFactory，你可以注入 nonXaJmsConnectionFactory　bean 而不是@Primary jmsConnectionFactory　bean。为了保持一致，jmsConnectionFactory　bean 将以别名 xaJmsConnectionFactor 来被使用。

示例如下：

```java
// Inject the primary (XA aware) ConnectionFactory
@Autowired
private ConnectionFactory defaultConnectionFactory;
// Inject the XA aware ConnectionFactory (uses the alias and injects the same as above)
@Autowired
@Qualifier("xaJmsConnectionFactory")
private ConnectionFactory xaConnectionFactory;
// Inject the non-XA aware ConnectionFactory
@Autowired
@Qualifier("nonXaJmsConnectionFactory")
private ConnectionFactory nonXaConnectionFactory; 
```

# 33\. Spring 集成

### 33\. Spring 集成

Spring 集成提供基于消息和其他协议的，比如 HTTP，TCP 等的抽象。如果 Spring 集成在 classpath 下可用，它将会通过@EnableIntegration 注解被初始化。如果 classpath 下'spring-integration-jmx'可用，则消息处理统计分析将被通过 JMX 发布出去。具体参考[IntegrationAutoConfiguration 类](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationAutoConfiguration.java)。

# 34\. 基于 JMX 的监控和管理

### 34\. 基于 JMX 的监控和管理

Java 管理扩展（JMX）提供了一个标准的用于监控和管理应用的机制。默认情况下，Spring Boot 将创建一个 id 为‘mbeanServer’的 MBeanServer，并导出任何被 Spring JMX 注解（@ManagedResource,@ManagedAttribute,@ManagedOperation）的 beans。具体参考[JmxAutoConfiguration 类](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.java)。

# 35\. 测试

### 35\. 测试

Spring Boot 提供很多有用的测试应用的工具。spring-boot-starter-test POM 提供 Spring Test，JUnit，Hamcrest 和 Mockito 的依赖。在 spring-boot 核心模块 org.springframework.boot.test 包下也有很多有用的测试工具。

# 35.1\. 测试作用域依赖

### 35.1\. 测试作用域依赖

如果使用 spring-boot-starter-test ‘Starter POM’（在 test 作用域内），你将发现下列被提供的库：

*   Spring Test - 对 Spring 应用的集成测试支持
*   JUnit - 事实上的(de-facto)标准，用于 Java 应用的单元测试。
*   Hamcrest - 一个匹配对象的库（也称为约束或前置条件），它允许 assertThat 等 JUnit 类型的断言。
*   Mockito - 一个 Java 模拟框架。

这也有一些我们写测试用例时经常用到的库。如果它们不能满足你的要求，你可以随意添加其他的测试用的依赖库。

# 35.2\. 测试 Spring 应用

### 35.2\. 测试 Spring 应用

依赖注入最大的优点就是它能够让你的代码更容易进行单元测试。你只需简单的通过 new 操作符实例化对象，而不需要涉及 Spring。你也可以使用模拟对象替换真正的依赖。

你常常需要在进行单元测试后，开始集成测试（在这个过程中只需要涉及到 Spring 的 ApplicationContext）。在执行集成测试时，不需要部署应用或连接到其他基础设施是非常有用的。

Spring 框架包含一个 dedicated 测试模块，用于这样的集成测试。你可以直接声明对 org.springframework:spring-test 的依赖，或使用 spring-boot-starter-test ‘Starter POM’以透明的方式拉取它。

如果你以前没有使用过 spring-test 模块，可以查看 Spring 框架参考文档中的[相关章节](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#testing)。

# 35.3\. 测试 Spring Boot 应用

### 35.3\. 测试 Spring Boot 应用

一个 Spring Boot 应用只是一个 Spring ApplicationContext，所以在测试它时除了正常情况下处理一个 vanilla Spring　context 外不需要做其他特别事情。唯一需要注意的是，如果你使用 SpringApplication 创建上下文，外部配置，日志和 Spring Boot 的其他特性只会在默认的上下文中起作用。

Spring Boot 提供一个@SpringApplicationConfiguration 注解用来替换标准的 spring-test　@ContextConfiguration 注解。如果使用@SpringApplicationConfiguration 来设置你的测试中使用的 ApplicationContext，它最终将通过 SpringApplication 创建，并且你将获取到 Spring Boot 的其他特性。

示例如下：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = SampleDataJpaApplication.class)
public class CityRepositoryIntegrationTests {
@Autowired
CityRepository repository;
// ...
} 
```

**提示**：上下文加载器会通过查找@WebIntegrationTest 或@WebAppConfiguration 注解来猜测你想测试的是否是 web 应用（例如，是否使用 MockMVC，MockMVC 和@WebAppConfiguration 是 spring-test 的一部分）。

如果想让一个 web 应用启动，并监听它的正常的端口，你可以使用 HTTP 来测试它（比如，使用 RestTemplate），并使用@WebIntegrationTest 注解你的测试类（或它的一个父类）。这很有用，因为它意味着你可以对你的应用进行全栈测试，但在一次 HTTP 交互后，你需要在你的测试类中注入相应的组件并使用它们断言应用的内部状态。

示例：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = SampleDataJpaApplication.class)
@WebIntegrationTest
public class CityRepositoryIntegrationTests {
@Autowired
CityRepository repository;
RestTemplate restTemplate = new TestRestTemplate();
// ... interact with the running server
} 
```

**注**：Spring 测试框架在每次测试时会缓存应用上下文。因此，只要你的测试共享相同的配置，不管你实际运行多少测试，开启和停止服务器只会发生一次。

你可以为@WebIntegrationTest 添加环境变量属性来改变应用服务器端口号，比如@WebIntegrationTest("server.port:9000")。此外，你可以将 server.port 和 management.port 属性设置为０来让你的集成测试使用随机的端口号，例如：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MyApplication.class)
@WebIntegrationTest({"server.port=0", "management.port=0"})
public class SomeIntegrationTests {
// ...
} 
```

可以查看 Section 64.4, “Discover the HTTP port at runtime”，它描述了如何在测试期间发现分配的实际端口。

# 35.3.1\. 使用 Spock 测试 Spring Boot 应用

### 35.3.1\. 使用 Spock 测试 Spring Boot 应用

如果期望使用 Spock 测试一个 Spring Boot 应用，你应该将 Spock 的 spock-spring 模块依赖添加到应用的构建中。spock-spring 将 Spring 的测试框架集成到了 Spock 里。

注意你不能使用上述提到的@SpringApplicationConfiguration 注解，因为[Spock 找不到@ContextConfiguration 元注解](https://code.google.com/p/spock/issues/detail?id=349)。为了绕过该限制，你应该直接使用@ContextConfiguration 注解，并使用 Spring Boot 特定的上下文加载器来配置它。

```java
@ContextConfiguration(loader = SpringApplicationContextLoader.class)
class ExampleSpec extends Specification {
// ...
} 
```

**注**：上面描述的注解在 Spock 中可以使用，比如，你可以使用@WebIntegrationTest 注解你的 Specification 以满足测试需要。

# 35.4\. 测试工具

### 35.4\. 测试工具

打包进 spring-boot 的一些有用的测试工具类。

# 35.4.1\. ConfigFileApplicationContextInitializer

### 35.4.1\. ConfigFileApplicationContextInitializer

ConfigFileApplicationContextInitializer 是一个 ApplicationContextInitializer，可以用来测试加载 Spring Boot 的 application.properties 文件。当不需要使用@SpringApplicationConfiguration 提供的全部特性时，你可以使用它。

```java
@ContextConfiguration(classes = Config.class,initializers = ConfigFileApplicationContextInitializer.class) 
```

# 35.4.2\. EnvironmentTestUtils

### 35.4.2\. EnvironmentTestUtils

EnvironmentTestUtils 允许你快速添加属性到一个 ConfigurableEnvironment 或 ConfigurableApplicationContext。只需简单的使用 key=value 字符串调用它： ```java EnvironmentTestUtils.addEnvironment(env, "org=Spring", "name=Boot");

# 35.4.3\. OutputCapture

### 35.4.3\. OutputCapture

OutputCapture 是一个 JUnit Rule，用于捕获 System.out 和 System.err 输出。只需简单的将捕获声明为一个@Rule，并使用 toString()断言：

```
import org.junit.Rule;
import org.junit.Test;
import org.springframework.boot.test.OutputCapture;
import static org.hamcrest.Matchers.*;
import static org.junit.Assert.*;

public class MyTest {
@Rule
public OutputCapture capture = new OutputCapture();
@Test
public void testName() throws Exception {
System.out.println("Hello World!");
assertThat(capture.toString(), containsString("World"));
}
} 
```java

# 35.4.4\. TestRestTemplate

### 35.4.4\. TestRestTemplate

TestRestTemplate 是一个方便进行集成测试的 Spring RestTemplate 子类。你会获取到一个普通的模板或一个发送基本 HTTP 认证（使用用户名和密码）的模板。在任何情况下，这些模板都表现出对测试友好：不允许重定向（这样你可以对响应地址进行断言），忽略 cookies（这样模板就是无状态的），对于服务端错误不会抛出异常。推荐使用 Apache HTTP Client(4.3.2 或更好的版本)，但不强制这样做。如果在 classpath 下存在 Apache HTTP Client，TestRestTemplate 将以正确配置的 client 进行响应。

```
public class MyTest {
RestTemplate template = new TestRestTemplate();
@Test
public void testRequest() throws Exception {
HttpHeaders headers = template.getForEntity("http://myhost.com", String.class).getHeaders();
assertThat(headers.getLocation().toString(), containsString("myotherhost"));
}
} 
```java

# 36\. 开发自动配置和使用条件

### 36\. 开发自动配置和使用条件

如果你在一个开发者共享库的公司工作，或你在从事一个开源或商业型的库，你可能想要开发自己的 auto-configuration。Auto-configuration 类能够在外部的 jars 中绑定，并仍能被 Spring Boot 发现。

# 36.1\. 理解 auto-configured beans

### 36.1\. 理解 auto-configured beans

从底层来讲，auto-configured 是使用标准的@Configuration 实现的类，另外的@Conditional 注解用来约束在什么情况下使用 auto-configuration。通常 auto-configuration 类使用@ConditionalOnClass 和@ConditionalOnMissingBean 注解。这是为了确保只有在相关的类被发现，和你没有声明自己的@Configuration 时才应用 auto-configuration。

你可以浏览 spring-boot-autoconfigure 的源码，查看我们提供的@Configuration 类（查看 META-INF/spring.factories 文件）。

# 36.2\. 定位 auto-configuration 候选者

### 36.2\. 定位 auto-configuration 候选者

Spring Boot 会检查你发布的 jar 中是否存在 META-INF/spring.factories 文件。该文件应该列出以 EnableAutoConfiguration 为 key 的配置类：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration 
```

如果配置需要应用特定的顺序，你可以使用[@AutoConfigureAfter](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.java)或[@AutoConfigureBefore](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.java)注解。例如，你想提供 web-specific 配置，你的类就需要应用在 WebMvcAutoConfiguration 后面。

# 36.3\. Condition 注解

### 36.3\. Condition 注解

你几乎总是需要在你的 auto-configuration 类里添加一个或更多的@Condition 注解。@ConditionalOnMissingBean 注解是一个常见的示例，它经常用于允许开发者覆盖 auto-configuration，如果他们不喜欢你提供的默认行为。

Spring Boot 包含很多@Conditional 注解，你可以在自己的代码中通过注解@Configuration 类或单独的@Bean 方法来重用它们。

# 36.3.1\. Class 条件

### 36.3.1\. Class 条件

@ConditionalOnClass 和@ConditionalOnMissingClass 注解允许根据特定类是否出现来跳过配置。由于注解元数据是使用[ASM](http://asm.ow2.org/)来解析的，你实际上可以使用 value 属性来引用真正的类，即使该类可能实际上并没有出现在运行应用的 classpath 下。如果你倾向于使用一个 String 值来指定类名，你也可以使用 name 属性。

# 36.3.2\. Bean 条件

### 36.3.2\. Bean 条件

@ConditionalOnBean 和@ConditionalOnMissingBean 注解允许根据特定 beans 是否出现来跳过配置。你可以使用 value 属性来指定 beans（by type），也可以使用 name 来指定 beans（by name）。search 属性允许你限制搜索 beans 时需要考虑的 ApplicationContext 的层次。

**注**：当@Configuration 类被解析时@Conditional 注解会被处理。Auto-configure @Configuration 总是最后被解析（在所有用户定义 beans 后面），然而，如果你将那些注解用到常规的@Configuration 类，需要注意不能引用那些还没有创建好的 bean 定义。

# 36.3.3\. Property 条件

### 36.3.3\. Property 条件

@ConditionalOnProperty 注解允许根据一个 Spring Environment 属性来决定是否包含配置。可以使用 prefix 和 name 属性指定要检查的配置属性。默认情况下，任何存在的只要不是 false 的属性都会匹配。你也可以使用 havingValue 和 matchIfMissing 属性创建更高级的检测。

# 36.3.4\. Resource 条件

### 36.3.4\. Resource 条件

@ConditionalOnResource 注解允许只有在特定资源出现时配置才会被包含。资源可以使用常见的 Spring 约定命名，例如 file:/home/user/test.dat。

# 36.3.5\. Web Application 条件

### 36.3.5\. Web Application 条件

@ConditionalOnWebApplication 和@ConditionalOnNotWebApplication 注解允许根据应用是否为一个'web 应用'来决定是否包含配置。一个 web 应用是任何使用 Spring WebApplicationContext，定义一个 session 作用域或有一个 StandardServletEnvironment 的应用。

# 36.3.6\. SpEL 表达式条件

### 36.3.6\. SpEL 表达式条件

@ConditionalOnExpression 注解允许根据[SpEL 表达式](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#expressions)结果来跳过配置。

# 37\. WebSockets

### 37\. WebSockets

Spring Boot 为内嵌的 Tomcat(8 和 7)，Jetty 9 和 Undertow 提供 WebSockets 自动配置。如果你正在将一个 war 包部署到一个单独的容器，Spring Boot 会假设该容器会对它的 WebSocket 支持相关的配置负责。

Spring 框架提供[丰富的 WebSocket 支持](http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/htmlsingle/#websocket)，通过 spring-boot-starter-websocket 模块可以轻易获取到。

# 38\. 接下来阅读什么

### 38\. 接下来阅读什么