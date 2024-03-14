> 主要记录一下Java如何获取前端传过来的一些值。

#### 获取URl中的参数（get请求及post请求）

在Java Spring Boot中，可以通过以下方法获取URL中的参数值，一般放在url中的参数，是**不容许复杂参数**（数组等）：

1. 使用`@RequestParam`注解获取URL中的参数值。例如：

   ```java
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class MyController {
   
       @GetMapping("/hello")
       public String hello(@RequestParam("name") String name) {
           return "Hello, " + name;
       }
   }
   
   ```

2. 使用`HttpServletRequest`对象获取URL中的参数值。例如：

   ```java
   import javax.servlet.http.HttpServletRequest;
   
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class MyController {
   
       @GetMapping("/hello")
       public String hello(HttpServletRequest request) {
           String name = request.getParameter("name");
           return "Hello, " + name;
       }
   }
   ```

3. 使用`@PathVariable`注解获取URL中的参数值。 (这种适合于Restful 风格的）例如：

   ```java
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class MyController {
   
       @GetMapping("/hello/{name}")
       public String hello(@PathVariable("name") String name) {
           return "Hello, " + name;
       }
   }
   ```

#### 获取Post请求中

后端可以通过以下几种方式获取前端通过POST请求传过来的值，包括URL中和请求体中都有：

1. 使用注解获取其参数（请求体和url中的参数）

   ```java
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;
   import javax.servlet.http.HttpServletRequest;
   
   @RestController
   public class UserController {
   
       @PostMapping("/submit")
       public String submit(@RequestParam("username") String username, @RequestParam("password") String password, @RequestBody String requestBody) {
           // 处理数据，例如验证、存储等
           System.out.println("URL参数：username=" + username + ", password=" + password);
           System.out.println("请求体：" + requestBody);
           return "提交成功";
       }
   }
   
   ```

2. 使用`HttpServletRequest` 获取其参数

   ```java
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RestController;
   import javax.servlet.http.HttpServletRequest;
   import java.io.BufferedReader;
   import java.io.IOException;
   
   @RestController
   public class UserController {
   
       @PostMapping("/submit")
       public String submit(HttpServletRequest request) throws IOException {
           // 获取URL中的参数
           String username = request.getParameter("username");
           String password = request.getParameter("password");
   
           // 获取请求体中的参数
           BufferedReader reader = request.getReader();
           StringBuilder sb = new StringBuilder();
           String line;
           while ((line = reader.readLine()) != null) {
               sb.append(line);
           }
           String requestBody = sb.toString();
   
           // 处理数据，例如验证、存储等
           System.out.println("URL参数：username=" + username + ", password=" + password);
           System.out.println("请求体：" + requestBody);
           return "提交成功";
       }
   }
   ```

#### 获取一些请求头，cookie之类的参数

可以通过`HttpServletRequest`对象获取请求头、session和cookie等数据

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

@RestController
public class UserController {

    @PostMapping("/submit")
    public String submit(HttpServletRequest request) {
        // 获取请求头
        String userAgent = request.getHeader("User-Agent");

        // 获取session
        HttpSession session = request.getSession();

        // 获取cookie
        Cookie[] cookies = request.getCookies();

        // 处理数据，例如验证、存储等
        System.out.println("请求头：" + userAgent);
        System.out.println("session：" + session);
        System.out.println("cookie：" + Arrays.toString(cookies));
        return "提交成功";
    }
}
```

