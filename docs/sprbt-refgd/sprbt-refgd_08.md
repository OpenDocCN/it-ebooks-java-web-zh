# VIII. 构建工具插件

### 构建工具插件

Spring Boot 为 Maven 和 Gradle 提供构建工具插件。该插件提供各种各样的特性，包括打包可执行 jars。本节提供关于插件的更多详情及用于扩展一个不支持的构建系统所需的帮助信息。如果你是刚刚开始，那可能需要先阅读[Part III, “Using Spring Boot”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot)章节的[“Chapter 13, Build systems”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-build-systems)。

# 58\. Spring Boot Maven 插件

### 58\. Spring Boot Maven 插件

Spring Boot Maven 插件为 Maven 提供 Spring Boot 支持，它允许你打包可执行 jar 或 war 存档，然后就地运行应用。为了使用它，你需要使用 Maven 3.2 （或更高版本）。

**注**：参考[Spring Boot Maven Plugin Site](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/)可以获取全部的插件文档。

# 58.1\. 包含该插件

### 58.1\. 包含该插件

想要使用 Spring Boot Maven 插件只需简单地在你的 pom.xml 的`plugins`部分包含相应的 XML：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- ... -->
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
</project> 
```

该配置会在 Maven 生命周期的`package`阶段重新打包一个 jar 或 war。下面的示例显示在`target`目录下既有重新打包后的 jar，也有原始的 jar：

```java
$ mvn package
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original 
```

如果不包含像上面那样的`<execution/>`，你可以自己运行该插件（但只有在 package 目标也被使用的情况）。例如：

```java
$ mvn package spring-boot:repackage
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original 
```

如果使用一个里程碑或快照版本，你还需要添加正确的 pluginRepository 元素：

```java
<pluginRepositories>
    <pluginRepository>
        <id>spring-snapshots</id>
        <url>http://repo.spring.io/snapshot</url>
    </pluginRepository>
    <pluginRepository>
        <id>spring-milestones</id>
        <url>http://repo.spring.io/milestone</url>
    </pluginRepository>
</pluginRepositories> 
```

# 58.2\. 打包可执行 jar 和 war 文件

### 58.2\. 打包可执行 jar 和 war 文件

一旦`spring-boot-maven-plugin`被包含到你的 pom.xml 中，它就会自动尝试使用`spring-boot:repackage`目标重写存档以使它们能够执行。为了构建一个 jar 或 war，你应该使用常规的 packaging 元素配置你的项目：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>jar</packaging>
    <!-- ... -->
</project> 
```

生成的存档在`package`阶段会被 Spring Boot 增强。你想启动的 main 类即可以通过指定一个配置选项，也可以通过为 manifest 添加一个`Main-Class`属性这种常规的方式实现。如果你没有指定一个 main 类，该插件会搜索带有`public static void main(String[] args)`方法的类。

为了构建和运行一个项目的 artifact，你可以输入以下命令：

```java
$ mvn package
$ java -jar target/mymodule-0.0.1-SNAPSHOT.jar 
```

为了构建一个即是可执行的，又能部署到一个外部容器的 war 文件，你需要标记内嵌容器依赖为"provided"，例如：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>war</packaging>
    <!-- ... -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <!-- ... -->
    </dependencies>
