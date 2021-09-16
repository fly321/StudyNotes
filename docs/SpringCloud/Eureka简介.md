# Eureka简介

在微服务架构中往往会有一个注册中心，每个微服务都会向注册中心去注册自己的地址及端口信息，注册中心维护着服务名称与服务实例的对应关系。每个微服务都会定时从注册中心获取服务列表，同时汇报自己的运行情况，这样当有的服务需要调用其他服务时，就可以从自己获取到的服务列表中获取实例地址进行调用，Eureka实现了这套服务注册与发现机制。

> ### 搭建Eureka注册中心

- 创建一个eureka-server模块，并使用Spring Initializer初始化一个SpringBoot项目

![img](http://www.macrozheng.com/images/springcloud_eureka_01.png)

- 填写应用信息

![img](http://www.macrozheng.com/images/springcloud_eureka_02.png)

- 选择你需要的SpringCloud组件进行创建

![img](http://www.macrozheng.com/images/springcloud_eureka_03.png)

- 创建完成后会发现pom.xml文件中已经有了eureka-server的依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

- 在启动类上添加**@EnableEurekaServer**注解来启用Euerka注册中心功能

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

- 在配置文件application.yml中添加Eureka注册中心的配置

```yaml
server:
  port: 8001 #指定运行端口
spring:
  application:
    name: eureka-server #指定服务名称
eureka:
  instance:
    hostname: localhost #指定主机地址
  client:
    fetch-registry: false #指定是否要从注册中心获取服务（注册中心不需要开启）
    register-with-eureka: false #指定是否要注册到注册中心（注册中心不需要开启）
#    service-url: #设置与eureka server交互的地址，服务查询和注册都依赖该地址
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
#this.serviceUrl.put("defaultZone", "http://localhost:8761/eureka/");这个是eureka的默认值，
  server:
    enable-self-preservation: false #关闭保护模式

```

> ### 搭建Eureka客户端

- 新建一个eureka-client模块，并在pom.xml中添加如下依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- 在启动类上添加**@EnableDiscoveryClient**注解表明是一个Eureka客户端

```java
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }

}
```

- 在配置文件application.yml中添加Eureka客户端的配置

```yaml
server:
  port: 8101 #运行端口号
spring:
  application:
    name: eureka-client #服务名称
eureka:
  client:
    register-with-eureka: true #注册到Eureka的注册中心
    fetch-registry: true #获取注册实例列表
    service-url:
      defaultZone: http://localhost:8001/eureka/ #配置注册中心地址
```

- 运行eureka-client

![img](http://www.macrozheng.com/images/springcloud_eureka_07.png)

- 查看注册中心http://localhost:8001/发现Eureka客户端已经成功注册

![img](http://www.macrozheng.com/images/springcloud_eureka_08.png)

提示：可以通过配置info来展示服务的信息，点击上面url会跳转到info

## 搭建Eureka注册中心集群

### 搭建两个注册中心

> 由于所有服务都会注册到注册中心去，服务之间的调用都是通过从注册中心获取的服务列表来调用，注册中心一旦宕机，所有服务调用都会出现问题。所以我们需要多个注册中心组成集群来提供服务，下面将搭建一个双节点的注册中心集群。

- 给eureka-sever添加配置文件application-replica1.yml配置第一个注册中心

```yaml
server:
  port: 8002
spring:
  application:
    name: eureka-server
eureka:
  instance:
    hostname: replica1
  client:
    serviceUrl:
      defaultZone: http://replica2:8003/eureka/ #注册到另一个Eureka注册中心
    fetch-registry: true
    register-with-eureka: true
```

- 给eureka-sever添加配置文件application-replica2.yml配置第二个注册中心

```yaml
server:
  port: 8003
spring:
  application:
    name: eureka-server
eureka:
  instance:
    hostname: replica2
  client:
    serviceUrl:
      defaultZone: http://replica1:8002/eureka/ #注册到另一个Eureka注册中心
    fetch-registry: true
    register-with-eureka: true
```

**这里我们通过两个注册中心互相注册，搭建了注册中心的双节点集群，由于defaultZone使用了域名，所以还需在本机的host文件中配置一下。**

- 修改本地host文件

```
127.0.0.1 replica1
127.0.0.1 replica2
```

### 运行Eureka注册中心集群

> 在IDEA中我们可以通过使用不同的配置文件来启动同一个SpringBoot应用。

- 添加两个配置，分别以application-replica1.yml和application-replica2.yml来启动eureka-server

> 从原启动配置中复制一个出来

![img](http://www.macrozheng.com/images/springcloud_eureka_09.png)

> 配置启动的配置文件

![img](http://www.macrozheng.com/images/springcloud_eureka_10.png)

- 启动两个eureka-server，访问其中一个注册中心http://replica1:8002/发现另一个已经成为其备份

![img](http://www.macrozheng.com/images/springcloud_eureka_11.png)

- 修改Eureka-client，让其连接到集群

> 添加eureka-client的配置文件application-replica.yml，让其同时注册到两个注册中心。

```yaml
server:
  port: 8102
spring:
  application:
    name: eureka-client
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://replica1:8002/eureka/,http://replica2:8003/eureka/ #同时注册到两个注册中心
```

> 以该配置文件启动后访问任意一个注册中心节点都可以看到eureka-client

![img](http://www.macrozheng.com/images/springcloud_eureka_12.png)

注意：在配置集群的时候，register-with-eureka: true，fetch-registry: true为false不影响，因为本身是server所以为false依然可以互相注册为集群，消费者可以值注册一个，当注册的eureka挂了，会自动使用集群中的另一个

## 给Eureka注册中心添加认证

### 创建一个eureka-security-server模块，在pom.xml中添加以下依赖

> 需要添加SpringSecurity模块。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

添加application.yml配置文件

> 主要是配置了登录注册中心的用户名和密码。

```yaml
server:
  port: 8004
spring:
  application:
    name: eureka-security-server
  security: #配置SpringSecurity登录用户名和密码
    user:
      name: adm
      password: 123456
eureka:
  instance:
    hostname: localhost
  client:
    fetch-registry: false
    register-with-eureka: false
```

### 添加Java配置WebSecurityConfig

> 默认情况下添加SpringSecurity依赖的应用每个请求都需要添加CSRF token才能访问，Eureka客户端注册时并不会添加，所以需要配置/eureka/**路径不需要CSRF token。

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```

### 运行eureka-security-server，访问http://localhost:8004发现需要登录认证

![img](http://www.macrozheng.com/images/springcloud_eureka_13.png)

### eureka-client注册到有登录认证的注册中心

- 配置文件中需要修改注册中心地址格式

```
http://${username}:${password}@${hostname}:${port}/eureka/
```

- 添加application-security.yml配置文件，按格式修改用户名和密码

```yaml
server:
  port: 8103
spring:
  application:
    name: eureka-client
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://macro:123456@localhost:8004/eureka/
```

- 以application-security.yml配置运行eureka-client，可以在注册中心界面看到eureka-client已经成功注册

![img](http://www.macrozheng.com/images/springcloud_eureka_14.png)

## Eureka的常用配置

```yaml
eureka:
  client: #eureka客户端配置
    register-with-eureka: true #是否将自己注册到eureka服务端上去
    fetch-registry: true #是否获取eureka服务端上注册的服务列表
    service-url:
      defaultZone: http://localhost:8001/eureka/ # 指定注册中心地址
    enabled: true # 启用eureka客户端
    registry-fetch-interval-seconds: 30 #定义去eureka服务端获取服务列表的时间间隔
  instance: #eureka客户端实例配置
    lease-renewal-interval-in-seconds: 30 #定义服务多久去注册中心续约
    lease-expiration-duration-in-seconds: 90 #定义服务多久不去续约认为服务失效
    metadata-map:
      zone: jiangsu #所在区域
    hostname: localhost #服务主机名称
    prefer-ip-address: false #是否优先使用ip来作为主机名
  server: #eureka服务端配置
    enable-self-preservation: false #关闭eureka服务端的保护机制
```

## 对比和Zookeeper区别

### **1. 回顾CAP原则**

RDBMS (MySQL\Oracle\sqlServer) ===> ACID

NoSQL (Redis\MongoDB) ===> CAP

### **2. ACID是什么？**

- A (Atomicity) 原子性
- C (Consistency) 一致性
- I (Isolation) 隔离性
- D (Durability) 持久性

### **3. CAP是什么?**

- C (Consistency) 强一致性
- A (Availability) 可用性
- P (Partition tolerance) 分区容错性

CAP的三进二：CA、AP、CP

### **4. CAP理论的核心**

- 一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求
- 根据CAP原理，将NoSQL数据库分成了满足CA原则，满足CP原则和满足AP原则三大类
  - CA：单点集群，满足一致性，可用性的系统，通常可扩展性较差
  - CP：满足一致性，分区容错的系统，通常性能不是特别高
  - AP：满足可用性，分区容错的系统，通常可能对一致性要求低一些

### **5. 作为分布式服务注册中心，Eureka比Zookeeper好在哪里？**

著名的CAP理论指出，一个分布式系统不可能同时满足C (一致性) 、A (可用性) 、P (容错性)，由于分区容错性P再分布式系统中是必须要保证的，因此我们只能再A和C之间进行权衡。

- Zookeeper 保证的是 CP —> 满足一致性，分区容错的系统，通常性能不是特别高
- Eureka 保证的是 AP —> 满足可用性，分区容错的系统，通常可能对一致性要求低一些

**Zookeeper保证的是CP**

 当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接收服务直接down掉不可用。也就是说，**服务注册功能对可用性的要求要高于一致性**。但zookeeper会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30-120s，且选举期间整个zookeeper集群是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因为网络问题使得zookeeper集群失去master节点是较大概率发生的事件，虽然服务最终能够恢复，但是，漫长的选举时间导致注册长期不可用，是不可容忍的。

**Eureka保证的是AP**

 Eureka看明白了这一点，因此在设计时就优先保证可用性。**Eureka各个节点都是平等的**，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册时，如果发现连接失败，则会自动切换至其他节点，只要有一台Eureka还在，就能保住注册服务的可用性，只不过查到的信息可能不是最新的，除此之外，Eureka还有之中自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

- Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务
- Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上 (即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其他节点中

因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪

