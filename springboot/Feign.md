# 一、What is Feign

==[Feign](https://github.com/Netflix/feign)是一个声明式的Web服务客户端==。这使得Web服务客户端的写入更加方便 要使用Feign创建一个界面并对其进行注释。它具有可插入注释支持，包括Feign注释和JAX-RS注释。Feign还支持可插拔编码器和解码器。Spring Cloud增加了对Spring MVC注释的支持，并使用Spring Web中默认使用的`HttpMessageConverters`。Spring Cloud集成Ribbon和Eureka以在使用Feign时提供负载均衡的http客户端。

只需创建一个接口, 然后在上面添加注解即可



==Feign 继承了Ribbon, 因此自带负载均衡配置, 默认轮询算法==



# 二、搭建 Feign 项目

在 api 工程建接口

## pom.xml

增加feign组件

```xml
        <!-- Feign 相关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```





## 新建接口DeptClientService

在 api 工程新建

```java
package com.walter.springcloud.service;

import com.walter.springcloud.entities.Dept;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@FeignClient(value = "MICROSERVICECLOUD-DEPT")
public interface DeptClientService {

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    boolean add(Dept dept);

    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    List<Dept> list();
}

```



对 api 项目做 mvn clean 和 mvn install

## 新建 feign 消费端项目

参考原consumer项目, 删除代码

### pom.xml

新增 feign 依赖

```xml
        <!-- Feign 相关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

### controller

直接通过service接口访问

```java
package com.walter.springcloud.controller;

import com.walter.springcloud.entities.Dept;
import com.walter.springcloud.service.DeptClientService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.List;

@RequestMapping(value = "/consumer/dept")
@RestController
public class DeptController_Consumer {

    @Autowired
    private DeptClientService service;

    @RequestMapping(value = "/get/{id}")
    public Dept get(@PathVariable("id") long id) {
        return service.get(id);
    }

    @RequestMapping(value = "/list")
    public List<Dept> list() {
        return service.list();
    }

    @RequestMapping(value = "/add")
    public boolean get(Dept dept) {
        return service.add(dept);
    }
}

```

### 主启动类

增加 `@EnableFeignClients`注解

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableFeignClients
@EnableEurekaClient
@SpringBootApplication
public class DeptConsumer80_Feign_APP {

    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_Feign_APP.class, args);
    }
}

```

### 启动测试

访问http://localhost/consumer/dept/list, 查看轮询效果