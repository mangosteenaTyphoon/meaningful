## Spring Security简介

Spring Security是一个框架，提供了各种安全功能，例如: 身份验证，创建安全Java Enterprise Applications的授权。此框架的优点在于其灵活的身份验证特性，可以与任何软件解决方案集成。有时，开发人员希望将其与不遵循任何安全性标准的遗留系统集成，在那里Spring Security可以很好地工作。

## 认证 

用户认证就是判断一个用户的身份是否合法的过程，用户去访问系统资源时系统要求验证用户的身份信息，身份合法方可继续访问，不合法则拒绝访问。常见的用户身份认证方式有：用户名密码登录，二维码登录，手机短信登录，指纹认证等方式。**简而言之，认证就是为了保护系统的隐私数据和资源，用户的身份合法可访问该系统的资源。**

## 授权

授权是用户认证通过后，根据用户的权限来控制用户访问资源的过程，拥有资源的访问权限则正常访问，没有权限则拒绝访问。**认证是为了保证用户身份的合法性，授权则是为了更细粒度的对隐私数据进行划分**，授权是在认证通过后发生的，控制不同的用户能够访问不同的资源。

## 会话

用户认证通过后，为了避免用户的每次操作都进行认证可将用户的信息保证在会话中。会话就是系统为了保持当前用户的登录状态所提供的机制，常见的有基于session方式、基于token方式等。

## 快速入门

我们搭建一个简单的springBoot工程并集成SpringSecurity。

### 准备工作

首先，创建一个maven项目，并添加依赖文件。

~~~ xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
            <version>2.6.3</version>
        </dependency>
    </dependencies>

~~~

创建启动类

```java
@SpringBootApplication
public class SecurityApplication {
    public static void main(String[] args) {
        SpringApplication.run(SecurityApplication.class,args);
    }
}
```

创建HelloWord控制类

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

启动程序，我们尝试访问hello接口，会被强制跳转到一个登录页面，登录账号为user，密码在控制台有输出，这就是最原始的Spring Security的使用了。接下来我们细细抛出所有可以修改的内容。

## 登录校验流程

在我们实际项目中，前后端分离进行账号登录。前端先将携带着账号和密码的请求发送给服务端进行登录，服务端将前端发送过来的账号和密码与数据库中的账号和密码进行对比。如果正确，将会生成一个jwt，并且将此jwt（也就是token）返回给前端。前端因此登录成功，并且获取到用户信息。

登录后访问其他接口时，在每次发送的请求中需要在请求头中携带token，服务端接收到请求后将token进行解析出userID。根据用户Id获取用户相关的信息，如果有权则容许访问相关资源。将可以访问的资源返回给前端。

![image-20230206161030037](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202302061610291.png)

### 原理初探

 我们如何使用Spring Security来实现自己的登录流程。我们必须要清楚Spring Security如何认证和授权。

### SpringSecurity 完整流程

SpringSecurity的原理其实就是一个过滤器链，内部包含了提供各种功能的过滤器。这里我们可以看看入门案例中的过滤器。

![image-20230206164702600](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202302061647728.png)

- **UsernamePasswordAuthenticationFilter**:负责处理登陆页面填写了用户名密码后的登陆请求。入门案例的认证工作主要有它负责。
- **ExceptionTranslationFilter：** 处理过滤器链中抛出的任何AccessDeniedException和AuthenticationException。
- **FilterSecurityInterceptor：** 负责权限校验的过滤器。

这里只展示了核心过滤器，其他非核心过滤器可以可以通过Debug查看当前系统中SpringSecurity过滤器链中有哪些过滤器及它们的顺序。

### 认证流程详解

![image-20230206165814457](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202302061658616.png)

- Authentication接口: 它的实现类，表示当前访问系统的用户，封装了用户相关信息。
- AuthenticationManager接口：定义了认证Authentication的方法
- UserDetailsService接口：加载用户特定数据的核心接口。里面定义了一个根据用户名查询用户信息的方法。
- UserDetails接口：提供核心用户信息。通过UserDetailsService根据用户名获取处理的用户信息要封装成UserDetails对象返回。然后将这些信息封装到Authentication对象中。

### 实现自定义认证

登录思路分析

- 自定义登录接口调用ProviderManager的方法进行认证 如果认证通过生成jwt把用户信息存入redis中。
- 自定义UserDetailsService在这个实现类中去查询数据库。

校验思路分析

- 定义Jwt认证过滤器获取token解析token获取其中的userid从redis中获取用户信息存入SecurityContextHolder。



















