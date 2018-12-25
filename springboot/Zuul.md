# 一、What is Zuul

Zuul 包含了对请求的路由和过滤两个最主要的功能:

其中路由功能负责将外部请求转发到具体的微服务实例上, 是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预, 是实现请求检验, 服务集合等功能的基础 Zuul 和 Eureka 进行整合, 将 Zuul 自身注册为 Eureka 服务治理下的应用, 同时从 Eureka 中获得其他微服务的消息, 也即以后的访问微服务是通过 Zuul 跳转后获得.

注意: Zuul 服务最终还是会注册进 Eureka



提供 = 代理 + 路由 + 过滤 三大功能



# 二、Zuul 路由配置

新建项目, 增加 zuul 和 eureka 组件, 主启动类增加`@EnableZuulProxy`注解



## 新建项目

新建项目`microservicecloud-zuul-gateway-9527`

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservicecloud</artifactId>
        <groupId>com.walter</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-zuul-gateway-9527</artifactId>

    <dependencies>
        <!-- zuul路由网关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>
</project>
```

## application.properties

鉴于当前用 SpringBoot 2.x ,这里使用properties文件

```properties
server.port=9527

# 对外的微服务名称
spring.application.name=microservicecloud-zuul-gateway

# 增加前缀
zuul.prefix=/walter
# 不允许用原服务名访问
#zuul.ignored-services=microservicecloud-dept
# 将所有的真实服务名都禁止
zuul.ignored-services=*
# 添加服务映射
zuul.routes.mydept.service-id=microservicecloud-dept
# **表示多级url
zuul.routes.mydept.path=/mydept/**

# 客户端注册进eureka服务列表内
eureka.client.service-url.defaultZone=http://eureka7001.com:7001/eureka/,http:/eureka7002.com:7002/eureka/,\
  http:/eureka7003.com:7003/eureka/
# 自定义服务名称信息
eureka.instance.instance-id=gateway-9527.com
# 访问路径可以显示IP地址
eureka.instance.prefer-ip-address=true

info.app.name=walter-microservicecloud
info.company.name=www.baidu.com
info.build.artifactId=${project.artifactId}
info.build.version=${project.version}
```



## hosts

可选, C:\Windows\System32\drivers\etc\hosts 中增加 127.0.0.1  myzuul.com

## 主启动类

`@EnableZuulProxy`注解

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@SpringBootApplication
public class Zuul_9527_StartSpringCloudApp {
    public static void main(String[] args) {
        SpringApplication.run(Zuul_9527_StartSpringCloudApp.class, args);
    }
}

```

## 启动测试

先启动三个 Eureka 服务, 然后启动服务提供者8001,最后启动9527路由模块

测试 eureka7001.com:7001 查看当前路由的模块是否被注册到 eureka 服务中

测试 http://myzuul.com:9527/walter/mydept/dept/get/3 是否能够访问, 以及直接用服务名是否能访问http://myzuul.com:9527/walter/microservicecloud-dept/dept/get/3, 如果不能访问, 则表示配置成功