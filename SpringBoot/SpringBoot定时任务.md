> springboot整合task实现定时任务，定时任务用于秒杀等定时操作。

#### 开启定时任务功能

使用注解`EnableScheduling`在启动类开启定时任务功能

```Java
@SpringBootApplication
//开启定时任务功能
@EnableScheduling
public class Springboot22TaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot22TaskApplication.class, args);
    }

}
```

#### 设置定时执行的任务，并设定执行周期

使用注解Scheduled 在要定时执行的方法上。并设置执行周期

```Java
 @Scheduled(cron = "0/1 * * * * ?")
    @GetMapping("hello")
    public String getHello()
    {
        System.out.println("我执行了");
        return "hello";
    }
```

#### 定时任务相关配置

```YAML
spring:
  task:
    scheduling:
      # 任务调度线程池大小 默认1
     pool:
       size: 1
     # 调度线程名称前缀 默认scheduling-
     thread-name-prefix: ssm_
     shutdown:
       # 线程池关闭时等待所有任务完成
       await-termination: false
       # 调度线程关闭前最大等待时间，确保最后一定关闭
       await-termination-period: 10s
```