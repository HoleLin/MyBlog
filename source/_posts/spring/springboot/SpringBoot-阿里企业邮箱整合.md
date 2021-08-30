---
title: SpringBoot-阿里企业邮箱整合
date: 2021-08-30 14:03:54
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- 阿里企业邮箱
- Mail
categories:
- SpringBoot
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [企业云邮箱POP\SMTP\IMAP地址和端口信息](https://help.aliyun.com/knowledge_detail/36576.html)

* [springboot 发送邮件 465端口](https://blog.csdn.net/qq_24132367/article/details/84493426)

#### 依赖

```xml
		<!--    <springboot.version>2.2.2.RELEASE</springboot.version>-->
		<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>${springboot.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>${springboot.version}</version>
    </dependency>
		<!-- 模板:用于邮件模板-->	
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-freemarker</artifactId>
      <version>${springboot.version}</version>
    </dependency>
		<!-- 邮件依赖-->	
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-mail</artifactId>
      <version>${springboot.version}</version>
    </dependency>
```

#### 配置

* 阿里云官网关于邮件端口说明

  > 企业云邮箱各个服务器地址及端口信息如下：
  >
  > 收件服务器地址：
  >
  > POP 服务器地址：pop.qiye.aliyun.com 端口110，SSL 加密端口995
  >
  > 或
  >
  > IMAP 服务器地址：imap.qiye.aliyun.com 端口143，SSL 加密端口993
  >
  > 发件服务器地址：
  >
  > SMTP 服务器地址：smtp.qiye.aliyun.com 端口25， SSL 加密端口465

```yaml
spring:
  mail:
    host: smtp.qiye.aliyun.com
    # 端口可以使用25/465
    # port: 25
    username: xxxx
    password: xxxxx
    test-connection: true
    protocol: smtp
    # 使用SSL加密端口(465)要补充一下配置
    properties:
      mail:
        smtp:
          socketFactory:
            class: javax.net.ssl.SSLSocketFactory
            port: 465
            auth: true
            starttls:
              enable: true
              required: true
```

#### 代码

```java
@Component
public class MailUtils {

    @Autowired
    private JavaMailSenderImpl sender;

    @Value("${spring.mail.username}")
    private String from;

    @Autowired
    private Configuration configuration;

    public void sendMessage() throws MessagingException {
        OrderMailInfo orderMailInfo = new OrderMailInfo();
        Faker faker = new Faker();
        orderMailInfo.setPatientName(faker.name().fullName());
        orderMailInfo.setCreateTime(faker.date().birthday());
        orderMailInfo.setOrderId(UUID.randomUUID().toString());
        orderMailInfo.setTotalFee(new BigDecimal(faker.number().randomNumber()));
        orderMailInfo.setSupplierCode(faker.name().fullName());
        orderMailInfo.setHospital(faker.company().name());
        orderMailInfo.setTelephoneNumber(faker.phoneNumber().subscriberNumber());
        MimeMessage mimeMessage = sender.createMimeMessage();
        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, false);
        mimeMessageHelper.setFrom(from);
        mimeMessageHelper.setSubject("测试");
        mimeMessageHelper.addTo("nolelin@163.com");
        try {
            // 指定模板文件路径
            ClassPathResource classPathResource = new ClassPathResource("/templates");
            configuration.setDirectoryForTemplateLoading(classPathResource.getFile());
            // 获取具体模板文件
            Template template = configuration.getTemplate("order.ftl");
            String content = FreeMarkerTemplateUtils.processTemplateIntoString(template, orderMailInfo);
            mimeMessageHelper.setText(content, true);
            sender.send(mimeMessageHelper.getMimeMessage());
        } catch (IOException | TemplateException e) {
            e.printStackTrace();
        }
    }
}
```

##### 从数据库中读取模板信息

* 配置`TemplateLoader`

```kotlin
/**
 * 数据库模板加载器
 * 目的: 使得FreeMaker模板从数据库加载
 */
@Component
class DatabaseTemplateLoader : TemplateLoader {
    @Autowired
    private lateinit var notificationDao: NotificationDao
		/**
		 * 此方法作用为在使用getTemplate()方法获取模板信息时,根据传入的参数到数据库中查询对应的数据模版信息
		 */
    override fun findTemplateSource(name: String): Any? {
        // 此处要记性字符串截取,因为传入的name格式为"2_zh_CN_#Han" 此处的2来源于最终getTemplate方法的入参
        val indexOf = name.indexOf('$')
        val param = name.substring(0, indexOf)
        // 此处要求传入的参数格式为"purpose;type"用来确定一条通知的记录
        val params = param.split(";")
        val purpose = params[0]
        val type = params[1]
        return notificationDao.queryByPurposeAndType(purpose, type)
    }

    override fun getLastModified(templateSource: Any?): Long {
        val notification = templateSource as Notification
        return notification.updatedTime.time
    }
		// 真正获取具体模板的方法
    override fun getReader(templateSource: Any?, encoding: String?): Reader {
        return StringReader(Base64.decodeStr((templateSource as Notification).templateContent))
    }

    override fun closeTemplateSource(templateSource: Any?) {
        // Do nothing.
    }
}
```

* 获取模板

```kotlin
    @Autowired
    private lateinit var freeMarkerConfigurer: FreeMarkerConfigurer
    
    /**
     * 采用来自数据库的模板发送邮件 默认以HTML格式发送
     * @param purpose 发送目的 用于检查模板是否存在和查询模板
     */
    @Throws(MailException::class)
    fun sendMessageWithDbTemplate(
        subject: String,
        recipients: Set<String>,
        dataModel: Any,
        purpose: String,
        type: String
    ) {
        // 检查当前传入的通知目的是否存在模板
        val countByPurpose = notificationDao.countByPurposeAndType(purpose, type)
        // 只能一条
        if (countByPurpose == 1L) {
            val template = freeMarkerConfigurer.configuration.getTemplate("$purpose;$type$")
            val content = FreeMarkerTemplateUtils.processTemplateIntoString(template, dataModel)
            this.sendMessage(subject, content, true, recipients, null, null, null)
        } else {
            // 发送邮件失败 模板信息不存在
            throw MailPreparationException("Template information does not exist in the database")
        }
    }
```







