## 1.背景

License，即版权许可证，一般用于收费软件给付费用户提供的访问许可证明。根据应用部署位置的不同，一般可以分为以下两种情况讨论：

- 应用部署在开发者自己的云服务器上，如现在的saas模式的软件供应商就是这样部署的。这种情况下用户通过账号登录的形式远程访问，因此只需要在账号登录的时候校验目标账号的有效期、访问权限等信息即可。
- 应用部署在客户的内网环境，即本地化部署。因为这种情况开发者无法控制客户的网络环境，也不能保证应用所在服务器可以访问外网，因此通常的做法是使用服务器许可文件，在应用启动的时候加载证书，然后在登录或者其他关键操作的地方校验证书的有效性。

**TrueLicense是一个开源的主流证书管理引擎，可以用于license的生成和有效性的验证**。

## 2.TrueLicense

使用TrueLicense相对简单，首先我们需要keytool生产[密钥](https://so.csdn.net/so/search?q=密钥&spm=1001.2101.3001.7020)对以供后续生成、验证证书使用。

keytool是jdk里面自带的命令。我们直接用keytool命令来生成密钥对。需要执行的命令如下：

```
## 1. 生成私匙库
# validity：私钥的有效期多少天
# alias：私钥别称
# keystore: 指定私钥库文件的名称(生成在当前目录)
# storepass：指定私钥库的密码(获取keystore信息所需的密码) 
# keypass：指定别名条目的密码(私钥的密码) 
keytool -genkeypair -keysize 1024 -validity 3650 -alias "privateKey" -keystore "privateKeys.keystore" -storepass "123456" -keypass "123456" -dname "CN=localhost, OU=localhost, O=localhost, L=SH, ST=SH, C=CN"

## 2. 把私匙库内的公匙导出到一个文件当中
# alias：私钥别称
# keystore：指定私钥库的名称(在当前目录查找)
# storepass: 指定私钥库的密码
# file：证书名称
keytool -exportcert -alias "privateKey" -keystore "privateKeys.keystore" -storepass "123456" -file "certfile.cer"

## 3. 再把这个证书文件导入到公匙库
# alias：公钥别称
# file：证书名称
# keystore：公钥文件名称
# storepass：指定私钥库的密码
keytool -import -alias "publicCert" -file "certfile.cer" -keystore "publicCerts.keystore" -storepass "123456"
```

在任意目录下执行完上述三个命令之后。我们会在当前目录下面得到三个文件：privateKeys.keystore、publicCerts.keystore、certfile.cer。

- privateKeys.keystore：私钥，这个我们自己留着，不能泄露给别人。
- publicCerts.keystore：公钥，这个给客人用的。在我们程序里面就是用他来解析license文件里面的信息的。
- certfile.cer：这个文件没啥用，可以删掉。

## 3.springboot整合TrueLicense

通过上面已经生成好了密钥对(私钥、公钥)。接下来就是生成license证书，以及验证证书的合法性了

#### 3.1 引入TrueLicense依赖

```
  <!-- License -->
  <dependency>
    <groupId>de.schlichtherle.truelicense</groupId>
    <artifactId>truelicense-core</artifactId>
    <version>1.33</version>
  </dependency>
```

#### 3.2 生成license证书

这里我们提供了一个restful接口[LicenseController](https://github.com/plasticene/plasticene-boot-starter-parent/blob/main/plasticene-boot-starter-license/src/main/java/com/plasticene/boot/license/web/LicenseController.java)直接返回流生成并下载license证书文件。

```
@RestController
@Api(tags = "license管理")
@RequestMapping("/license")
public class LicenseController {

    @Resource
    private LicenseCreator licenseCreator;
    @Resource
    private LicenseProperties licenseProperties;

    @PostMapping
    @ApiOperation("生成license")
    public void create(LicenseCreatorParam creatorParam, HttpServletResponse response) throws IOException {
        boolean flag = licenseCreator.generateLicense(creatorParam);
        List<String> list = StrUtil.split(licenseProperties.getLicensePath(), "/");
        String fileName = list.get(list.size()-1);
        response.setContentType("application/octet-stream");
        response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
        BufferedInputStream inputStream = FileUtil.getInputStream(licenseProperties.getLicensePath());
        IoUtil.copy(inputStream, response.getOutputStream());
    }
}
```

这里的生成证书入参[LicenseCreatorParam](https://github.com/plasticene/plasticene-boot-starter-parent/blob/main/plasticene-boot-starter-license/src/main/java/com/plasticene/boot/license/core/param/LicenseCreatorParam.java)核心参数：设置证书有效日期和服务器系统信息

```
  /**
     * 证书失效时间
     */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date expiryTime;

    /**
     * 服务器系统信息
     */
    private SystemInfo systemInfo;
```

我们知道**只验证license的证书的合法性、有效期，还是有被破解的风险，因为别人可以找到该证书文件copy到其他服务器相同路径下就可以了。所以为了加强证书验证的严谨性，我们添加对服务器系统信息验证，针对系统唯一标识、cpu序列号等进行验证，从而达到同一证书在不同服务器之间是不能共用的**。

这里需要提一下：我们并没有根据系统服务器的mac地址进行验证，因为目前一般使用主流的云原生容器化技术部署java服务，java服务对应的容器的mac地址是不固定的，所以这里就不使用macId了，当然如果一定要校验服务器宿主机的mac地址，可以将mac地址配置到容器的环境变量中，然后java代码从环境变量中获取mac地址，也能实现基于mac地址验证

**生成license证书核心逻辑**：[LicenseCreator](https://github.com/plasticene/plasticene-boot-starter-parent/blob/main/plasticene-boot-starter-license/src/main/java/com/plasticene/boot/license/core/LicenseCreator.java)

```
@Component
public class LicenseCreator {
    @Resource
    private LicenseProperties licenseProperties;

    private static Logger logger = LogManager.getLogger(LicenseCreator.class);
    private final static X500Principal DEFAULT_HOLDER_AND_ISSUER = new X500Principal("CN=localhost, OU=localhost, O=localhost, L=SH, ST=SH, C=CN");


    /**
     * 生成License证书
     */
    public boolean generateLicense(LicenseCreatorParam param){
        try {
            LicenseManager licenseManager = new LicenseManager(initLicenseParam());
            LicenseContent licenseContent = initLicenseContent(param);
            licenseManager.store(licenseContent, new File(licenseProperties.getLicensePath()));
            return true;
        }catch (Exception e){
            logger.error("证书生成失败：", e);
            throw new BizException("生成license证书失败");
        }
    }

    /**
     * 初始化证书生成参数
     */
    private LicenseParam initLicenseParam(){
        Preferences preferences = Preferences.userNodeForPackage(LicenseCreator.class);

        //设置对证书内容加密的秘钥
        CipherParam cipherParam = new DefaultCipherParam(licenseProperties.getStorePass());

        KeyStoreParam privateStoreParam = new CustomKeyStoreParam(LicenseCreator.class
                ,licenseProperties.getPrivateKeysStorePath()
                ,licenseProperties.getPrivateAlias()
                ,licenseProperties.getStorePass()
                ,licenseProperties.getKeyPass());

        LicenseParam licenseParam = new DefaultLicenseParam(licenseProperties.getSubject()
                ,preferences
                ,privateStoreParam
                ,cipherParam);

        return licenseParam;
    }

    /**
     * 设置证书生成正文信息
     */
    private LicenseContent initLicenseContent(LicenseCreatorParam param){
        LicenseContent licenseContent = new LicenseContent();
        licenseContent.setHolder(DEFAULT_HOLDER_AND_ISSUER);
        licenseContent.setIssuer(DEFAULT_HOLDER_AND_ISSUER);

        licenseContent.setSubject(licenseContent.getSubject());
        licenseContent.setIssued(param.getIssuedTime());
        licenseContent.setNotBefore(param.getIssuedTime());
        licenseContent.setNotAfter(param.getExpiryTime() == null ? addYears(new Date(), 10) : param.getExpiryTime());
        licenseContent.setConsumerType(param.getConsumerType());
        licenseContent.setConsumerAmount(param.getConsumerAmount());
        licenseContent.setInfo(param.getDescription());
        licenseContent.setExtra(param.getSystemInfo());
        return licenseContent;
    }

    public  Date addYears(Date date, int n) {
        Calendar cal = Calendar.getInstance();
        cal.setTime(date);
        cal.add(Calendar.YEAR, n);
        return cal.getTime();
    }
}
```

通过以上操作流程我们就可以生产license证书文件了，接下来就是对证书的检验。

#### 3.3 license证书验证

这里我们支持两种情况下的证书验证：

- **服务启动时对证书校验**：这时候会根据license配置进行证书的合法性、有效性、服务器系统信息验证。
- **接口层面使用@License注解进行证书检验**：如登录接口验证license的有效性，如果已过期，及时告知客户。

验证流程图如下所示：

![123](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/blog/202301311126434.png)

通过流程图可知，相关检验都可以通过license配置属性开关控制，做到高度可插拔。

通过实现`ApplicationRunner`在服务启动时进行license验证

```
@Order(OrderConstant.RUNNER_LICENSE)
public class LicenseCheckApplicationRunner implements ApplicationRunner {
    @Resource
    private LicenseVerify licenseVerify;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        LicenseContent content = licenseVerify.install();
    }
}
```

通过license配置属性，条件装配注入组件：

```
    @Bean
    @ConditionalOnProperty(name = "ptc.license.start-check", havingValue = "true", matchIfMissing = true)
    public LicenseCheckApplicationRunner licenseCheckApplicationRunner() {
        return new LicenseCheckApplicationRunner();
    }
```

通过注解`@License`实现接口层面的license检验切面

```
@Aspect
@Order(OrderConstant.AOP_LICENSE)
public class LicenseAspect extends AbstractAspectSupport {

    @Resource
    private LicenseVerify licenseVerify;

    // 指定切入点为License注解
    @Pointcut("@annotation(com.plasticene.boot.license.core.anno.License)")
    public void licenseAnnotationPointcut() {
    }

    // 环绕通知
    @Around("licenseAnnotationPointcut()")
    public Object aroundLicense(ProceedingJoinPoint pjp) throws Throwable {
        boolean b = licenseVerify.verify();
        if (b) {
            return pjp.proceed();
        }
        return null;
    }

}
```

license证书验证核心逻辑：[LicenseVerify](https://github.com/plasticene/plasticene-boot-starter-parent/blob/main/plasticene-boot-starter-license/src/main/java/com/plasticene/boot/license/core/LicenseVerify.java)

```
@Component
public class LicenseVerify {
    @Resource
    private LicenseProperties licenseProperties;

    private static Logger logger = LogManager.getLogger(LicenseVerify.class);
    private static final  DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");


    /**
     * 安装License证书
     * 项目服务启动时候安装证书，检验合法性
     * 此时根据开关验证服务器系统信息
     */
    public synchronized LicenseContent install() {
        LicenseContent result = null;
        try{
            LicenseManager licenseManager = new LicenseManager(initLicenseParam());
            licenseManager.uninstall();
            result = licenseManager.install(new File(licenseProperties.getLicensePath()));
            verifySystemInfo(result);
            logger.info("证书安装成功，证书有效期：{} - {}", df.format(result.getNotBefore()),
                    df.format(result.getNotAfter()));
        }catch (Exception e){
            logger.error("证书安装失败:", e);
            throw new BizException("证书安装失败");
        }
        return result;
    }

    /**
     * 校验License证书, 在接口使用{@link com.plasticene.boot.license.core.anno.License}
     * 时候进入license切面时候调用，此时无需再验证服务器系统信息，验证证书和有效期即可
     */
    public boolean verify() {
        try {
            LicenseManager licenseManager = new LicenseManager(initLicenseParam());
            LicenseContent licenseContent = licenseManager.verify();
            verifyExpiry(licenseContent);
            return true;
        }catch (Exception e){
            logger.error("证书校验失败:", e);
            throw new BizException("证书检验失败");
        }
    }

    /**
     * 初始化证书生成参数
     */
    private LicenseParam initLicenseParam(){
        Preferences preferences = Preferences.userNodeForPackage(LicenseVerify.class);

        CipherParam cipherParam = new DefaultCipherParam(licenseProperties.getStorePass());

        KeyStoreParam publicStoreParam = new CustomKeyStoreParam(LicenseVerify.class
                ,licenseProperties.getPublicKeysStorePath()
                ,licenseProperties.getPublicAlias()
                ,licenseProperties.getStorePass()
                ,null);

        return new DefaultLicenseParam(licenseProperties.getSubject()
                ,preferences
                ,publicStoreParam
                ,cipherParam);
    }

    // 验证证书有效期
    private void verifyExpiry(LicenseContent licenseContent) {
        Date expiry = licenseContent.getNotAfter();
        Date current = new Date();
        if (current.after(expiry)) {
            throw new BizException("证书已过期");
        }
    }

    private void verifySystemInfo(LicenseContent licenseContent) {
        if (licenseProperties.getVerifySystemSwitch()) {
            SystemInfo systemInfo = (SystemInfo) licenseContent.getExtra();
            VerifySystemType verifySystemType = licenseProperties.getVerifySystemType();
            switch (verifySystemType) {
                case CPU_ID:
                    checkCpuId(systemInfo.getCpuId());
                    break;
                case SYSTEM_UUID:
                    checkSystemUuid(systemInfo.getUuid());
                    break;
                default:
            }
        }
    }


    private void checkCpuId(String cpuId) {
        cpuId = cpuId.trim().toUpperCase();
        String systemCpuId = DmcUtils.getCpuId().trim().toUpperCase();
        logger.info("配置cpuId = {},  系统cpuId = {}", cpuId, systemCpuId);
        if (!Objects.equals(cpuId, systemCpuId)) {
            throw new BizException("license检验cpuId不一致");
        }
    }

    private void checkSystemUuid(String uuid) {
        uuid = uuid.trim().toUpperCase();
        String systemUuid = DmcUtils.getSystemUuid().trim().toUpperCase();
        logger.info("配置uuid = {},  系统uuid= {}", uuid, systemUuid);
        if (!Objects.equals(uuid, systemUuid)) {
            throw new BizException("license检验uuid不一致");
        }
    }

}
```

至此，关于使用 TrueLicense 生成和验证License就结束了，完整项目代码请看：https://github.com/plasticene/plasticene-boot-starter-parent/tree/main/plasticene-boot-starter-license