</project> 
```

**注**：具体参考“Section 74.1, “Create a deployable war file”” 章节。

在[插件信息页面](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/)有高级的配置选项和示例。

# 59\. Spring Boot Gradle 插件

### 59\. Spring Boot Gradle 插件

Spring Boot Gradle 插件为 Gradle 提供 Spring Boot 支持，它允许你打包可执行 jar 或 war 存档，运行 Spring Boot 应用，对于"神圣的"依赖可以在你的 build.gradle 文件中省略版本信息。

# 59.1\. 包含该插件

### 59.1\. 包含该插件

想要使用 Spring Boot Gradle 插件，你只需简单的包含一个`buildscript`依赖，并应用`spring-boot`插件：

```java
buildscript {
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.0.BUILD-SNAPSHOT")
    }
}
apply plugin: 'spring-boot' 
```

如果想使用一个里程碑或快照版本，你可以添加相应的 repositories 引用：

```java
buildscript {
    repositories {
        maven.url "http://repo.spring.io/snapshot"
        maven.url "http://repo.spring.io/milestone"
    }
    // ...
} 
```

# 59.2\. 声明不带版本的依赖

### 59.2\. 声明不带版本的依赖

`spring-boot`插件会为你的构建注册一个自定义的 Gradle `ResolutionStrategy`，它允许你在声明对"神圣"的 artifacts 的依赖时获取版本号。为了充分使用该功能，只需要想通常那样声明依赖，但将版本号设置为空：

```java
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.thymeleaf:thymeleaf-spring4")
    compile("nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect")
} 
```

**注**：你声明的`spring-boot` Gradle 插件的版本决定了"blessed"依赖的实际版本（确保可以重复构建）。你最好总是将`spring-boot` gradle 插件版本设置为你想用的 Spring Boot 实际版本。提供的版本详细信息可以在附录中找到。

`spring-boot`插件对于没有指定版本的依赖只会提供一个版本。如果不想使用插件提供的版本，你可以像平常那样在声明依赖的时候指定版本。例如：

```java
dependencies {
    compile("org.thymeleaf:thymeleaf-spring4:2.1.1.RELEASE")
} 
```

# 59.2.1\. 自定义版本管理

### 59.2.1\. 自定义版本管理

如果你需要不同于 Spring Boot 的"blessed"依赖，有可能的话可以自定义`ResolutionStrategy`使用的版本。替代的版本元数据使用`versionManagement`配置。例如：

```java
dependencies {
    versionManagement("com.mycorp:mycorp-versions:1.0.0.RELEASE@properties")
    compile("org.springframework.data:spring-data-hadoop")
} 
```

版本信息需要作为一个`.properties`文件发布到一个仓库中。对于上面的示例，`mycorp-versions.properties`文件可能包含以下内容：

```java
org.springframework.data\:spring-data-hadoop=2.0.0.RELEASE 
```

属性文件优先于 Spring Boot 默认设置，如果有必要的话可以覆盖版本号。

# 59.3\. 默认排除规则

### 59.3\. 默认排除规则

Gradle 处理"exclude rules"的方式和 Maven 稍微有些不同，在使用 starter POMs 时这可能会引起无法预料的结果。特别地，当一个依赖可以通过不同的路径访问时，对该依赖声明的 exclusions 将不会生效。例如，如果一个 starter POM 声明以下内容：

```java
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.0.5.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.0.5.RELEASE</version>
    </dependency>
</dependencies> 
```

`commons-logging` jar 不会被 Gradle 排除，因为通过没有`exclusion`元素的`spring-context`可以传递性的拉取到它（spring-context → spring-core → commons-logging）。

为了确保正确的排除被实际应用，Spring Boot Gradle 插件将自动添加排除规则。所有排除被定义在`spring-boot-dependencies` POM，并且针对"starter" POMs 的隐式规则也会被添加。

如果不想自动应用排除规则，你可以使用以下配置：

```java
springBoot {
    applyExcludeRules=false
} 
```

# 59.4\. 打包可执行 jar 和 war 文件

### 59.4\. 打包可执行 jar 和 war 文件

一旦`spring-boot`插件被应用到你的项目，它将使用`bootRepackage`任务自动尝试重写存档以使它们能够执行。为了构建一个 jar 或 war，你需要按通常的方式配置项目。

你想启动的 main 类既可以通过一个配置选项指定，也可以通过向 manifest 添加一个`Main-Class`属性。如果你没有指定 main 类，该插件会搜索带有`public static void main(String[] args)`方法的类。

为了构建和运行一个项目 artifact，你可以输入以下内容：

```java
$ gradle build
$ java -jar build/libs/mymodule-0.0.1-SNAPSHOT.jar 
```

为了构建一个即能执行也可以部署到外部容器的 war 包，你需要将内嵌容器依赖标记为"providedRuntime"，比如：

```java
...
apply plugin: 'war'

war {
    baseName = 'myapp'
    version =  '0.5.0'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/libs-snapshot" }
}

