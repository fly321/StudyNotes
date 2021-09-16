# SpringBoot_JPA实现Rest

## 1、构建项目

> 导入依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.6</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.22</version>
</dependency>
<!--lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.16</version>
    <scope>provided</scope>
</dependency>
```

> 添加yml配置

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/ssm?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
  jpa:
    hibernate:
      ddl-auto: update
    database: mysql
    show-sql: true
```

> 新建测试类

```java
package com.example.restful.pojo;

import lombok.Data;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity(name = "t_book")
@Data
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private String author;

}
```

```java
package com.example.restful.repository;

import com.example.restful.pojo.Book;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book,Integer> {

}
```

经过如上几步，一个RESTFUL服务就构建成功了

## 2、添加测试

运行后自动创建表

Hibernate: create table t_book (id integer not null auto_increment, author varchar(255), name varchar(255), primary key (id)) engine=MyISAM

使用postman进行测试

> 新增(POST)

![image-20210620145310373](C:\Users\sky\AppData\Roaming\Typora\typora-user-images\image-20210620145310373.png)

> 查询(GET请求)

http://127.0.0.1:8080/books

![image-20210620145642136](C:\Users\sky\AppData\Roaming\Typora\typora-user-images\image-20210620145642136.png)

若要查询具体在后面加id

http://127.0.0.1:8080/books/3

> 分页

查询第2页数据并且每页记录数为3,page按数组从0开始

http://127.0.0.1:8080/books?page=1&size=3

若要排序，如按id倒序排序

http://127.0.0.1:8080/books?page=1&size=3&sort=id,desc

> 修改(PUT请求)

http://127.0.0.1:8080/books/2

![image-20210620152759387](C:\Users\sky\AppData\Roaming\Typora\typora-user-images\image-20210620152759387.png)

> 删除(DELETE请求)

http://127.0.0.1:8080/books/2

## 3、自定义请求路径

```java
package com.example.restful.repository;

import com.example.restful.pojo.Book;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;
import org.springframework.data.rest.core.annotation.RestResource;

@RepositoryRestResource(path = "rest",collectionResourceRel = "collection",itemResourceRel = "item")
public interface BookRepository extends JpaRepository<Book,Integer> {

    @RestResource(path = "name" ,rel = "name")
    Book findByNameEquals(@Param("name") String name);
}

```

@RepositoryRestResource(path = "rest",collectionResourceRel = "collection",itemResourceRel = "item")注解

path 将所有请求路径中的books都修改为rest，collectionResourceRel 是返回json集合中key的名字，itemResourceRel 是返回单个key的名字

> 自定义方法查询

​	自定义查询值需要在BookRepository中定义相关查询方法，方法定义好可以不使用@RestResource注解

默认路径就是http://127.0.0.1:8080/rest/search/findByNameEquals?name=红楼梦

![image-20210620155646729](C:\Users\sky\AppData\Roaming\Typora\typora-user-images\image-20210620155646729.png)

@RestResource(path = "name" ,rel = "name")，path是最后访问路径的name，rel是返回json的key可以作为提示。

## 4、隐藏方法

默认情况下，但凡继承了Repository接口或者是子类都会被暴露出来

若有不想暴露的接口使用

@RepositoryRestResource(exported = false)放置在类上，使得整个类的操作全部关闭

@RestResource(exported = false)放置在方法上，屏蔽单独一个方法