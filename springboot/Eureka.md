#  一、Eureka是什么

> Eureka是Netflix的一个子模块, 也是核心模块之一. Eureka是一个基于REST的服务, 用于定位服务, 以实现远端中间层服务发现和故障转移. 服务祖册与发现对于微服务甲沟炎来说是非常重要的, 有了服务发现与注册, ==只需要使用服务的标识符, 就可以访问到服务==,  而不需要修改服务调用的配置文件了. ==功能类似于dubbo的注册中心, 比如Zookeeper 在设计上遵循 AP 原则.==

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来==实现服务注册和发现(对比Zookeeper)==

Eureka 采用 CS 的设计架构. Eereka Server 作为服务注册功能的服务器, 他是服务注册公司

而系统的其他微服务, 使用 Eureka 的客户端连接带 Eereka Server 并维持心跳连接, 这样系统的维护人员就可以通过 Eereka Server 来监控系统中各个微服务是否正常运行. Spring Cloud 的一些其他模块 (比如Zuul) 就可以通过 Eereka Server 来发现系统中的其他微服务, 并执行相关的逻辑.

==Eureka 包含两个组件 Eereka Server 和 Eereka Client==

- Eereka Server : 提供服务注册服务, 各个节点启动后会在 Eereka Server 中进行注册
- Eereka Client  : java 客户端, 用于与 Server 进行交互

Eureka 与 Dubbo 架构对比图

