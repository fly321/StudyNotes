# Spring Cloud Alibaba：Nacos 作为注册中心和配置中心使用

> Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案，Nacos 作为其核心组件之一，可以作为注册中心和配置中心使用，本文将对其用法进行详细介绍。

## Nacos简介

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

nacos快速开始：https://nacos.io/zh-cn/docs/quick-start.html

Nacos 具有如下特性:

- 服务发现和服务健康监测：支持基于DNS和基于RPC的服务发现，支持对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求；
- 动态配置服务：动态配置服务可以让您以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置；
- 动态 DNS 服务：动态 DNS 服务支持权重路由，让您更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单DNS解析服务；
- 服务及其元数据管理：支持从微服务平台建设的视角管理数据中心的所有服务及元数据。

## 使用Nacos作为注册中心

- 我们先从官网下载Nacos，这里下载的是`nacos-server-1.1.4.zip`文件，下载地址：https://github.com/alibaba/nacos/releases
- 配置`JAVA_HOME`环境变量，不配置会导致无法运行Nacos；

```bash
JAVA_HOME=D:\developer\env\Java\jdk1.8.0_91Copy to clipboardErrorCopied
```

- 解压安装包，直接运行`bin`目录下的`startup.cmd`；
- 运行成功后，访问`http://localhost:8848/nacos`可以查看Nacos的主页，默认账号密码都是nacos。

![img](http://www.macrozheng.com/images/spingcloud_nacos_01.png)

## 创建应用注册到Nacos

- 创建nacos-user-service模块和nacos-ribbon-service模块；
- 如果要使用Spring Cloud Alibaba 的组件都需要在pom.xml中添加如下的配置；

```xml
	<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

- 添加nacos依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>${latest.version}</version>
</dependency>
```

- 修改配置文件application.yml，将Consul的注册发现配置改为Nacos的：

```yaml
server:
  port: 8206
spring:
  application:
    name: nacos-user-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址
management:
  endpoints:
    web:
      exposure:
        include: '*'Copy to clipboardErrorCopied
```

> 说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

**运行两个nacos-user-service和一个nacos-ribbon-service，在Nacos页面上可以看到如下信息：**

![img](http://www.macrozheng.com/images/spingcloud_nacos_02.png)

## 负载均衡功能

> 由于我们运行了两个nacos-user-service，而nacos-ribbon-service默认会去调用它的接口，我们调用nacos-ribbon-service的接口来演示下负载均衡功能。

导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

使用ribbon进行负载均衡，**使用@LoadBalanced注解赋予RestTemplate负载均衡的能力**

```java
@Configuration
public class RibbonConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

多次调用接口：http://localhost:8308/user/1 ，可以发现两个nacos-user-service的控制台交替打印如下信息。

```bash
2019-11-06 14:28:06.458  INFO 12092 --- [nio-8207-exec-2] c.macro.cloud.controller.UserController  : 根据id获取用户信息，用户名称为：macro
```

## 使用Nacos作为配置中心

> 我们通过创建nacos-config-client模块，并在Nacos页面中添加配置信息来演示下配置管理的功能。

### 创建nacos-config-client模块

- 在pom.xml中添加相关依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

- 添加配置文件application.yml，启用的是dev环境的配置：

```yaml
spring:
  profiles:
    active: dev
```

- 添加配置文件bootstrap.yml，主要是对Nacos的作为配置中心的功能进行配置：

```yaml
server:
  port: 9101
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos地址
      config:
        server-addr: localhost:8848 #Nacos地址
        file-extension: yaml #这里我们获取的yaml格式的配置
```

- 创建ConfigClientController，从Nacos配置中心中获取配置信息：

```java
/**
 * Created by macro on 2019/9/11.
 */
@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

### 在Nacos中添加配置

- 在 `bootstrap.properties` 中配置 Nacos server 的地址和应用名

```
spring.cloud.nacos.config.server-addr=127.0.0.1:8848

spring.application.name=example
```

说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```plain
${prefix}-${spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，就算不为空也可以dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新：

```
@RestController
@RequestMapping("/config")
@RefreshScope
public class ConfigController {

    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    @RequestMapping("/get")
    public boolean get() {
        return useLocalCache;
    }
}
```

- 按照以上dataid添加如下配置：

```yaml
config:
  info: "config info for dev"
```

- 填写配置示意图：

![img](http://www.macrozheng.com/images/spingcloud_nacos_03.png)

- 启动nacos-config-client，调用接口查看配置信息：http://localhost:9101/configInfo

```bash
config info for dev
```