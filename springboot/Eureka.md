#  一、Eureka是什么

> Eureka是Netflix的一个子模块, 也是核心模块之一. Eureka是一个基于REST的服务, 用于定位服务, 以实现远端中间层服务发现和故障转移. 服务祖册与发现对于微服务甲沟炎来说是非常重要的, 有了服务发现与注册, ==只需要使用服务的标识符, 就可以访问到服务==,  而不需要修改服务调用的配置文件了. ==功能类似于dubbo的注册中心, 比如Zookeeper 在设计上遵循 AP 原则.==

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来==实现服务注册和发现(对比Zookeeper)==

Eureka 采用 CS 的设计架构. Eereka Server 作为服务注册功能的服务器, 他是服务注册公司

而系统的其他微服务, 使用 Eureka 的客户端连接带 Eereka Server 并维持心跳连接, 这样系统的维护人员就可以通过 Eereka Server 来监控系统中各个微服务是否正常运行. Spring Cloud 的一些其他模块 (比如Zuul) 就可以通过 Eereka Server 来发现系统中的其他微服务, 并执行相关的逻辑. 

==Eureka 包含两个组件 Eereka Server 和 Eereka Client==

- Eereka Server : 提供服务注册服务, 各个节点启动后会在 Eereka Server 中进行注册
- Eereka Client  : java 客户端, 用于与 Server 进行交互

Eureka 与 Dubbo 架构对比图

![Eureka 架构图](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545233258035.png)



![Dubbo 架构图](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545233391305.png)

# 二、项目构建

> 构建原理
>
> Eureka Server 提供服务注册和发现
> Service Provider 服务提供方将自身服务注册带 Eureka, 从而使消费方能够找到
> Service Consumer 服务消费方从 Eureka 获取注册服务列表, 从而能够消费服务

当前工程情况

- 总父工程
- 通用模块 api
- 服务提供者 Provider
- 服务消费者 Consumer

## 1. 构建 Server 端

创建模块 `microservicecloud-eureka-7001` , 作为 eureka 服务注册中心

在 pom 中引入 eureka 相关组件

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
```

在配置文件中配置实例名与交互地址, 作为服务注册中心, 将自身排除在服务注册之外

```yml
eureka:
  instance:
    hostname: localhost  # eureka 服务端实例名称
  client:
    register-with-eureka: false  # false 表示不向注册中心注册自己
    fetch-registry: false  # false 表示自己端就是注册中心, 我的职责就是维护服务实例, 并不需要去检索服务
    service-url:
      # 设置与 Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

在启动类上加上主键注解

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServer7001_App {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class, args);
    }
}
```



相关文件

- pom.xml
- application.yml
- EurekaServer7001_App.java

### Server 模块

模块名 `microservicecloud-eureka-7001`

![eureka 服务模块](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545239376704.png)

### pom.xml 

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

    <artifactId>microservicecloud-eureka-7001</artifactId>

    <dependencies>
        <!--eureka-server服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 修改后立即生效，热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```



### application.yml

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost  # eureka 服务端实例名称
  client:
    register-with-eureka: false  # false 表示不向注册中心注册自己
    fetch-registry: false  # false 表示自己端就是注册中心, 我的职责就是维护服务实例, 并不需要去检索服务
    service-url:
      # 设置与 Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 主启动类

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServer7001_App {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class, args);
    }
}

```

### 启动测试

根据配置的地址 `service-url` , 在浏览器中访问, 比如本次地址是 http://localhost:7001/ ,如果在浏览器中看到如下标志, 表示配置成功

![eureka配置成功](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545239986468.png)

如果页面中显示`No instances available` 表示没有服务注册进来

## 2.  Eureka 注册

> 将微服务提供者注册到 Eureka Server

这里将 `microservicecloud-provider-dept-8001` 注册到 `microservicecloud-eureka-7001`

在原工程上修改, 引入组件, 增加配置, 加上`@EnableEurekaClient`注解

### pom.xml

在服务提供者的 maven 中加入 `eureka-client` 相关依赖

```xml
        <!-- 将微服务 privider 侧注册进 erureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

### application

配置文件 `application.yml` 中加入配置信息, 主要是 Eureka 服务端的访问地址

```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
```

或 (properties)

```properties
#客户端注册进eureka服务列表内
eureka.client.service-url.defaultZone=http://localhost:7001/eureka/
```

### 主启动类

在主启动类上加上`@EnableEurekaClient`注解

```java
@EnableEurekaClient
@SpringBootApplication
public class DeptProvider8001_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```

### 测试

重新启动服务提供者模块, 如果在服务端 (http://localhost:7001) 页面看到如下信息, 表示服务注册成功

![eureka 注册成功](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545243132558.png)

项目名称 `MICROSERVICECLOUD-DEPT` 是在服务提供者配置文件中配置的

```properties
# 对外的微服务名称
spring.application.name=microservicecloud-dept
```



## 3. actuator 与注册微服务信息完善

### 1). 主机名称 : 服务名称修改

目的: 修改 Eureka 页面中微服务的 Status 的名字

方法: 在注册服务的配置文件中, 添加 eureka 的客户端实例别名 `eureka.instance.instance-id`

```properties
# 客户端注册进eureka服务列表内
eureka.client.service-url.defaultZone=http://localhost:7001/eureka/
# 自定义服务名称信息
eureka.instance.instance-id=microservicecloud-dept8001
```

或者 (yml)

```yml
eureka:
  client:  # 客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001  # 自定义服务名称信息
```

如果配置成功, 即可达到上图效果

![status 信息修改](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545248163884.png)

### 2). 访问信息有 ip 提示

目的: 在鼠标放在 `Status` 的信息上时, 浏览器左下角会显示 ip 地址

方法: 同上, 在配置文件中加上 `eureka.instance.prefer-ip-address=true`

```properties
# 客户端注册进eureka服务列表内
eureka.client.service-url.defaultZone=http://localhost:7001/eureka/
# 自定义服务名称信息
eureka.instance.instance-id=microservicecloud-dept8001
# 访问路径可以显示IP地址
eureka.instance.prefer-ip-address=true
```

或者 (yml)

```yml
eureka:
  client:  # 客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: microservicecloud-dept8001  # 自定义服务名称信息
    prefer-ip-address: true  # 访问路径可以显示IP地址
```

添加后的效果如下

![显示微服务 ip](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545249217206.png)



### 3). 微服务 info 内容详细信息

目的: 在点开 Eureka 的服务页面的微服务连接时, 会展示相关微服务信息, 而不是 ErrorPage

方法: 

- 服务提供者 pom 添加 `actuator` 组件, 该组件主管监控和信息配置

  ```xml
          <!-- 将微服务 privider 侧注册进 erureka -->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
          </dependency>
  ```

- 在总父工程的pom中添加构建信息, 用于给配置文件中的 `info`配置注入项目信息

  ```xml
      <build>
          <!-- 父工程名称 -->
          <finalName>microservicecloud</finalName>
          <resources>
              <resource>
                  <!-- 允许访问所有项目 src/main/resources 路径下的内容 -->
                  <directory>src/main/resources</directory>
                  <filtering>true</filtering>
              </resource>
          </resources>
          <plugins>
              <plugin>
                  <!-- 负责解析 resource 文件 -->
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-resources-plugin</artifactId>
                  <configuration>
                      <!-- 以 $ 开头的文件 -->
                      <delimiters>
                          <delimit>$</delimit>
                      </delimiters>
                  </configuration>
              </plugin>
          </plugins>
      </build>
  ```

- 服务提供者的配置文件中加入 `info`

  ```properties
  info.app.name=walter-microservicecloud
  info.company.name=www.baidu.com
  info.build.artifactId=${project.artifactId}
  info.build.version=${project.version}
  ```

  或者 (yml)

  ```yml
  info:
    app.name: walter-microservicecloud
    company.name: www.baidu.com
    build.artifactId: ${project.artifactId}
    build.version: ${project.version}
  ```

  $ 开头的变量将由 maven 注入

 配置后, 在点开 Eureka 服务页面的微服务连接页面

![info 页面信息](C:\Users\walter\AppData\Roaming\Typora\typora-user-images\1545251252847.png)



## 4. Eureka 自我保护

```text
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
```

