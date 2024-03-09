# 第二十三章 使用 Spring 注入

# Chapter 23\. Spring DI 使用 Spring 注入

Jersey 提供对 Spring DI 的扩展。使得 Jersey 在使用 Spring bean 时 就像是 JAX-RS 的组件 (比如 资源和提供者) 并且允许 Spring 注入 Jersey 管理的组件中。

这个 Spring 扩展模块配置是基于注解的。当 Spring bean 注入，就会产生的 受 Spring 管理的 JAX-RS 类。 注入的 Spring bean 不只是 通过 Spring XML 来配置，也支持用 Spring 的 单例和 请求域。

为了 JAX-RS 资源能和 Spring 的功能正常工作还需要代理，比如 Spring 的事务管理（用 @Transactional）， Spring Security 和 面向切面编程(如 @Aspect)，资源必须通过 Spring 的注解 @Component, @Service, @Controller 或 @Repository 来管理：

```java
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import org.springframework.stereotype.Component;

@Component
@Path("/")
public class SomeResource {

    @Transactional
    @GET
    public void updateResource() {
        // ...
    }
} 
```

*局限性： Spring bean 不能通过 Spring XML 配置被直接注射到 JAX-RS 类*