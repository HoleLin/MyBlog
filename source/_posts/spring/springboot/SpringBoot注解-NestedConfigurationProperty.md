---
title: SpringBoot注解-@NestedConfigurationProperty
date: 2021-08-10 09:46:47
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- NestedConfigurationProperty
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

#### 示例

* 依赖

  ```
      implementation 'org.springframework.boot:spring-boot-configuration-processor:2.2.2.RELEASE'
  ```

* `JavaConfig`:

  ```kotlin
  import com.holelin.config.oss.AliYunConfig
  import com.holelin.config.oss.MinioConfig
  import org.springframework.boot.context.properties.ConfigurationProperties
  import org.springframework.boot.context.properties.NestedConfigurationProperty
  import org.springframework.stereotype.Component
  
  @Component
  @ConfigurationProperties(prefix = "oss")
  open class OSSConfig {
  
      @NestedConfigurationProperty
      var aliYunConfig: AliYunConfig = AliYunConfig()
  
      @NestedConfigurationProperty
      var minioConfig: MinioConfig = MinioConfig()
  }
  ```

  ```kotlin
  import java.io.Serializable
  
  class AliYunConfig :Serializable{
      /**
       * Region表示OSS的数据中心所在物理位置
       */
      lateinit var region: String
  
      /**
       * Endpoint表示OSS对外服务的访问域名。
       */
      lateinit var endpoint: String
  
      /**
       * AccessKey简称AK，指的是访问身份验证中用到的AccessKeyId和AccessKeySecret。
       */
      lateinit var accessKey: String
  
      lateinit var accessSecret: String
  
  }
  ```

  ```kotlin
  import java.io.Serializable
  
  class MinioConfig :Serializable {
      /**
       * Endpoint表示OSS对外服务的访问域名。
       */
       lateinit var endpoint: String
  
      /**
       * 密钥
       */
       lateinit var accessKey: String
  
      /*
       * 密钥密码
       */
       lateinit var accessSecret: String
  
  }
  ```

* `application.yml`:

  ```yaml
  oss:
    minio-config:
      endpoint: http://127.0.0.1:9000
      access-key: holelin
      access-secret: 12345678
    ali-yun-config:
      region: xxx
      endpoint: xxx
      access-key: xxx
      access-secret: xxx
  ```

  