![Eureka 架构图](https://github.com/walter9527/mdphoto/raw/master/eureka%E6%9E%B6%E6%9E%84%E5%9B%BE.png)



![Dubbo 架构图](https://github.com/walter9527/mdphoto/raw/master/dubbo%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

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

![eureka 服务模块](https://github.com/walter9527/mdphoto/raw/master/%E6%9C%8D%E5%8A%A1%E6%8F%90%E4%BE%9B%E8%80%85%E6%A8%A1%E5%9D%97.png)

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
    register-with-eureka: false  # 是否将自己注册到Eureka, false表示不向注册中心注册自己
    fetch-registry: false  # 是否同步其他节点信息, false 表示不需要, 这里是单点服务
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

![eureka配置成功](https://github.com/walter9527/mdphoto/raw/master/eureka%E9%85%8D%E7%BD%AE%E6%88%90%E5%8A%9F.png)

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

![eureka 注册成功](https://github.com/walter9527/mdphoto/raw/master/eureka%E6%B3%A8%E5%86%8C%E6%88%90%E5%8A%9F.png)

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

![status 信息修改](https://github.com/walter9527/mdphoto/raw/master/eureka%E4%BF%A1%E6%81%AF%E4%BF%AE%E6%94%B9.png)

### 2). 访问信息有 ip 提示

目的: 在鼠标放在 `Status` 的信息上时, 浏览器左下角会显示 ip 地址

方法: 同上, 在配置文件中加上 `eureka.instance.prefer-ip-address=true`

```properties
# 客户端注册进eureka服务列表内
eureka.client.service-url.defaultZone=http://localhost:7001/eureka/
# 自定义服务名称信息
eureka.instance.instance-id=microservicecloud-dept8001
# 访问路径可以显示IP地址, 如果不设置或false, 则表示将微服务所在操作系统的hostname注册到微服务
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

![显示微服务 ip](https://github.com/walter9527/mdphoto/raw/master/%E6%98%BE%E7%A4%BA%E5%BE%AE%E6%9C%8D%E5%8A%A1ip.png)



### 3). 微服务 info 内容详细信息

目的: 在点开 Eureka 的服务页面的微服务连接时, 会展示相关微服务信息, 而不是 ErrorPage

方法: 服务提供者 pom 添加 `actuator` 组件, 该组件主管监控和信息配置

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

![info 页面信息](https://github.com/walter9527/mdphoto/raw/master/info%E9%A1%B5%E9%9D%A2%E4%BF%A1%E6%81%AF.png)



## 4. Eureka 自我保护

```text
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. 
RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
```

某时刻某个微服务不可用了, eureka 不会立即清理, 依旧会对该微服务的信息进行保存

> 默认情况下, 如果 Eureka Server 在一定时间内没有接收到某个实例服务的心跳, Eureka Server将会注销实例 (默认90秒). 但是当网络分区故障发生时, 微服务与 Eureka Server 之间无法正常通信, 以上行为可能变得非常危险了 ---- 因为微服务本身其实是健康的, ==此时本不该注销这个微服务==. Eureka 通过"自我保护模式"来解决这个问题 ---- 当 Eureka Server 节点在短时间内丢失过多客户端时 (可能发生了网络分区故障) , 那么这个节点就会进入自我保护模式. 一旦进入该模式, Eureka Server 就会保护服务注册表中的信息, 不再删除服务注册表中的数据 (也就是不会注销任何微服务). 当网络故障恢复后, 该 Eureka Server 节点会自动退出保护模式



> ==在自我保护模式中, Eureka Server 会保护服务注册表中的信息, 不再注销任何服务实例. 当它收到的心跳数重新恢复到阈值以上时, 该 Eureka Server 节点就会自动退出自我保护模式. 它的设计哲学是宁可保留错误的服务注册信息, 也不盲目注销任何可能健康的服务实例. 一句话讲解: 好死不如赖活着==



自我保护模式是为了应对网络异常,  为避免将健康的服务注销, 宁可保留所有服务, 包括不健康的服务.

这样可以使 Eureka 集群更加健壮, 稳定



如何禁用自我保护模式, ==不推荐==

```properties
eureka.server.enable-self-preservation=false
```



## 5. 服务发现

对外暴露服务信息, 让外部可以查到在 Eureka 上注册的微服务信息



方法: 使用`DiscoveryClient`获取微服务注册信息, 主启动类需加上注解`@EnableDiscoveryClient`

```java
// 获取当前注册的所有微服务
List<String> services = client.getServices();
// 获取 MICROSERVICECLOUD-DEPT8001 服务信息
List<ServiceInstance> instances = client.getInstances("MICROSERVICECLOUD-DEPT");
```



实现:

服务提供者`microservicecloud-provider-dept-8001`



 `DeptController.lava`

```java
@Autowired
private DiscoveryClient client;

@RequestMapping(value = "/dept/discovery", method = RequestMethod.GET)
public Object discovery() {
    // 获取当前注册的所有微服务
    List<String> services = client.getServices();
    System.out.println("********************" + services);

    // 获取 MICROSERVICECLOUD-DEPT8001 服务信息
    List<ServiceInstance> instances = client.getInstances("MICROSERVICECLOUD-DEPT");
    for (ServiceInstance instance : instances) {
        System.out.println(instance.getServiceId() + "\t" + instance.getHost() + "\t"
                           + instance.getPort() + "\t" + instance.getUri());
    }
    return this.client;
}
```



主启动类

```java
@EnableDiscoveryClient
@EnableEurekaClient
@SpringBootApplication
public class DeptProvider8001_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class, args);
    }
}
```



微服务消费者`microservicecloud-consumer-dept-80`

`DeptController_Consumer.java`

```java
/**
     * 测试 @EnableDiscoveryClient, 消费端可以调用服务发现
     *
     */
@RequestMapping(value = "/discovery")
public Object discovery() {
    return restTemplate.getForObject(REST_URL_PREFIX + "/dept/discovery", Object.class);
}

@RequestMapping(value = "/discovery1")
public Object discovery1() {
    Mono<Object> objectMono = webClient.get()
        .uri("/dept/discovery")
        .retrieve()
        .bodyToMono(Object.class);

    return objectMono.block();
}
```



## 6. Eureka 集群

同时启动多个 Eureka 服务, 并在 `defaultZone` 中添加其他服务的地址, 就可以实现集群



### 1). 增加 Eureka 服务

根据 `microservicecloud-eureka-7001`, 增加模块`microservicecloud-eureka-7002`, `microservicecloud-eureka-7003`, 分别配置端口 7002, 7003



### 2).修改各自配置文件中的 defaultZone

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com  # eureka 服务端实例名称
  client:
    register-with-eureka: false  # false 表示不向注册中心注册自己
    fetch-registry: false  # false 表示自己端就是注册中心, 我的职责就是维护服务实例, 并不需要去检索服务
    service-url:
      # 设置与 Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

`hostname`更改 (可选),

如果 windows 本机要更改成 `localhost` 意外的值,  则需要更改`hosts`文件

`C:\Windows\System32\drivers\etc\hosts'

```text
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
```



### 3). 更改服务提供者的 defaultZone

```yml
eureka:
  client:  # 客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http:/eureka7002.com:7002/eureka/,http:/eureka7003.com:7003/eureka/
```



### 4). 启动测试

![1545418545455](https://github.com/walter9527/mdphoto/raw/master/1545418545455.png)



# 三、对比Zookeeper



## CAP理论

> CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，
> ==最多只能同时较好的满足两个==。
> 因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：
> CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
> CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。
> AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

![1545419316551](https://github.com/walter9527/mdphoto/raw/master/1545419316551.png)





## Zookeeper 保证的是 CP

Zookeeper 的master节点在失去网络联系的时候, 剩余的节点会进行 leader 选举,  选举时间在 30s ~ 120s 之间 , 这段时间 zk 集群是不可用的, 导致选举期间服务瘫痪(不保证高可用), 但是能够保证注册信息是最新的(强一致)



## Eureka 保证的是 AP

Eureka 在设计时即保证高可用. ==Eureka 各节点都是平等的==, 几个节点挂掉不会影响正常节点正常工作, 如果客户端发现某个节点连接失败, 会自动转向其他节点, 只要有一个还在, 几乎能保证服务可用(高可用性), 只是查到的信息可能不是最新的(不保证强一致), 此外, Eureka 还有自我保护机制, 如果在15分钟内85%的节点都没有心跳, 那么 Eureka 就认为客户端与注册中心出现了网络故障, 此时会出现以下几种情况:

- Eureka 不再从注册表中移除因为长时间没收到心跳而应该过期的服务
- Eureka 仍能够接受新服务的注册和查询请求, 但不会被同步到其他节点上(既保证当前节点依然可用)
- 当网络稳定时, 当前实例新的注册信息会被同步到其他节点中



==因此, Eureka 可以很好的应对因网络故障导致部分节点失去联系的情况, 而不会像 Zookeeper 那样使整个注册服务瘫痪==