configurations {
    providedRuntime
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")
    ...
} 
```

**注**：具体参考“Section 74.1, “Create a deployable war file””。

# 59.5\. 就地（in-place）运行项目

### 59.5\. 就地（in-place）运行项目

为了在不先构建 jar 的情况下运行项目，你可以使用"bootRun"任务：

```java
$ gradle bootRun 
```

默认情况下，以这种方式运行项目可以让你的静态 classpath 资源（比如，默认位于`src/main/resources`下）在应用运行期间被重新加载。使静态资源可以重新加载意味着`bootRun`任务不会使用`processResources`任务的输出，比如，当调用`bootRun`时，你的应用将以资源未处理的形式来使用它们。

你可以禁止直接使用静态 classpath 资源。这意味着资源不再是可重新加载的，但`processResources`任务的输出将会被使用。想要这样做，只需将`bootRun`任务的`addResources`设为 false：

```java
bootRun {
    addResources = false
} 
```

# 59.6\. Spring Boot 插件配置

### 59.6\. Spring Boot 插件配置

Gradle 插件自动扩展你的构建脚本 DSL，它为脚本添加一个`springBoot`元素以此作为 Boot 插件的全局配置。你可以像配置其他 Gradle 扩展那样为`springBoot`设置相应的属性（下面有配置选项列表）。

```java
springBoot {
    backupSource = false
} 
```

# 59.7\. Repackage 配置

### 59.7\. Repackage 配置

该插件添加了一个 bootRepackage 任务，你可以直接配置它，比如：

```java
bootRepackage {
    mainClass = 'demo.Application'
} 
```

下面是可用的配置选项：

| 名称 | 描述 |
| --- | --- |
| enabled | 布尔值，用于控制 repackager 的开关（如果你只想要 Boot 的其他特性而不是这个，那它就派上用场了） |
| mainClass | 要运行的 main 类。如果没有指定，则使用 project 属性 mainClassName。如果没有定义 mainClassName id，则搜索存档以寻找一个合适的类。"合适"意味着一个唯一的，具有良好格式的`main()`方法的类（如果找到多个则构建会失败）。你也可以通过"run"任务（main 属性）指定 main 类的名称，和/或将"startScripts"（mainClassName 属性）作为"springBoot"配置的替代。 |
| classifier | 添加到存档的一个文件名字段（在扩展之前），这样最初保存的存档仍旧存放在最初的位置。在存档被重新打包（repackage）的情况下，该属性默认为 null。默认值适用于多数情况，但如果你想在另一个项目中使用原 jar 作为依赖，最好使用一个扩展来定义该可执行 jar |
| withJarTask | Jar 任务的名称或值，用于定位要被 repackage 的存档 |
| customConfiguration | 自定义配置的名称，用于填充内嵌的 lib 目录（不指定该属性，你将获取所有编译和运行时依赖） |

# 59.8\. 使用 Gradle 自定义配置进行 Repackage

### 59.8\. 使用 Gradle 自定义配置进行 Repackage

有时候不打包解析自 compile，runtime 和 provided 作用域的默认依赖可能更合适些。如果创建的可执行 jar 被原样运行，你需要将所有的依赖内嵌进该 jar 中；然而，如果目的是 explode 一个 jar 文件，并手动运行 main 类，你可能在 CLASSPATH 下已经有一些可用的库了。在这种情况下，你可以使用不同的依赖集重新打包（repackage）你的 jar。

使用自定义的配置将自动禁用来自 compile，runtime 和 provided 作用域的依赖解析。自定义配置即可以定义为全局的（处于 springBoot 部分内），也可以定义为任务级的。

```java
task clientJar(type: Jar) {
    appendix = 'client'
    from sourceSets.main.output
    exclude('**/*Something*')
}

task clientBoot(type: BootRepackage, dependsOn: clientJar) {
    withJarTask = clientJar
    customConfiguration = "mycustomconfiguration"
} 
```

在以上示例中，我们创建了一个新的 clientJar Jar 任务从你编译后的源中打包一个自定义文件集。然后我们创建一个新的 clientBoot BootRepackage 任务，并让它使用 clientJar 任务和 mycustomconfiguration。

```java
configurations {
    mycustomconfiguration.exclude group: 'log4j'
}

