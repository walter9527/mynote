# 一、What is Config

分布式系统面临的配置问题:

> 微服务意味着要将单体应用中的业务拆分成一个个子服务, 每个服务的粒度相对较小, 因此系统中会出现大量的服务, 由于每个服务都需要必要的配置信息才能运行, 所以一套集中式的, 动态的配置管理设施是必不可少的.  SpringCloud 提供了 ConfigServer 来解决这个问题. 我们每一个微服务自己带一个application.yml. 上百个配置文件的管理......



SpringCloud Config 为微服务架构中的微服务提供集中化的外部配置支持, 配置服务器为**各个不同微服务应用**的所有环境提供了一个**中心化的外部配置**



**SpringCloud Config 分为服务端和客户端两部分**

服务端也称为分布式配置中心, **他是一个独立的微服务应用**, 用来连接配置服务器并为客户端提供获取配置信息, 加密/解密信息等访问接口



客户端则是通过指定的配置中心来管理应用资源, 以及与业务相关的配置内容, 并在启动的时候从配置中心获取和加载配置信息. 配置服务器默认采用git来存储配置信息, 这样就有助于对环境配置进行版本管理, 并且可以通过git客户端工具来方便的管理和访问配置内容

![1545713675337](E:\mdphoto\5C1545713675337.png)

功能

- 集中管理配置文件
- 根据不同的环境使用不同的配置, 动态化的配置更新, 分环境部署比如 dev/test/prod/beta/release
- 服务会向配置中心统一拉取配置自己的信息, 不需要单独配置
- 配置发生变化时, 服务不需要重启, 就可以感知配置的变化, 并应用新的配置(需用到 SpringCloud Bus)
- 将配置信息以 REST 接口的形式暴露



需要与 GitHub 进行整合, 推荐用git, 而且是http/https的方式



# 二、Config 服务端

新建模块, 增加`spring-cloud-config-server`组件,  配置文件加入`spring.cloud.config.server.git.uri`的github地址, 主启动类上加上`@EnableConfigServer`注解



在此之前, 需要先在github上创建仓库`microservicecloud-config`, 以放置配置文件, 在本地clone项目, 新建文件`application.yml`,内容如下, **务必保存为UTF-8**, 然后上传github

```yml
spring:
  profiles:
    active:
    - dev
---
spring:
  profiles: dev  # 开发环境
  application:
      name: microservicecloud-config-walter-dev
---
spring:
  profiles: test  # 测试环境
  application:
      name: microservicecloud-config-walter-test
 # 保存为UTF-8格式

```





## 新建模块

新建`microservicecloud-config-3344`

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

    <artifactId>microservicecloud-config-3344</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
    </dependencies>
</project>
```

## application.yml

注意, 对于springboot2.x, 最好用http 的方式, ssh可能会报错

```yml
server:
  port: 3344

spring:
  application:
    name: microservicecloud-config
  cloud:
    config:
      server:
        git:
          uri: https://github.com/walter9527/microservicecloud-config.git  # github上面的git仓库的名称
```

## 主启动类

添加`@EnableConfigServer`注解

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@SpringBootApplication
public class Config_3344_StartCloudApp {
    public static void main(String[] args) {
        SpringApplication.run(Config_3344_StartCloudApp.class, args);
    }
}

```

## 启动测试

在windows下修改配置文件hosts, 新增127.0.0.1  config-3344.com, 这个可选

启动3344服务

http://config-3344.com:3344/application-dev.yml

```yml
spring:
  application:
    name: microservicecloud-config-walter-dev
  profiles:
    active:
    - dev
```



http://config-3344.com:3344/application-test.yml

```yml
spring:
  application:
    name: microservicecloud-config-walter-test
  profiles:
    active:
    - dev
```



http://config-3344.com:3344/application-xxx.yml(不存在的配置)

```yml
spring:
  profiles:
    active:
    - dev
```



查看是否会出现相应的配置



访问配置文件的方式

```text
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

即

```reStructuredText
/application/dev      /application/dev/master
/application-dev.yml
/master/application-dev.yml
/application-dev.properties
/master/application-dev.properties
```



# 三、Config 客户端

客户端建立`bootstrap.xml`来从服务端获取配置文件, pom中要加上config客户端组件`spring-cloud-starter-config`



## github配置文件

配置文件`microservicecloud-config-client.yml`, 注意端口不同

```yml
spring:
  profiles:
    active:
    - dev
---
server:
  port: 8201
spring:
  profiles: dev
  application:
    name: microservicecloud-config-client
eureka:
  client:                           
    service-url:
      defaultZone: http://eureka-dev:7001/eureka
---
server:
  port: 8202
spring:
  profiles: test
  application:
    name: microservicecloud-config-client
eureka:
  client:                           
    service-url:
      defaultZone: http://eureka-test:7001/eureka

```

## 新建模块

新建`microservicecloud-config-client-3355`

## pom.xml

注意 除了config客户端组件外, 还要加上web组件

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

    <artifactId>microservicecloud-config-client-3355</artifactId>

    <dependencies>
        <!-- SpringCloud Config客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```



## bootstrap.xml

bootstrap.xml是系统级的配置文件, 优先级更高, 主要作用是从外部获取并解析配置信息

```yml
spring:
  cloud:
    config:
      name: microservicecloud-config-client #需要从github上读取的资源名称. 注意没有yml后缀名
      profile: dev  #本次访问的配置项
      label: master
      uri: http://config-3344.com:3344 #本微服务启动后先去找3344号服务. 通过SpringCloudConfig获取GitHub的服务地址
```



## application.yml

用户级的配置文件, 可选

