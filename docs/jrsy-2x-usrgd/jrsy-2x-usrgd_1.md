# 第九章 对常用媒体类型的支持

# Chapter 9\. Support for Common Media Type Representations 支持常用媒体类型

# 9.1 JSON

# 9.1\. JSON

Jersey JSON 支持之际,一组扩展模块,每个模块包含一个[功能](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Feature.html)的实现,需要注册到您的[配置](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configurable.html)实例(客户机/服务器)。有多个框架提供支持 JSON 处理和/或 JSON-to-Java 绑定。下面列出的模块提供支持 JSON 表示通过整合个人 JSON 框架 Jersey。目前,Jersey 集成了以下模块提供 JSON 支持:

*   [MOXy](https://jersey.java.net/documentation/latest/user-guide.html#json.moxy)-JSON 默认通过 MOXy 来绑定，并且在 Jersey 2.0 以来的应用程序是支持 JSON 绑定首选方法 。当 JSON MOXy 模块在 类路径,Jersey 将自动发现模块和无缝地支持 JSON 绑定支持通过 MOXy 在应用程序中。(见 4.3 节,“自动发现功能”。
*   [Java API 为 JSON 处理(JSON-P)](https://jersey.java.net/documentation/latest/user-guide.html#json.json-p)
*   [Jackson](https://jersey.java.net/documentation/latest/user-guide.html#json.jackson)
*   [Jettison](https://jersey.java.net/documentation/latest/user-guide.html#json.jettison)

## 9.1.1\. Approaches to JSON Support 支持 JSON 方法

每个上述扩展模块使用一个或多个可用的三种基本方法在处理 JSON 表示:

*   基于 POJO 的 JSON 绑定
*   基于 JAXB 的 JSON 绑定
*   低级的 JSON 解析和处理支持

第一个方法是非常通用的,允许您将任何 Java 对象映射到 JSON,反之亦然。其他两种方法限制你在 Java 类型资源方法可以生产和/或使用。基于 JAXB 方法是有用的,如果你打算使用 JAXB 的某些特性和支持 XML 和 JSON 表示。最后,低级方法给你最好的细粒度控制输出的 JSON 数据格式。

### 9.1.1.1\. POJO support 基于 POJO

POJO 的支持是最简单的方法将 Java 对象转换为 JSON 和转回去。 媒体模块,支持这种方法是 [MOXy](https://jersey.java.net/documentation/latest/user-guide.html#json.moxy) 和 [Jackson](https://jersey.java.net/documentation/latest/user-guide.html#json.jackson)

## 9.1.1.2\. JAXB based JSON support 基于 JAXB

采取这种方法可以节省大量的时间,如果你想轻松地生成/使用 JSON 和 XML 数据格式。与 JAXB bean 你将能够使用相同的 Java 模型生成 JSON 和 XML 表示。与这样一个合作的另一个优点是简单模型和 API 在 Java SE 平台的可用性。 JAXB 使用注解的 POJO,这些可以处理简单的 Java bean。

基于 JAXB 方法的一个缺点可能是如果你需要使用一个非常具体的 JSON 格式。然后可能很难找到一个合适的方法来得到这样一个格式生产和消费。这是一个原因提供了许多配置选项,这样你就可以控制如何 JAXB bean 序列化和反序列化。额外的配置选项但是需要你更详细的了解您所使用的框架。

下面是一个非常简单的例子,来说明 JAXB bean 可能看起来像。

Example 9.1\. Simple JAXB bean implementation

```java
@XmlRootElement
public class MyJaxbBean {
    public String name;
    public int age;

    public MyJaxbBean() {} // JAXB needs this

    public MyJaxbBean(String name, int age) {
        this.name = name;
        this.age = age;
    }
} 
```

使用上面的 JAXB bean 生成 JSON 数据格式资源方法,然后一样简单:

Example 9.2\. JAXB bean used to generate JSON representation

```java
@GET
@Produces("application/json")
public MyJaxbBean getMyBean() {
    return new MyJaxbBean("Agamemnon", 32);
} 
```

注意,JSON @Produces 注释中指定特定的 mime 类型,MyJaxbBean 的方法返回一个实例,JAXB 能够处理。生成的 JSON 在这种情况下会看起来像:

```java
{"name":"Agamemnon", "age":"32"} 
```

正确使用 JAXB 注解本身可以控制一定 JSON 格式输出。具体来说,直接通过使用 JAXB 注释很容易做到重命名和删除属性。例如,下面的例子描述了上述 MyJaxbBean 变化将导致 {"king":"Agamemnon"} JSON 输出。

Example 9.3\. Tweaking JSON format using JAXB

```java
@XmlRootElement
public class MyJaxbBean {

    @XmlElement(name="king")
    public String name;

    @XmlTransient
    public int age;

    // several lines removed
} 
```

媒体模块,支持这种方法是 [MOXy](https://jersey.java.net/documentation/latest/user-guide.html#json.moxy), [Jackson](https://jersey.java.net/documentation/latest/user-guide.html#json.jackson),[Jettison](https://jersey.java.net/documentation/latest/user-guide.html#json.jettison)

### 9.1.1.3\. Low-level based JSON support 低级的 JSON 解析和处理支持

JSON 处理 API 是一个新的标准 API 进行解析和处理 JSON 结构以类似的方式,SAX 和 StAX 解析器提供对 XML 。这个 API 是 Java EE 7 和后来的一部分。另一个 JSON 解析/处理抛弃框架提供的 API。这两种 api 提供一个低级访问生产和消费 JSON 数据结构。采用这种低级的方法你会使用 JsonObject(或 JsonObject) 和/或 JsonArray (或分别 JsonArray )类在处理 JSON 数据表示。

这些低级 api 的最大优势是,你会得到完全控制和消费产生的 JSON 格式。你也能够生产和消费非常大的 JSON 结构使用流 JSON 解析器/生成器 api。另一方面,处理您的数据模型对象可能会更复杂,相对于 POJO 或基于 JAXB 绑定方法。差异是描述在以下代码片段。

基于 JAXB 绑定方法

Example 9.4\. JAXB bean creation

```java
MyJaxbBean myBean = new MyJaxbBean("Agamemnon", 32); 
```

当你构建一个 JAXB bean 时，JSON 写成 {"name":"Agamemnon", "age":32}

现在构建一个等价的 JsonObject / JsonObject(生成的 JSON 的表达式),您需要几行代码。下面的例子说明了如何构造相同的 JSON 数据使用标准的 Java EE 7 JSON 处理 API。

```java
JsonObject myObject = Json.createObjectBuilder()
        .add("name", "Agamemnon")
        .add("age", 32)
        .build(); 
```

最后看下使用 Jettison 来做同样的事，

Example 9.6\. Constructing a JSONObject (Jettison)

```java
JSONObject myObject = new JSONObject();
try {
    myObject.put("name", "Agamemnon");
    myObject.put("age", 32);
} catch (JSONException ex) {
    LOGGER.log(Level.SEVERE, "Error ...", ex);
} 
```

媒体模块,支持低级 JSON 解析和生成方法是 [Java API for JSON Processing (JSON-P)](https://jersey.java.net/documentation/latest/user-guide.html#json.json-p)和[Jettison](https://jersey.java.net/documentation/latest/user-guide.html#json.jettison)。除非你有强烈的理由使用非标准抛[Jettison](https://jersey.java.net/documentation/latest/user-guide.html#json.jettison) API,我们推荐您使用新标准[Java API for JSON Processing (JSON-P)](https://jersey.java.net/documentation/latest/user-guide.html#json.json-p) API。

## 9.1.2\. MOXy

### 9.1.2.1\. Dependency

需要添加 jersey-media-moxy 依赖库在你的 pom.xml 来使用 MOXy

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-moxy</artifactId>
    <version>2.16</version>
</dependency> 
```

不用 maven 的话，要确保所有需要的库在类路径下，建[jersey-media-moxy](https://jersey.java.net/project-info/2.16/jersey/project/jersey-media-moxy/dependencies.html)

### 9.1.2.2\. Configure and register 配置和注册

如上所述在见 4.3 节,“自动发现功能”以及在本章早些时候,MOXy 模块是您不需要显式地注册它的特性(MoxyJsonFeature)在您的客户端/服务器[配置](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configurable.html)的模块之一，这个特性是自动发现和注册时将 jersey-media-moxy 模块添加到您的类路径。

自动发现的 jersey-media-moxy 模块定义了几个属性,可用于控制自动登记 MoxyJsonFeature(除了通用[CommonProperties.FEATURE_AUTO_DISCOVERY_DISABLE](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/CommonProperties.html#FEATURE_AUTO_DISCOVERY_DISABLE)一个客户机/服务器变量):

*   [CommonProperties.MOXY_JSON_FEATURE_DISABLE](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/CommonProperties.html#MOXY_JSON_FEATURE_DISABLE)
*   [ServerProperties.MOXY_JSON_FEATURE_DISABLE](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/server/ServerProperties.html#MOXY_JSON_FEATURE_DISABLE)
*   [ClientProperties.MOXY_JSON_FEATURE_DISABLE](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/client/ClientProperties.html#MOXY_JSON_FEATURE_DISABLE)

**注意** 手动注册其他 Jersey JSON 提供者功能(除了[Java API for JSON Processing (JSON-P)](https://jersey.java.net/documentation/latest/user-guide.html#json.json-p)) 禁用 MoxyJsonFeature 的自动启用和配置。

配置 MOXy 所提供的[MessageBodyReader](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyReader.html) / [MessageBodyWriter](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyWriter.html) 您可以简单地创建一个 MoxyJsonConfig 实例,并设置必要的属性的值。最常见的属性可以使用一个特定的方法来设置属性的值也可以使用更通用的方法来设置属性:

*   [MoxyJsonConfig#property(java.lang.String, java.lang.Object)](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/moxy/json/MoxyJsonConfig.html#property(java.lang.String, java.lang.Object)) ——设置 Marshaller 和 Unmarshaller 属性值
*   [MoxyJsonConfig#marshallerProperty(java.lang.String, java.lang.Object)](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/moxy/json/MoxyJsonConfig.html#marshallerProperty(java.lang.String, java.lang.Object)) ——设置 Marshaller 属性值
*   [MoxyJsonConfig#unmarshallerProperty(java.lang.String, java.lang.Object)](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/moxy/json/MoxyJsonConfig.html#unmarshallerProperty(java.lang.String, java.lang.Object)) ——设置 Unmarshaller 属性值

Example 9.7\. MoxyJsonConfig - Setting properties.

```java
final Map<String, String> namespacePrefixMapper = new HashMap<String, String>();
namespacePrefixMapper.put("http://www.w3.org/2001/XMLSchema-instance", "xsi");

final MoxyJsonConfig configuration = new MoxyJsonConfig()
        .setNamespacePrefixMapper(namespacePrefixMapper)
        .setNamespaceSeparator(':'); 
```

为了使 [MoxyJsonConfig](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/moxy/json/MoxyJsonConfig.html) 对 MOXy 可见,您需要创建并注册 ContextResolver <t class="hljs-annotation">在您的客户端/服务器的代码。</t>

Example 9.8\. Creating ContextResolver

```java
final Map<String, String> namespacePrefixMapper = new HashMap<String, String>();
namespacePrefixMapper.put("http://www.w3.org/2001/XMLSchema-instance", "xsi");

final MoxyJsonConfig moxyJsonConfig = MoxyJsonConfig()
            .setNamespacePrefixMapper(namespacePrefixMapper)
            .setNamespaceSeparator(':');

final ContextResolver<MoxyJsonConfig> jsonConfigResolver = moxyJsonConfig.resolver(); 
```

配置属性传递给底层 MOXyJsonProvider 的另一种方法是设置直接到您的[配置](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configurable.html)实例(参见下面的一个例子)。这些都是被属性设置覆盖到 [MoxyJsonConfig](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/moxy/json/MoxyJsonConfig.html) 。

Example 9.9\. Setting properties for MOXy providers into [Configurable](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configurable.html)

```java
new ResourceConfig()
                            .property(MarshallerProperties.JSON_NAMESPACE_SEPARATOR, ".")
                            // further configuration 
```

当 MOXy 的 [MessageBodyReader](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyReader.html)/ [MessageBodyWriter](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyWriter.html) 被使用，有一些 Jersey 的属性被设置默认值

Table 9.1\. Default property values for MOXy [MessageBodyReader](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyReader.html)/ [MessageBodyWriter](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyWriter.html)

javax.xml.bind.Marshaller#JAXB_FORMATTED_OUTPUT org.eclipse.persistence.jaxb.JAXBContextProperties#JSON_INCLUDE_ROOT false org.eclipse.persistence.jaxb.MarshallerProperties#JSON_MARSHAL_EMPTY_COLLECTIONStrue org.eclipse.persistence.jaxb.JAXBContextProperties#JSON_NAMESPACE_SEPARATORorg.eclipse.persistence.oxm.XMLConstants#DOT

Example 9.10\. Building client with MOXy JSON feature enabled.

```java
final Client client = ClientBuilder.newBuilder()
        // The line below that registers MOXy feature can be
        // omitted if FEATURE_AUTO_DISCOVERY_DISABLE is
        // not disabled.
        .register(MoxyJsonFeature.class)
        .register(jsonConfigResolver)
        .build(); 
```

Example 9.11\. Creating JAX-RS application with MOXy JSON feature enabled.

```java
// Create JAX-RS application.
final Application application = new ResourceConfig()
        .packages("org.glassfish.jersey.examples.jsonmoxy")
        // The line below that registers MOXy feature can be
        // omitted if FEATURE_AUTO_DISCOVERY_DISABLE is
        // not disabled.
        .register(MoxyJsonFeature.class)
        .register(jsonConfigResolver); 
```

### 9.1.2.3\. Examples

Jersey 提供一个 [JSON MOXy example](https://github.com/jersey/jersey/tree/2.16/examples/json-moxy)如何使用 MOXy 来消费/生成 JSON。

## 8.1.3\. Java API for JSON Processing (JSON-P)

### 8.1.3.1\. Dependency 依赖

使用 JSON-P 作为 JSON 的提供者需要添加 jersey-media-json-processing 模块到 pom.xml 文件:

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-processing</artifactId>
    <version>2.16</version>
</dependency> 
```

如果你不使用 Maven，要确保所有需要的依赖关系(见[jersey-media-json-processing](https://jersey.java.net/project-info/2.16/jersey/project/jersey-media-json-processing/dependencies.html))到类的路径。

### 9.1.3.2\. Configure and register 配置和注册

正如见 4.3 节,“自动发现功能”中提到的,JSON-Processing 模块,您不需要显式地注册它的特性(JsonProcessingFeature)在您的客户端/服务器[配置](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configurable.html)，这个特性是将 jersey-media-json-processing 模块添加到您的类路径中时自动发现和注册时。

至于其他模块,jersey-media-json-processing 还几个属性,会影响 JsonProcessingFeature 的注册 (除了[CommonProperties.FEATURE_AUTO_DISCOVERY_DISABLE](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/CommonProperties.html#FEATURE_AUTO_DISCOVERY_DISABLE)等):

*   [CommonProperties.JSON_PROCESSING_FEATURE_DISABLE](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/CommonProperties.html#JSON_PROCESSING_FEATURE_DISABLE)
*   [ServerProperties.JSON_PROCESSING_FEATURE_DISABLE](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/server/ServerProperties.html#JSON_PROCESSING_FEATURE_DISABLE)
*   [ClientProperties.JSON_PROCESSING_FEATURE_DISABLE](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/client/ClientProperties.html#JSON_PROCESSING_FEATURE_DISABLE)

JSON-P 提供配置 [MessageBodyReader](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyReader.html)/[MessageBodyWriter](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyWriter.html) ，只需提供支持的属性值添加到[配置](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configuration.html)实例(客户机/服务器)。目前支持这些属性:

*   JsonGenerator.PRETTY_PRINTING ("javax.json.stream.JsonGenerator.prettyPrinting")

Example 9.12\. Building client with JSON-Processing JSON feature enabled.

```java
ClientBuilder.newClient(new ClientConfig()
        // The line below that registers JSON-Processing feature can be
        // omitted if FEATURE_AUTO_DISCOVERY_DISABLE is not disabled.
        .register(JsonProcessingFeature.class)
        .property(JsonGenerator.PRETTY_PRINTING, true)
); 
```

Example 9.13\. Creating JAX-RS application with JSON-Processing JSON feature enabled.

```java
// Create JAX-RS application.
final Application application = new ResourceConfig()
        // The line below that registers JSON-Processing feature can be
        // omitted if FEATURE_AUTO_DISCOVERY_DISABLE is not disabled.
        .register(JsonProcessingFeature.class)
        .packages("org.glassfish.jersey.examples.jsonp")
        .property(JsonGenerator.PRETTY_PRINTING, true); 
```

### 9.1.3.3\. Examples

Jersey 提供了一个[JSON Processing 实例](https://github.com/jersey/jersey/tree/2.16/examples/json-processing-webapp)如何使用 JSON-Processing 处理消费/生成 JSON。

## 9.1.4\. Jackson (1.x and 2.x)

### 9.1.4.1\. Dependency 依赖

使用 Jackson 2.x 需添加 jersey-media-json-jackson 模块到 pom.xml:

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.16</version>
</dependency> 
```

使用 Jackson 1.x 用法如下:

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson1</artifactId>
    <version>2.16</version>
</dependency> 
```

如果你不使用 Maven，要确保所有需要的依赖关系(见 [jersey-media-json-jackson](https://jersey.java.net/project-info/2.16/jersey/project/jersey-media-json-jackson/dependencies.html) 或 [jersey-media-json-jackson](https://jersey.java.net/project-info/2.16/jersey/project/jersey-media-json-jackson/dependencies.html)) )到类的路径。

### 9.1.4.2\. Configure and register 配置和注册

**注意**

注意,不同的名称空间，Jackson 1.x (org.codehaus.jackson) 和 Jackson 2.x (com.fasterxml.jackson)

Jackson JSON 处理器可以通过提供一个自定义 Jackson 2 的[ObjectMapper](http://fasterxml.github.io/jackson-databind/javadoc/2.3.0/com/fasterxml/jackson/databind/ObjectMapper.html) (或者 Jackson 1 的 [ObjectMapper](http://jackson.codehaus.org/1.9.9/javadoc/org/codehaus/jackson/map/ObjectMapper.html) ) 实例来控制。这可能是方便的,如果你需要重新定义默认 Jackson 行为和调整你的 JSON 数据结构。Jackson 的所有特性的详细描述了本指南的范围。下面的例子给你一个提示如何写 ObjectMapper ([ObjectMapper](http://jackson.codehaus.org/1.9.9/javadoc/org/codehaus/jackson/map/ObjectMapper.html))实例 到你的 Jersey 的应用程序。

如果需要,在你的 [配置](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configurable.html)(客户机/服务器)中，为了使用 Jackson 作为 JSON(JAXB/POJO)提供者需要给 ObjectMapper 注册[JacksonFeature](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/jackson/JacksonFeature.html)([Jackson1Feature](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/jackson1/Jackson1Feature.html))和 ContextResolver<t class="hljs-annotation">。</t>

Example 9.14\. ContextResolver<objectmapper class="hljs-annotation"></objectmapper>

```java
@Provider
public class MyObjectMapperProvider implements ContextResolver<ObjectMapper> {

    final ObjectMapper defaultObjectMapper;

    public MyObjectMapperProvider() {
        defaultObjectMapper = createDefaultMapper();
    }

    @Override
    public ObjectMapper getContext(Class<?> type) {
            return defaultObjectMapper;
        }
    }

    private static ObjectMapper createDefaultMapper() {
        final ObjectMapper result = new ObjectMapper();
        result.configure(Feature.INDENT_OUTPUT, true);

        return result;
    }

    // ...
} 
```

完整示例，见 来自 [JSON-Jackson](https://github.com/jersey/jersey/tree/2.16/examples/json-jackson) 例子中的 [MyObjectMapperProvider](https://github.com/jersey/jersey/tree/2.16/examples/json-jackson/src/main/java/org/glassfish/jersey/examples/jackson/MyObjectMapperProvider.java) 类.

Example 9.15\. Building client with Jackson JSON feature enabled.

final Client client = ClientBuilder.newBuilder() .register(MyObjectMapperProvider.class) //无特殊要求无需注册这个 .register(JacksonFeature.class) .build();

Example 9.16\. Creating JAX-RS application with Jackson JSON feature enabled.

// Create JAX-RS application. final Application application = new ResourceConfig() .packages("org.glassfish.jersey.examples.jackson") .register(MyObjectMapperProvider.class) //无特殊要求无需注册这个 .register(JacksonFeature.class);

### 9.1.4.3\. Examples

Jersey 提供 [JSON Jackson (2.x) 的例子](https://github.com/jersey/jersey/tree/2.16/examples/json-jackson)和 [JSON Jackson (1.x) 例子](https://github.com/jersey/jersey/tree/2.16/examples/json-jackson1) 展示如何使用 Jackson 消费/生成 JSON。

## 9.1.5\. Jettison

Jettison 模块提供 (反)序列化 JSON 的 JAXB 方法，除了使用纯 JAXB,配置选项可以设置在一个[JettisonConfig](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/jettison/JettisonConfig.html) 实例。然后实例可以进一步用于创建 [JettisonJaxbContext](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/jettison/JettisonJaxbContext.html),作为主要的配置点。通过你的专业 JettisonJaxbContext to Jersey,你将最终需要实现一个 JAXBContext [ContextResolver](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/ContextResolver.html) ((见下文)。

### 9.1.5.1\. Dependency 依赖

如果使用 Jettison 需要添加 jersey-media-json-jettison 模块到 pom.xml :

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jettison</artifactId>
    <version>2.16</version>
</dependency> 
```

如果没有使用 Maven ，确保所有依赖库 (详见 [jersey-media-json-jettison](https://jersey.java.net/project-info/2.13/jersey/project/jersey-media-json-jettison/dependencies.html)) 在 classpath 中.

### 9.1.5.2\. JSON Notations 符号

JettisonConfig 允许你使用两种 JSON 符号，每种序列化 JSON 的方式是不同的。下面是支持符号的列表：

*   JETTISON_MAPPED (默认符号)
*   BADGERFISH

在处理更复杂的 XML 文档，你可能想要使用这些符号。即当你在 JAXB bean 处理多个 XML 名称空间。

独立的符号及其进一步的配置选项如下所述。而不是解释规则映射 XML 结构转换为 JSON,描述的符号将使用一个简单的例子。以下是 JAXB bean,它将被使用。

Example 9.17\. JAXB beans for JSON supported notations description, simple address bean

```java
@XmlRootElement
public class Address {
    public String street;
    public String town;

    public Address(){}

    public Address(String street, String town) {
        this.street = street;
        this.town = town;
    }
} 
```

Example 9.18\. JAXB beans for JSON supported notations description, contact bean

```java
@XmlRootElement
public class Contact {

    public int id;
    public String name;
    public List<Address> addresses;

    public Contact() {};

    public Contact(int id, String name, List<Address> addresses) {
        this.name = name;
        this.id = id;
        this.addresses =
            (addresses != null) ? new LinkedList<Address>(addresses) : null;
    }
} 
```

以下文本主要工作是 contact bean 初始化:

Example 9.19\. JAXB beans for JSON supported notations description, initialization

```java
Address[] addresses = {new Address("Long Street 1", "Short Village")};
Contact contact = new Contact(2, "Bob", Arrays.asList(addresses)); 
```

例子中 contact bean 的 id=2, name="Bob" 包含一个 address (street="Long Street 1", town="Short Village").

下面所有的配置选项描述的记录也在 [JettisonConfig](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/jettison/JettisonConfig.html) api 文档

#### 9.1.5.2.1\. Jettison mapped notation 隐射符号

如果你需要处理各种 XML 名称空间,你会发现 Jettison 映射符号非常有用。允许定义一个特定名称空间 id 项:

```java
...
@XmlElement(namespace="http://example.com")
public int id;
... 
```

然后你只需配置从 XML 名称空间映射到 JSON 前缀如下:

Example 9.20\. XML namespace to JSON mapping configuration for Jettison based mapped notation

```java
Map<String,String> ns2json = new HashMap<String, String>();
ns2json.put("http://example.com", "example");
context = new JettisonJaxbContext(
    JettisonConfig.mappedJettison().xml2JsonNs(ns2json).build(),
    types); 
```

JSON 的结果就像下面的例子.

Example 9.21\. JSON expression with XML namespaces mapped into JSON

```java
{
   "contact":{
      "example.id":2,
      "name":"Bob",
      "addresses":{
         "street":"Long Street 1",
         "town":"Short Village"
      }
   }
} 
```

请注意,该 id 项变成了 example.id 基于 XML 名称空间映射的 id。如果你有更多的 XML 名称空间的 XML ,您需要为所有这些配置合适的映射。

Jersey 版本 2.2 中引入另一个可配置的选项与序列化 JSON 数组与 Jettison 的映射的符号。当序列化元素代表单项列表/数组时,您可能想要使用以下 Jersey 配置方法来显式地名称元素将其视为数组不管实际内容是什么。

Example 9.22\. JSON Array configuration for Jettison based mapped notation

```java
context = new JettisonJaxbContext(
    JettisonConfig.mappedJettison().serializeAsArray("name").build(),
    types); 
```

JSON 结果想下面例子，不重要的行已经删除

Example 9.23\. JSON expression with JSON arrays explicitly configured via Jersey

```java
{
   "contact":{
      ...
      "name":["Bob"],
      ...
   }
} 
```

#### 9.1.5.2.2\. Badgerfish notation

从 JSON 和 JavaScript 的角度来看,这种表示法绝对是最可读的。您可能不希望使用它,除非你需要确保你的 JAXB bean 可以完美地读写和 JSON ,无需顾及任何格式配置中,名称空间等。

JettisonConfig 使用 badgerfish 符号可以通过下面语句创建

```java
JettisonConfig.badgerFish().build() 
```

JSON 输出如下：

Example 9.24\. JSON expression produced using badgerfish notation

```java
{
   "contact":{
      "id":{
         "$":"2"
      },
      "name":{
         "$":"Bob"
      },
      "addresses":{
         "street":{
            "$":"Long Street 1"
         },
         "town":{
            "$":"Short Village"
         }
      }
   }
} 
```

### 9.1.5.3\. Configure and register 配置和注册

若使用 Jettison 为你的 JSON (JAXB/POJO) 提供者，需给 JAXBContext（如果需要） 注册 [JettisonFeature](https://jersey.java.net/apidocs/2.13/jersey/org/glassfish/jersey/jettison/JettisonFeature.html) 和 ContextResolver <t class="hljs-annotation">到 在你的[配置](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/core/Configurable.html) (client/server).</t>

Example 9.25\. ContextResolver<objectmapper class="hljs-annotation"></objectmapper>

```java
@Provider
public class JaxbContextResolver implements ContextResolver<JAXBContext> {

    private final JAXBContext context;
    private final Set<Class<?>> types;
    private final Class<?>[] cTypes = {Flights.class, FlightType.class, AircraftType.class};

    public JaxbContextResolver() throws Exception {
        this.types = new HashSet<Class<?>>(Arrays.asList(cTypes));
        this.context = new JettisonJaxbContext(JettisonConfig.DEFAULT, cTypes);
    }

    @Override
    public JAXBContext getContext(Class<?> objectType) {
        return (types.contains(objectType)) ? context : null;
    }
} 
```

Example 9.26\. Building client with Jettison JSON feature enabled.

```java
final Client client = ClientBuilder.newBuilder()
        .register(JaxbContextResolver.class)  // No need to register this provider if no special configuration is required.
        .register(JettisonFeature.class)
        .build(); 
```

Example 9.27\. Creating JAX-RS application with Jettison JSON feature enabled.

```java
// Create JAX-RS application.
final Application application = new ResourceConfig()
        .packages("org.glassfish.jersey.examples.jettison")
        .register(JaxbContextResolver.class)  // No need to register this provider if no special configuration is required.
        .register(JettisonFeature.class); 
```

##### 9.1.5.4\. Examples 例子

Jersey 提供 [JSON Jettison 的例子](https://github.com/jersey/jersey/tree/2.16/examples/json-jettison).

## 8.1.6\. @JSONP - JSON with Padding Support

Jersey 提供 开箱即用的支持 [JSONP](http://en.wikipedia.org/wiki/JSONP) - JSON with Padding。以下条件必须满足利用此功能:

*   资源的方法,它应该返回 JSON,包装需要由 [@JSONP](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/server/JSONP.html) 注释的。
*   [MessageBodyWriter](http://jax-rs-spec.java.net/nonav/$%7Bjaxrs.api.version%7D/apidocs/javax/ws/rs/ext/MessageBodyWriter.html) application/json 媒体类型,也接受资源方法的返回类型,需要注册(见本章[JSON](https://jersey.java.net/documentation/latest/media.html#json)部分)。
*   用户的请求必须包含 Accept 标头的 JavaScript 定义媒体类型(见下文)。

可接受的媒体类型兼容 @JSONP 是:pplication/javascript, application/x-javascript, application/ecmascript, text/javascript, text/x-javascript, text/ecmascript, text/jscript.

Example 9.28\. Simplest case of using @JSONP

```java
@GET
@JSONP
@Produces({"application/json", "application/javascript"})
public JaxbBean getSimpleJSONP() {
    return new JaxbBean("jsonp");
} 
```

假设我们有注册一个 JSON 提供者和 JaxbBean 看起来像:

Example 9.29\. JaxbBean for @JSONP example

```java
@XmlRootElement
public class JaxbBean {

    private String value;

    public JaxbBean() {}

    public JaxbBean(final String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    public void setValue(final String value) {
        this.value = value;
    }
} 
```

当你发送一个 GET 请求接受标题设置为 application/javascript 你会得到一个结果实体看起来像:

```java
callback({
    "value" : "jsonp",
}) 
```

当然,方法配置包装方法返回的实体默认回调可以看到在前面的例子。@JSONP 有两个参数,可以配置:回调,queryParam。回调的名称代表 JavaScript 应用程序定义的回调函数。queryParam,第二个参数定义的名称查询参数的回调函数的名称使用在请求(如果存在)。queryParam 值默认为 __callback,所以即使你不自己设置查询参数的名称,客户总是可以影响结果包装 JavaScript 回调方法的名称。

**注意**

queryParam 值(如果设置)总是优先于回调函数值。

稍微改下代码

Example 9.30\. Example of @JSONP with configured parameters.

```java
@GET
@Produces({"application/json", "application/javascript"})
@JSONP(callback = "eval", queryParam = "jsonpCallback")
public JaxbBean getSimpleJSONP() {
    return new JaxbBean("jsonp");
} 
```

两次提交:

> curl -X GET [`localhost:8080/jsonp`](http://localhost:8080/jsonp)

将返回

```java
eval({
    "value" : "jsonp",
}) 
```

以及

> curl -X GET [`localhost:8080/jsonp?jsonpCallback=alert`](http://localhost:8080/jsonp?jsonpCallback=alert)

将返回

```java
alert({
    "value" : "jsonp",
}) 
```

Example. 这里提供[示例](https://github.com/jersey/jersey/tree/2.16/examples/json-with-padding).

# 9.2 XML

# 9.2\. XML

正如您可能已经知道,Jersey 使用 MessageBodyWriter <t class="hljs-annotation">和 MessageBodyReader <t class="hljs-annotation">年代来解析传入的请求和创建传出的响应。每个用户都可以创建 自己的表现但是…这是不建议这样做。XML 是证明交换信息的标准,特别是在 web 服务。Jersey 支持低水平数据类型用于直接操作和 JAXB XML 实体。</t></t>

## 9.2.1\. Low level XML support 低级 XML 支持

Jersey 目前支持一些低水平的数据类型:[StreamSource](http://docs.oracle.com/javase/7/docs/api/javax/xml/transform/stream/StreamSource.html), [SAXSource](http://docs.oracle.com/javase/7/docs/api/javax/xml/transform/sax/SAXSource.html), [DOMSource](http://docs.oracle.com/javase/7/docs/api/javax/xml/transform/dom/DOMSource.html) 和 [Document](http://docs.oracle.com/javase/7/docs/api/org/w3c/dom/Document.html)。您可以使用这些类型的返回类型或方法(资源)参数。让说我们想要测试这个功能,我们有 [helloworld 示例](https://github.com/jersey/jersey/tree/2.16/examples/helloworld) 作为起点。所有我们需要做的就是添加方法(资源)的消耗和产生的 XML 和类型将使用上面提到的。

Example 8.31\. Low level XML test - methods added to HelloWorldResource.java

```java
@POST
@Path("StreamSource")
public StreamSource getStreamSource(StreamSource streamSource) {
    return streamSource;
}

@POST
@Path("SAXSource")
public SAXSource getSAXSource(SAXSource saxSource) {
    return saxSource;
}

@POST
@Path("DOMSource")
public DOMSource getDOMSource(DOMSource domSource) {
    return domSource;
}

@POST
@Path("Document")
public Document getDocument(Document document) {
    return document;
} 
```

MessageBodyWriter <t class="hljs-annotation">和 MessageBodyReader <t class="hljs-annotation">都在这个例子中使用了,我们需要的是一个 POST 请求 XML 文档作为一个请求的实体。让这只尽可能简单的根元素没有内容将发送:“<test class="hljs-annotation">”。您可以创建 JAX-RS 客户端,或使用其他一些工具,例如 curl:</test></t></t>

> curl -v [`localhost:8080/base/helloworld/StreamSource`](http://localhost:8080/base/helloworld/StreamSource) -d "<test class="hljs-annotation">"</test>

你应该从我们的服务得到完全相同的 XML ;在本例中,XML 头添加到响应但内容停留。自由的遍历所有资源。

## 9.2.2\. Getting started with JAXB 开始

好的开始,人们已经有了一些 JAXB 注解的经验,例子是是 [JAXB 示例](https://github.com/jersey/jersey/tree/2.16/examples/jaxb)。你可以看到不同的用例。本文主要是针对那些没有 JAXB 经验。别指望所有可能的注释和他们的组合将在这一章,[JAXB(JSR 222 实现)](http://jaxb.java.net/)是相当复杂和全面。但如果你只是想知道如何与 REST 服务交换 XML 消息,你看着合适的章节。

可以从简单的例子开始。让我们说我们有类 Planet 和服务生产的“Planets”。

Example 9.32\. Planet class

```java
@XmlRootElement
public class Planet {
    public int id;
    public String name;
    public double radius;
} 
```

Example 9.33\. Resource class

```java
@Path("planet")
public class Resource {

    @GET
    @Produces(MediaType.APPLICATION_XML)
    public Planet getPlanet() {
        final Planet planet = new Planet();

        planet.id = 1;
        planet.name = "Earth";
        planet.radius = 1.0;

        return planet;
    }
} 
```

你可以看到有一些额外的注释声明类 Planet,尤其是[@XmlRootElement](http://jaxb.java.net/nonav/2.2.7/docs/api/javax/xml/bind/annotation/XmlRootElement.html)。这是一个 JAXB 注释的 java 类映射到 XML 元素。我们不需要指定任何其他,因为 Planet 非常简单的类,所有的字段都是公开的。在这种情况下,XML 元素名称将派生类名或你可以设置名称属性:@XmlRootElement(name="yourName")。

我们的资源类将响应 GET/planet 请求

```java
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<planet>
    <id>1</id>
    <name>Earth</name>
    <radius>1.0</radius>
</planet> 
```

这可能正是我们想要的……与否。或者我们可能不关心,因为我们可以使用 JAX-RS 客户端发出请求该资源,这很容易:

```java
Planet planet = webTarget.path("planet").request(MediaType.APPLICATION_XML_TYPE).get(Planet.class); 
```

有预先创建 WebTarget 对象指向我们的应用程序的上下文根,只需添加路径(在我们的例子中是 planet),接收 header(不是强制性的,但服务可以提供不同的内容基于这头,例如可以为 text/html 在 web 浏览器),最后我们指定,我们预计 Planet 类通过 GET 请求。

不仅可能需要生成 XML ,我们可能希望使用它。

Example 9.34\. Method for consuming Planet

```java
@POST
@Consumes(MediaType.APPLICATION_XML)
public void setPlanet(Planet planet) {
    System.out.println("setPlanet " + planet);
} 
```

有效的请求后,服务将打印字符串表示的 Planet,可以像 Planet{id=2, name='Mars', radius=1.51}。通过 JAX-RS 客户端你能做到:

```java
webTarget.path("planet").post(planet); 
```

如果有需要其他(非默认的) XML 表示,其他 JAXB 注解需要被使用。简化这一过程通常是由从 XML 模式生成 java 源代码是通过 XML 到 java 编译器和它的 xjc 是 JAXB 的一部分。

## 9.2.3\. POJOs

有时，你不能或者不想在代码里面使用注解，但又想用消费和生成 XML 的类的表现形式。这种情况下就可以用 [JAXBElement](http://jaxb.java.net/nonav/2.2.7/docs/api/javax/xml/bind/JAXBElement.html) 。下面例子就是没有用 [@XmlRootElement](http://jaxb.java.net/nonav/2.2.7/docs/api/javax/xml/bind/annotation/XmlRootElement.html) 注解：

Example 9.35\. Resource class - JAXBElement

```java
@Path("planet")
public class Resource {

    @GET
    @Produces(MediaType.APPLICATION_XML)
    public JAXBElement<Planet> getPlanet() {
        Planet planet = new Planet();

        planet.id = 1;
        planet.name = "Earth";
        planet.radius = 1.0;

        return new JAXBElement<Planet>(new QName("planet"), Planet.class, planet);
    }

    @POST
    @Consumes(MediaType.APPLICATION_XML)
    public void setPlanet(JAXBElement<Planet> planet) {
        System.out.println("setPlanet " + planet.getValue());
    }
} 
```

正如您可以看到的,一切都是用了 JAXBElement 就会复杂一些。这是因为现在需要显式地设置元素名称的给 Planet 类的 XML 表示。客户端比服务器端更加复杂,因为你不能做 `JAXBElement<Planet>` 所以 JAX-RS 客户端 API 提供了如何通过 声明 `GenericType<T>` 的子类 解决它

Example 9.36\. Client side - JAXBElement

```java
// GET
GenericType<JAXBElement<Planet>> planetType = new GenericType<JAXBElement<Planet>>() {};

Planet planet = (Planet) webTarget.path("planet").request(MediaType.APPLICATION_XML_TYPE).get(planetType).getValue();
System.out.println("### " + planet);

// POST
planet = new Planet();

// ...

webTarget.path("planet").post(new JAXBElement<Planet>(new QName("planet"), Planet.class, planet)); 
```

## 9.2.4\. Using custom JAXBContext 使用自定义 JAXBContext

有些场景适合使用自定义 [JAXBContext](http://jaxb.java.net/nonav/2.2.7/docs/api/javax/xml/bind/JAXBContext.html)。JAXBContext 的创建是一个昂贵的操作，如果你已经创建了一个，相同的实例被 Jersey 使用。其他可能使用的情况是当你需要给 JAXBContext 建立一些特定的东西，例如设置不同的类装载器。

Example 9.37\. PlanetJAXBContextProvider

```java
@Provider
public class PlanetJAXBContextProvider implements ContextResolver<JAXBContext> {
    private JAXBContext context = null;

    public JAXBContext getContext(Class<?> type) {
        if (type != Planet.class) {
            return null; // we don't support nothing else than Planet
        }

        if (context == null) {
            try {
                context = JAXBContext.newInstance(Planet.class);
            } catch (JAXBException e) {
                // log warning/error; null will be returned which indicates that this
                // provider won't/can't be used.
            }
        }

        return context;
    }
} 
```

上面示例简单创建 JAXBContext 的过程,所有你需要做的就是把这个`@Provider` 注释放上，这样 Jersey 就能找到它。用户有时在客户端使用 provider （提供者）类 出问题,所以只是为了提醒——你必须 在客户端配置(客户端做任何事情不像通过服务器包扫描)声明他们。

Example 9.38\. Using Provider with JAX-RS client

```java
ClientConfig config = new ClientConfig();
config.register(PlanetJAXBContextProvider.class);

Client client = ClientBuilder.newClient(config); 
```

## 9.2.5\. MOXy

如果你想使用 [MOXy](http://www.eclipse.org/eclipselink/moxy.php) 作为 JAXB 实现而不是 JAXB RI 您有两种选择。您可以使用标准的 JAXB 机制来定义 从 JAXBContext 实例将获得(有关此主题的更多信息,读 JavaDoc [JAXBContext](http://jaxb.java.net/nonav/2.2.7/docs/api/javax/xml/bind/JAXBContext.html))的 JAXBContextFactory 或者你可以将 jersey-media-moxy 模块添加到您的项目和注册/配置 [MoxyXmlFeature](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/moxy/xml/MoxyXmlFeature.html) 类/实例的[Configurable](http://jax-rs-spec.java.net/nonav/2.0/apidocs/javax/ws/rs/core/Configurable.html)。

Example 9.39\. Add jersey-media-moxy dependency.

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-moxy</artifactId>
    <version>2.16</version>
</dependency> 
```

Example 9.40\. Register the MoxyXmlFeature class.

```java
final ResourceConfig config = new ResourceConfig()
.packages("org.glassfish.jersey.examples.xmlmoxy")
.register(MoxyXmlFeature.class); 
```

Example 9.41\. Configure and register an MoxyXmlFeature instance.

```java
// Configure Properties.
final Map<String, Object> properties = new HashMap<String, Object>();
// ...

// Obtain a ClassLoader you want to use.
final ClassLoader classLoader = Thread.currentThread().getContextClassLoader();

final ResourceConfig config = new ResourceConfig()
    .packages("org.glassfish.jersey.examples.xmlmoxy")
    .register(new MoxyXmlFeature(
        properties,
        classLoader,
        true, // Flag to determine whether eclipselink-oxm.xml file should be used for lookup.
        CustomClassA.class, CustomClassB.class  // Classes to be bound.
    )); 
```

# 9.3 Multipart

# 9.3\. Multipart

## 9.3.1\. Overview 概述

在 JAX-RS 运行环境,这个模块中的类提供了 multipart/* 请求和响应体的集成。注册提供者的集合是为杠杆，在这样的一个消息体部分的内容类型重用同一 MessageBodyReader<t class="hljs-annotation">/MessageBodyWriter <t class="hljs-annotation">实现将用于该内容类型作为一个独立的实体。</t></t>

下面列出的是目前支持常见的 MIME MultiPart ：

*   MIME-Version： 1.0 HTTP header 包含在生成的响应中。这是可以接受的，但在处理请求中不是必需的。
*   [MessageBodyReader](http://jax-rs-spec.java.net/nonav/2.0/apidocs/javax/ws/rs/ext/MessageBodyReader.html) 实现，用于消耗 MIME MultiPart 实体。
*   `MessageBodyWriter<T>` 实现用于产生 MIME MultiPart 实体。适当的 `@Provider` 是基于媒体类型，用于序列化响应体的每个部分。
*   如果不是已经存在，在平常的 Content-Type header 创建一个可选的适当的边界参数 。

更多信息，见 [Multi Part](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/package-summary.html)

### 9.3.1.1\. Dependency 依赖

添加 jersey-media-multipart 到 pom.xml

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-multipart</artifactId>
    <version>2.16</version>
</dependency> 
```

如果你不使用 Maven,确保有所有需要的依赖（见[jersey-media-multipart](https://jersey.java.net/project-info/2.16/jersey/project/jersey-media-multipart/dependencies.html)）在类路径

### 9.3.1.2\. Registration 注册

为了在客户端/服务端代码 使用 jersey-media-multipart 模块的功能，先注册 MultiPartFeature

Example 9.42\. Building client with MultiPart feature enabled.

```java
final Client client = ClientBuilder.newBuilder()
    .register(MultiPartFeature.class)
    .build(); 
```

Example 9.43\. Creating JAX-RS application with MultiPart feature enabled.

```java
// Create JAX-RS application.
final Application application = new ResourceConfig()
    .packages("org.glassfish.jersey.examples.multipart")
    .register(MultiPartFeature.class) 
```

### 9.3.1.3\. Examples 实例

见 [Multipart Web Application Example](https://github.com/jersey/jersey/tree/2.16/examples/multipart-webapp)

## 9.3.2\. Client 客户端

[MultiPart](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/MultiPart.html) 类（或子类）可以当做实体指向使用 jersey-media-multipart 的模块在客户端。这个类 表现为 [MIME multipart 消息](http://en.wikipedia.org/wiki/MIME#Multipart_messages) 并且能够容纳任意数量的[BodyPart](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/BodyPart.html)。 MultiPart 实体默认的媒体类型 multipart/mixed，而 BodyPart 是 text/plain 。

Example 9.44\. MultiPart entity

```java
final MultiPart multiPartEntity = new MultiPart()
        .bodyPart(new BodyPart().entity("hello"))
        .bodyPart(new BodyPart(new JaxbBean("xml"), MediaType.APPLICATION_XML_TYPE))
        .bodyPart(new BodyPart(new JaxbBean("json"), MediaType.APPLICATION_JSON_TYPE));

final WebTarget target = // Create WebTarget.
final Response response = target
        .request()
        .post(Entity.entity(multiPartEntity, multiPartEntity.getMediaType())); 
```

如果发送 multiPartEntity 到服务端，实体的 Content-Type header 在 HTTP message 就像下面那样：（别忘了注册 JSON 提供者）

Example 9.45\. MultiPart entity in HTTP message.

```java
Content-Type: multipart/mixed; boundary=Boundary_1_829077776_1369128119878

--Boundary_1_829077776_1369128119878
Content-Type: text/plain

hello
--Boundary_1_829077776_1369128119878
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8" standalone="yes"?><jaxbBean><value>xml</value></jaxbBean>
--Boundary_1_829077776_1369128119878
Content-Type: application/json

{"value":"json"}
--Boundary_1_829077776_1369128119878-- 
```

当涉及到 form 表单时，（例如媒体类型 multipart/form-data）且有多个字段，有一个更方便使用的类- [FormDataMultiPart](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/FormDataMultiPart.html)。它会自动设置为 FormDataMultiPart 实体 的媒体类型为 multipart/form-data 及 Content-Disposition 报头 为 FormDataBodyPart 。

Example 9.46\. FormDataMultiPart entity

```java
final FormDataMultiPart multipart = new FormDataMultiPart()
    .field("hello", "hello")
    .field("xml", new JaxbBean("xml"))
    .field("json", new JaxbBean("json"), MediaType.APPLICATION_JSON_TYPE);

final WebTarget target = // Create WebTarget.
final Response response = target.request().post(Entity.entity(multipart, multipart.getMediaType())); 
```

为了说明 使用 FormDataMultiPart 替换 FormDataBodyPart 不同点，可以看下 FormDataMultiPart 的 HTML 消息中的 实体：

Example 9.47\. FormDataMultiPart entity in HTTP message.

```java
Content-Type: multipart/form-data; boundary=Boundary_1_511262261_1369143433608

--Boundary_1_511262261_1369143433608
Content-Type: text/plain
Content-Disposition: form-data; name="hello"

hello
--Boundary_1_511262261_1369143433608
Content-Type: application/xml
Content-Disposition: form-data; name="xml"

<?xml version="1.0" encoding="UTF-8" standalone="yes"?><jaxbBean><value>xml</value></jaxbBean>
--Boundary_1_511262261_1369143433608
Content-Type: application/json
Content-Disposition: form-data; name="json"

{"value":"json"}
--Boundary_1_511262261_1369143433608-- 
```

对于许多用户来说常见的情况是从客户端向服务器发送文件。为了这个目的，你可以使用来自 org.glassfish.jersey.jersey.media.multipart 包类，如 [FileDataBodyPart](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/file/FileDataBodyPart.html) 或 [StreamDataBodyPart](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/file/StreamDataBodyPart.html)

Example 9.48\. Multipart - sending files.

```java
// MediaType of the body part will be derived from the file.
final FileDataBodyPart filePart = new FileDataBodyPart("my_pom", new File("pom.xml"));

final FormDataMultiPart multipart = new FormDataMultiPart()
    .field("foo", "bar")
    .bodyPart(filePart);

final WebTarget target = // Create WebTarget.
final Response response = target.request()
    .post(Entity.entity(multipart, multipart.getMediaType())); 
```

*警告*

*不要使用 ApacheConnectorProvider 、 GrizzlyConnectorProvider 或者 JettyConnectorProvider 连接器实现 Jersey Multipart features。见 [Header modification issue](https://jersey.java.net/documentation/latest/user-guide.html#connectors.warning)*

## 9.3.3\. Server

从服务器返回一个 multipart 响应到 客户端，跟客户端描述的美誉太大不同。为获得 客户端发送的多个实体的应用中，你可以使用两种方法：

*   注入整个 [MultiPart](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/MultiPart.html) 实体
*   通过[@FormDataParam](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/FormDataParam.html) 注解 ,将请求中特定 form-data multipar 部分注入。

### 9.3.3.1\. Injecting and returning the MultiPart entity

注入和返回 MultiPart 实体

MultiPart 类型的工作方式 与注入/返回其他实体类型不同。Jersey 提供 `MessageBodyReader<T>` 用来读取请求实体，并且注入 这个实体到资源方法的参数中，而 `MessageBodyWriter<T>` 用于实体的输出。 你可以预计,多部分或 FormDataMultiPart(多部分/格式数据媒体类型)对象注入资源的方法。你可以预期 MultiPart 或 FormDataMultiPart (multipart/form-data 媒体类型) 对象用来注入到资源方法中。

Example 9.49\. Resource method using MultiPart as input parameter / return value.

```java
@POST
@Produces("multipart/mixed")
public MultiPart post(final FormDataMultiPart multiPart) {
    return multiPart;
} 
```

### 9.3.3.2\. Injecting with @FormDataParam 通过 @FormDataParam 注入

如果你只是需要 multipart/form-data 请求实体 到资源的 方法中，可以使用 [@FormDataParam](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/FormDataParam.html) 注解。

这个注解结合使用的媒体类型 multipart/form-data 应该包含文件、非 ASCII 数据, 和编译数据的提交和消费形式。

注解的类型参数可以是下列之一(更多详细描述见 javadoc [@FormDataParam](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/media/multipart/FormDataParam.html)):

*   FormDataBodyPart - 参数的值将会是第一个命名的 body 部分或 null 如果这样的 body 部分不存在
*   FormDataBodyPart 的集合-参数的值将会是一个或多个具有相同名称的命名的 body 部位或 null 如果这样的 body 部位不存在。
*   FormDataContentDisposition - 参数的值将被会是第一个命名的 body 部分的内容处理部分或 null 如果这样的 body 部分不存在。
*   FormDataContentDisposition 集合。参数的值将一个或多个内容处理指定的 body 部分使用相同的名称或 null 如果这样命名的 body 部分是不存在的。
*   一种类型的消息体的读者可以给出第一个命名为主体的媒体类型。参数的值将使用给定类型的消息体读者阅读的结果，对指定的媒体类型，以及指定的 body 的一部分作为输入字节。

如果没有指定部分存在,有一个默认值存在用 [@DefaultValue](http://jax-rs-spec.java.net/nonav/2.0/apidocs/javax/ws/rs/DefaultValue.html) 声明，那么媒体类型将被设置为 text/plain 。参数的值将被阅读的结果使用消息体的读者类型 T,媒体类型 text/plain,UTF-8 编码的字节的默认值作为输入。

如果没有消息体读者可用, 那么类型 T 符合类型 `@FormParam` 然后通过`@FormParam`特定处理,在形式参数的值是由读取字节的字符串实例指定的 body 部分使用字符串类型的消息体的读者和媒体类型的 text/plain。

如果没有指定部分表现那么处理执行规定的 @FormParam .

Example 9.50\. Use of @FormDataParam annotation

```java
@POST
@Consumes(MediaType.MULTIPART_FORM_DATA_TYPE)
public String postForm(
    @DefaultValue("true") @FormDataParam("enabled") boolean enabled,
    @FormDataParam("data") FileData bean,
    @FormDataParam("file") InputStream file,
    @FormDataParam("file") FormDataContentDisposition fileDisposition) {

    // ...
} 
```

示例中，服务器消耗 multipart/form-data 请求实体 body ,包含了一个可选的指定的 body 部分，和两个必须的指定的 body 部分数据和文件。

可选部分启动是当做一个 布尔值 处理，如果这部分不再那么值是 true。

数据部分当做 JAXB bean 处理，包含了下面部分的 元数据。

文件部分是上次的文件，处理成 InputStream。从 Content-Disposition header 看到附加信息关于文件 可以通过参数 fileDisposition 访问。

*提示*

*@FormDataParam 注解同样适用于字段*

# 第十章 过滤器和拦截器

# Chapter 10\. Filters and Interceptors 过滤器和拦截器

# 第十一章 异步服务器和客户端

# Chapter 11\. Asynchronous Services and Clients 异步服务器和客户端

# 第二十章 MVC 模板

# Chapter 20\. MVC Templates 模板

Jersey 提供了支持 Model-View-Controller (MVC) 设计模式的扩展。 在 Jersey 组件的上下文中，在 MVC 模式中的 Controller 对应于一个资源类或方法，View 对应绑定到资源类或方法的模板，model 对应从资源方法返回（控制器）的 Java 对象（或 Java Bean）。

*注：从本章的一些段落/例子来自 Paul Sandoz 的 [MVCJ](https://blogs.oracle.com/sandoz/entry/mvcj) 博客。*

在 Jersey 2，基础 MVC API 由两个类组成（在 org.glassfish.jersey.server.mvc 包），可用于绑定模型视图（模板），分别是 [Viewable](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/server/mvc/Viewable.html) 和 [@Template](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/server/mvc/Template.html)。在使用 Jersey MVC 模板支持时,这些类确定哪种方法（显式/隐）。

# 20.1\. Viewable

为了让资源的方法显式地返回对于一个视图模板和数据模型被使用的引用。为此，Jersey 1 引入了 [Viewable](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/server/mvc/Viewable.html) 类，目前也存在于（不同的包下）Jersey 2。见下面 一个简单的例子， Example 20.1, “Using Viewable in a resource class”

Example 20.1\. Using Viewable in a resource class

```java
package com.example;

@Path("foo")
public class Foo {

    @GET
    public Viewable get() {
        return new Viewable("index.foo", "FOO");
    }
} 
```

在这个例子中，foo JAX-RS 资源类是控制器，Viewable 实例提供的数据模型（FOO 字符串）和引用装载进与之关联的视图模板（index.foo）里。

*提示：所有的 HTTP 方法都可以返回 Viewable 实例 。因此，POST 方法可能会返回一个模板引用的模板，产生一个 HTML Form 表单的视图*

# 20.2\. 模板

# 20.2\. @Template

### 20.2.1\. Annotating Resource methods 注释资源的方法

不需要每次都用 Viewable ，如果你想绑定模型到模板上。为了让资源方法可读性更强（为了避免冗长的包装模板参考模型到 Viewable）,你可以简单的通过 [@Template](https://jersey.java.net/apidocs/2.16/jersey/org/glassfish/jersey/server/mvc/Template.html) 注释资源的方法。从前面的例子，我们做下修改，见 Example 20.2, “Using @Template on a resource method”

Example 20.2\. Using @Template on a resource method

```java
package com.example;

@Path("foo")
public class Foo {

    @GET
    @Template("index.foo")
    public String get() {
        return "FOO";
    }
} 
```

在这个例子中，Foo JAX-RS 资源类仍然是上一节中的 控制器，但是 模型现在是返回注解资源的方法。

这种方法的处理基本上是和 放回一个 Viewable 类实例是相同的。如果一个方法是`@Template`注解的，也可返回 Viewable 类实例，Viewable 类实例将优先于那些在注释的定义。产生的媒体类型，是 Viewable 还是 @Template 将有方法或者类级别的`@Produces`注解决定。

### 20.2.2\. Annotating Resource classes 注解资源类

资源类可以隐式地通过`@Template`注解来与模板进行关联。见 Example 20.3, “Using @Template on a resource class”

Example 20.3\. Using @Template on a resource class

```java
@Path("foo")
@Template
public class Foo {

    public String getFoo() {
        return "FOO";
    }
} 
```

这个例子需要更多的解释是这样的。首先，你可能已经注意到，没有定义资源的方法为 JAX-RS 资源。同时，没有被定义的模板引用。在这种情况下，由于`@Template` 注释放在资源类中不包含任何信息，默认模板将使用相对引用 index（详见 20.3 节，20.3\. Absolute vs. Relative template reference）。对于缺少资源的方法，默认的 `@GET` 方法将自动生成的 Foo 资源（现在是 MVC 的控制器）。生成的资源的方法执行与下列显示资源的方法的实现是等效的：

```java
@GET
public Viewable get() {
    return new Viewable("index", this);
} 
```

可见，资源类充当了 model 的角色。产生媒体类型是由 声明在资源类的 `@Produces`注解决定的，如果需要的话。

*注意：在这种基于资源类的隐式的 MVC 视图模板中，控制器同时也是模型。在这种情况下，模板引用 index 是特殊的，它的模板引用关联的是控制器实例本身。*

下面例子，MVC 控制器以 JAX-RS @GET 子资源方法来表示，同时也可以在资源类中注明 `@Template`来生成

```java
@GET
@Path("{implicit-view-path-parameter}")
public Viewable get(@PathParameter("{implicit-view-path-parameter}") String template) {
    return new Viewable(template, this);
} 
```

这允许 Jersey 来支持隐式的子资源模板。举例，一个在 `foo/bar` 路径的 JAX-RS 将视图使用相对模板引用 bar ,分解为 绝对模板引用 `/com/foo/Foo/bar`

换句话说，一个 HTTP GET 请求`/foo/bar`会通过 Foo 资源方法自动处理产生并将请求转到注册模板处理器来支持绝对参考引用 /com/foo/Foo/bar，其中模型仍然是相同的 JAX-RS 资源类 Foo 的一个实例。

# 20.3\. 绝对 vs. 相对模块引用

如在上一节讨论 [@Template](https://jersey.java.net/apidocs/latest/jersey/index.html) 和 [Viewable](https://jersey.java.net/apidocs/latest/jersey/org/glassfish/jersey/server/mvc/Viewable.html) 可提供的方式来定义一个参考模板。现在我们将讨论如何将这些值解释和如何发现具体的模板。

### 20.3.1\. 相对模块引用

相对引用是指任何路径不以 '/'字符开头 (如 index.foo)。这种类型的引用，将会通过前面加上最后匹配的完全名字的值来转为绝对路径。

考虑[Example 20.3, “Using @Template on a resource class”](https://jersey.java.net/documentation/latest/user-guide.html#mvc.example.implicit.class),模板名称引用 index 是一个相对值，Jersey 将会使用完全匹配的类名称 Foo 来转为绝对模板引用（更多详见 [Viewable](https://jersey.java.net/apidocs/latest/jersey/org/glassfish/jersey/server/mvc/Viewable.html)），我们的例子：

"/com/foo/Foo/index"

Jersey 将会搜索所有的注册的模板处理器（见 20.7\. Writing Custom Templating Engines）来找到可以转为绝对模板引用的模板处理器。如果这样的模板处理器找到了，那么可以被“处理”的模板将会使用提供的数据模型来处理。

*注意：如果不提供或空的模板参考（无论是在 Viewable 或通过 @Template）那么假设为 index 引用，对这个值的所有进一步的处理是这一价值。*

### 20.3.2\. 绝对模板引用

修改我们的 Foo 资源

Example 20.4\. Using absolute path to template in Viewable

```java
@GET
public Viewable get() {
    return new Viewable("/index", "FOO");
} 
```

这个例子，模板引用是以 "/" 开头，所以无需进行绝对化，就已经是绝对引用了。

绝对模板引用以 "/" 字符开头，（如 /com/example/index.foo） ，无需再做转换，就能通过提供的路径找到模板。

注意，然而，对于自定义模板引擎的模板处理器可以修改（和支持）绝对模板引用通过在前面添加‘基模板路径’（如果定义）而后添加模板后缀（如 foo）如果后缀不提供在引用中。

例如，假设我们想用 Mustache 模板给我们的页面，我们定义了‘基模板路径’作为页面。那么绝对模板引用 /com/example/Foo/index 模板处理器将会将引用转换成以下路径：/pages/com/example/Foo/index.mustache.

# 20.4\. MVC 错误处理

# 20.5\. 注册和配置

# 20.6\. 支持的模板引擎

# 20.7\. 写自定义的模板引擎

# 20.8\. 其他例子