dependencies {
    mycustomconfiguration configurations.runtime
} 
```

在 BootRepackage 中引用的配置是一个正常的[Gradle 配置](http://www.gradle.org/docs/current/dsl/org.gradle.api.artifacts.Configuration.html)。在上面的示例中，我们创建了一个新的名叫 mycustomconfiguration 的配置，指示它来自一个 runtime，并排除对 log4j 的依赖。如果 clientBoot 任务被执行，重新打包的 jar 将含有所有来自 runtime 作用域的依赖，除了 log4j jars。

# 59.8.1\. 配置选项

### 59.8.1\. 配置选项

可用的配置选项如下：

| 名称 | 描述 |
| --- | --- |
| mainClass | 可执行 jar 运行的 main 类 |
| providedConfiguration | provided 配置的名称（默认为 providedRuntime） |
| backupSource | 在重新打包之前，原先的存档是否备份（默认为 true） |
| customConfiguration | 自定义配置的名称 |
| layout | 存档类型，对应于内部依赖是如何制定的（默认基于存档类型进行推测） |
| requiresUnpack | 一个依赖列表（格式为"groupId:artifactId"，为了运行，它们需要从 fat jars 中解压出来。）所有节点被打包进胖 jar，但运行的时候它们将被自动解压 |

# 59.9\. 理解 Gradle 插件是如何工作的

### 59.9\. 理解 Gradle 插件是如何工作的

当`spring-boot`被应用到你的 Gradle 项目，一个默认的名叫`bootRepackage`的任务被自动创建。`bootRepackage`任务依赖于 Gradle `assemble`任务，当执行时，它会尝试找到所有限定符为空的 jar artifacts（也就是说，tests 和 sources jars 被自动跳过）。

由于`bootRepackage`查找'所有'创建 jar artifacts 的事实，Gradle 任务执行的顺序就非常重要了。多数项目只创建一个单一的 jar 文件，所以通常这不是一个问题。然而，如果你正打算创建一个更复杂的，使用自定义`jar`和`BootRepackage`任务的项目 setup，有几个方面需要考虑。

如果'仅仅'从项目创建自定义 jar 文件，你可以简单地禁用默认的`jar`和`bootRepackage`任务：

```java
jar.enabled = false
bootRepackage.enabled = false 
```

另一个选项是指示默认的`bootRepackage`任务只能使用一个默认的`jar`任务：

```java
bootRepackage.withJarTask = jar 
```

如果你有一个默认的项目 setup，在该项目中，主（main）jar 文件被创建和重新打包。并且，你仍旧想创建额外的自定义 jars，你可以将自定义的 repackage 任务结合起来，然后使用`dependsOn`，这样`bootJars`任务就会在默认的`bootRepackage`任务执行以后运行：

```java
task bootJars
bootJars.dependsOn = [clientBoot1,clientBoot2,clientBoot3]
build.dependsOn(bootJars) 
```

上面所有方面经常用于避免一个已经创建的 boot jar 又被重新打包的情况。重新打包一个存在的 boot jar 不是什么大问题，但你可能会发现它包含不必要的依赖。

# 60\. 对其他构建系统的支持

### 60\. 对其他构建系统的支持

如果想使用除了 Maven 和 Gradle 之外的构建工具，你可能需要开发自己的插件。可执行 jars 需要遵循一个特定格式，并且一些实体需要以不压缩的方式写入（详情查看附录中的[可执行 jar 格式](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#executable-jar)章节）。

Spring Boot Maven 和 Gradle 插件都利用`spring-boot-loader-tools`来实际地产生 jars。如果需要，你也可以自由地直接使用该库。

# 60.1\. 重新打包存档

### 60.1\. 重新打包存档

使用`org.springframework.boot.loader.tools.Repackager`可以将一个存在的存档重新打包，这样它就变成一个自包含的可执行存档。`Repackager`类需要提供单一的构造器参数，它引用一个存在的 jar 或 war 包。使用两个可用的`repackage()`方法中的一个来替换原始的文件或写入一个新的目标。在 repackager 运行前还可以设置各种配置。

# 60.2\. 内嵌的库

### 60.2\. 内嵌的库

当重新打包一个存档时，你可以使用`org.springframework.boot.loader.tools.Libraries`接口来包含对依赖文件的引用。在这里我们不提供任何该 Libraries 接口的具体实现，因为它们通常跟具体的构建系统相关。

如果你的存档已经包含 libraries，你可以使用`Libraries.NONE`。

# 60.3\. 查找 main 类

### 60.3\. 查找 main 类

如果你没有使用`Repackager.setMainClass()`指定一个 main 类，该 repackager 将使用[ASM](http://asm.ow2.org/)去读取 class 文件，然后尝试查找一个合适的，具有`public static void main(String[] args)`方法的类。如果发现多个候选者，将会抛出异常。

# 60.4\. repackage 实现示例

### 60.4\. repackage 实现示例

这里是一个传统的 repackage 示例：

```java
Repackager repackager = new Repackager(sourceJarFile);
repackager.setBackupSource(false);
repackager.repackage(new Libraries() {
            @Override
            public void doWithLibraries(LibraryCallback callback) throws IOException {
                // Build system specific implementation, callback for each dependency
                // callback.library(new Library(nestedFile, LibraryScope.COMPILE));
            }
        }); 
```

# 61\. 接下来阅读什么

### 61\. 接下来阅读什么