```yml
spring:
  application:
    name: microservicecloud-config-client
```



## rest类

主要是测试客户端是否能从github上读取信息

```java
package com.walter.springcloud.rest;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigClientRest {

    // 读取配置文件的信息
    @Value("${spring.application.name}")
    private String applicationName;

    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServers;

    @Value("${server.port}")
    private String port;

    @RequestMapping("/config")
    public String getConfig()
    {
        String str = "applicationName: " + applicationName + "\t eurekaServers:" + eurekaServers + "\t port: " + port;
        System.out.println("******str: " + str);
        return "applicationName: " + applicationName + "\t eurekaServers:" + eurekaServers + "\t port: " + port;
    }
}

```



## 主启动类

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConfigClient_3355_StartSpringCloudApp {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient_3355_StartSpringCloudApp.class, args);
    }
}

```

## 启动测试

启动3344, 3355模块

hosts添加 127.0.0.1  client-config.com , 这个可选

访问  client-config.com:8201/config,, 查看是否有页面信息, 切换test文件, 访问 client-config.com:8202/config





# 四、模拟github统一配置

构建一个 eureka + Dept提供方的微服务, 两者的配置统一有github管理



## github配置文化

microservicecloud-config-eureka-client.yml

```yml
spring:
  profiles:
    active:
    - dev
--- 
server:
  port: 7001 #注册中心占用7001端口, 冒号后面必须要有空格

spring:
  profiles: dev
  application:
    name: microservicecloud-config-eureka-client

eureka:
  instance:
    hostname: eureka7001.com  # eureka 服务端实例名称
  client:
    register-with-eureka: false  # false 表示不向注册中心注册自己
    fetch-registry: false  # false 表示自己端就是注册中心, 我的职责就是维护服务实例, 并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
--- 
server:
  port: 7001 #注册中心占用7001端口, 冒号后面必须要有空格

spring:
  profiles: test
  application:
    name: microservicecloud-config-eureka-client

eureka:
  instance:
    hostname: eureka7001.com  # eureka 服务端实例名称
  client:
    register-with-eureka: false  # false 表示不向注册中心注册自己
    fetch-registry: false  # false 表示自己端就是注册中心, 我的职责就是维护服务实例, 并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```



microservicecloud-config-dept-client.yml

注意: dev和test的数据库不一样

```yml
spring:
  profiles:
    active:
    - dev
---
server:
  port: 8001    
spring:
  profiles: dev
  application:
    name: microservicecloud-config-dept-client
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: com.mysql.cj.jdbc.Driver              # mysql驱动包
    url: jdbc:mysql://192.168.184.128:3306/cloudDB03              # 数据库名称
    username: dev
    password: 123456
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200    
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.walter.springcloud.entities    # 所有Entity别名类所在包
  
eureka:
  client:  # 客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
  instance:
    instance-id: dept-8001.com  # 自定义服务名称信息
    prefer-ip-address: true  # 访问路径可以显示IP地址
 
info:
  app.name: walter-microservicecloud-springcloudconfig02
  company.name: www.baidu.com
  build.artifactId: ${project.artifactId}
  build.version: ${project.version} 
---
server:
  port: 8001    
spring:
  profiles: test
  application:
    name: microservicecloud-config-dept-client
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: com.mysql.cj.jdbc.Driver              # mysql驱动包
    url: jdbc:mysql://192.168.184.128:3306/cloudDB02              # 数据库名称
    username: dev
    password: 123456
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200    
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.walter.springcloud.entities    # 所有Entity别名类所在包
  
eureka:
  client:  # 客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
  instance:
    instance-id: dept-8001.com  # 自定义服务名称信息
    prefer-ip-address: true  # 访问路径可以显示IP地址
 
info:
  app.name: walter-microservicecloud-springcloudconfig02
  company.name: www.baidu.com
  build.artifactId: ${project.artifactId}
  build.version: ${project.version} 

```



## eureka 部分

新建模块`microservicecloud-config-eureka-client-7001`

### pom.xml

比原来新增config客户端组件

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

    <artifactId>microservicecloud-config-eureka-client-7001</artifactId>

    <dependencies>
        <!--eureka-server服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- SpringCloud Config客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
    </dependencies>
</project>
```

### bootstrap.yml

```yml
spring:
  cloud:
    config:
      name: microservicecloud-config-eureka-client #需要从github上读取的资源名称. 注意没有yml后缀名
      profile: dev  #本次访问的配置项
      label: master
      uri: http://config-3344.com:3344 #本微服务启动后先去找3344号服务. 通过SpringCloudConfig获取GitHub的服务地址
```

### application.yml

```yml
spring:
  application:
    name: microservicecloud-config-eureka-client
```

### 主启动类

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class Config_Git_EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(Config_Git_EurekaServerApplication.class, args);
    }
}

```

### 启动测试

访问 eureka7001.com:7001 查看是否配置成功,



## dept部分

新建模块`microservicecloud-config-dept-client-8001`

### pom.xml

在原基础上增加config客户端配置

```xml
        <!-- SpringCloud Config客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

### bootstrap.yml

```yml
spring:
  cloud:
    config:
      name: microservicecloud-config-dept-client #需要从github上读取的资源名称. 注意没有yml后缀名
      profile: dev  #本次访问的配置项
      label: master
      uri: http://config-3344.com:3344 #本微服务启动后先去找3344号服务. 通过SpringCloudConfig获取GitHub的服务地址
```

### application.yml

```yml
spring:
  application:
    name: microservicecloud-config-dept-client
```

其余一切不变



### 启动测试

查看dept是否注册成功, 以及修改profile后, 访问的数据库是否改变  localhost:8001/dept/list