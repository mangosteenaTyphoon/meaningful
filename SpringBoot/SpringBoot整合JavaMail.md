> 使用java来编写发送邮件的功能。

### 简单邮箱代码编写

#### 导入依赖

```XML
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
  </dependencies>
```

#### 配置相关文件

打开qq邮箱点击设置——>账户 然后拿取密码

```YAML
spring:
  mail:
    host: smtp.qq.com
    username: 发送邮件的邮箱账号
    password: 去smtp拿密码
```

#### 编写纯文本邮件

简单邮件使用的是SimpleMailMessage 来发送邮件

```Java
@Service
public class SendMailServiceImpl implements SendMailService {

    @Autowired
    private JavaMailSender javaMailSender;

    //发送人
    private String from = "test@qq.com";
    //接收人
    private String to = "test@126.com";
    //标题
    private String subject = "测试邮件";
    //正文
    private String context = "测试邮件正文内容";

    @Override
    public void sendMail() {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from + "(小甜甜)");
        message.setTo(to);
        message.setSubject(subject);
        message.setText(context);
        javaMailSender.send(message);
    }
}
```

#### 发送多部件邮件

和简单邮件不一样的时这里的SimpleMailMessage 我们要用 MimeMessage ，代表你要发送的邮件是多部件邮件

```Java
@Service
public class SendMailServiceImpl implements SendMailService {

    @Autowired
    private JavaMailSender javaMailSender;

    //发送人
    private String from = "test@qq.com";
    //接收人
    private String to = "test@126.com";
    //标题
    private String subject = "测试邮件";
    //正文
    private String context = hmtl标签包裹的语言;

    @Override
    public void sendMail() {
       try{
        MimeMessage message = javaMailSendar.CreateMineMessage();
        MineMessageHelper helper=new MimeMessageHelper(message);
        helper.serFrom(from+"(小王)");
        helper.setSubject(subject);
        helper.setText(context,true);
        }catch(Exception e){
        e.prinstackTrace();
    }
}
```

#### 发送带有附件的邮箱

```Java
 //实现发送多部件邮件
    @Override
    public void sendMimeMail() {
 
        try {
            //再利用javaMailsender创建一个多部件的邮件
            MimeMessage message = javaMailSender.createMimeMessage();
            //再使用这个api设置邮件
            //这里true代表你要发附件邮件，只要这里为true才可以发文件
            MimeMessageHelper helper = new MimeMessageHelper(message,true);
            helper.setFrom(from+"(鹿晗)");
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(context,true);//这个含义是识别html的标签
            //发送本地文件也是可以
            File file=new File("D:\\idea_place\\work\\springboot2-mail\\target\\springboot2-mail-0.0.1-SNAPSHOT.jar");
            File file1=new File("D:\\idea_place\\work\\springboot2-mail\\src\\main\\java\\com\\hb\\img\\s2.jpg");
            //利用这个方法将文件作为附件发过去
            helper.addAttachment(file.getName(),file);
            helper.addAttachment("海景图.png",file1);
 
 
            javaMailSender.send(message);
        } catch (Exception e) {
            e.printStackTrace();
        }
 
    }
```