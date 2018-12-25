# 一、What is Hystrix

Hystrix 能够保证在一个依赖出现超时, 异常等问题的情况下, 不会出现整体服务失败. 避免级联故障, 提高分布式系统弹性



断路器会向调用方返回一个符合预期的, 可处理的备选响应(fallBack), 而不是长时间等待,或者跑出调用方无法处理的异常 , 并会对故障服务进行服务降级, 进而熔断该节点的微服务调用, 快速返回"错误"的返回信息, 检测到该节点响应正常后恢复链路



服务熔断

一种应对雪崩效应的微服务链路保护机制

Hystrix 会监控微服务间的调用状况, 当失败到达一定的阈值, 缺省是5秒内20次调用失败就会启动熔断机制, 注解是`@HystrixCommmand`



服务降级

当系统整体资源不够, 现将某些不重要的服务关掉, 这时客户端需要返回相应的提示信息, 以避免长时间处于等待状态

服务降级只在客户端, 与服务端没关系



# 二、服务熔断

参考服务提供者`microservicecloud-provider-dept-8001`

新建加入了 `Hystrix` 组件的模块`microservicecloud-provider-dept-hystrix-8001`



## pom.xml

```xml
        <!-- hystrix 相关 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```



## application

修改 Eureka 注册实例名

```properties
    instance-id: microservicecloud-dept8001-hystrix 
```



## 业务类

这里只写get方法,  方便测试,  在业务方法上加上`@HystrixCommand`注解

```java
package com.walter.springcloud.controller;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.walter.springcloud.entities.Dept;
import com.walter.springcloud.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class DeptController {

    @Autowired
    private DeptService service;

    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    // 一旦方法失败并跑出异常, 会自动调用HystrixCommand指定的方法
    @HystrixCommand(fallbackMethod = "processHystrix_Get")
    public Dept get(@PathVariable("id") Long id) {
        Dept dept = service.get(id);
        if (dept == null) {
            throw new RuntimeException("该ID: " + id + "没有对应的信息");
        }
        return dept;
    }

    public Dept processHystrix_Get(@PathVariable("id") Long id) {
        return new Dept().setDeptno(id).setDname("该ID: " + id + "没有对应的信息").setDb_source("No this database in my MySQL");
    }

}
```



## 主启动类

加入`@EnableCircuitBreaker`注解

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableCircuitBreaker
@EnableDiscoveryClient
@EnableEurekaClient
@SpringBootApplication
public class DeptProvider8001_Hystrix_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_Hystrix_App.class, args);
    }
}
```

## 启动测试

启动三个eureka模块, 启动加入了hystrix的服务提供者项目, 启动消费者的80项目, 访问http://localhost/consumer/dept/1, 修改最后的id数字, 看看浏览器中是否有返回相应的错误信息



# 三、服务降级

在原 Feign 项目基础上修改, 在配置文件中加入允许熔断机制的配置, 修改api的访问接口文件`DeptClientService`,加上定义好的`fallbackFactory = DeptClientServiceFallbackFactory.class`, 创建实现了`FallbackFactory`的`DeptClientServiceFallbackFactory`, 



## Feign 项目的 application.yml

```yml
feign:
  hystrix:
    enabled: true
```



## api 项目的 Feign 接口文件

`@FeignClient` 加入 `fallbackFactory`参数

```java
package com.walter.springcloud.service;

import com.walter.springcloud.entities.Dept;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory = DeptClientServiceFallbackFactory.class)
public interface DeptClientService {

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    boolean add(Dept dept);

    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    List<Dept> list();
}

```



## FallbackFactory 类

为接口方法配置错误信息, 注意: 这个类需要加`@Component`注解

```java
package com.walter.springcloud.service;

import com.walter.springcloud.entities.Dept;
import feign.hystrix.FallbackFactory;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public boolean add(Dept dept) {
                return false;
            }

            @Override
            public Dept get(long id) {
                return new Dept().setDeptno(id).setDname("该ID: " + id + "没有对应的信息, Consumer 客户端提供的降级信息, " +
                        "此刻服务Provider已经关闭").setDb_source("No this database in my MySQL");
            }

            @Override
            public List<Dept> list() {
                return null;
            }
        };
    }
}
```



## 测试

启动 Feign 类, 停掉`microservicecloud-provider-dept-hystrix-8001`, 访问http://localhost/consumer/dept/1, 看看是否有错误信息返回



# 四、Hystrix Dashbord

准实时的调用监控

持续的记录所有通过Hystrix发起的请求的执行信息, 以图表的方式展示给用户



## 新建工程

新建`microservicecloud-consumer-hystrix-dashboard`



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

    <artifactId>microservicecloud-consumer-hystrix-dashboard</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
    </dependencies>
</project>
```



## application.yml

```yml
server:
  port: 9001
```



## 主启动类

```java
package com.walter.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@EnableHystrixDashboard
@SpringBootApplication
public class DeptConsumer_DashBoard_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_DashBoard_App.class, args);
    }
}
```



## 启动测试

在浏览器中访问 http://localhost:9001/hystrix, 有豪猪图案表示进入hystrix主监控面板

![1545610177609](E:\mdphoto\5C1545610177609.png)

点击 monitor 按钮的页面, 刷新localhost:8001/dept/get/1 , 查看连接数的变化

![1545610440627](E:\mdphoto\5C1545610440627.png)



注意:

- 被监控的服务主要添加`actuator`组件, 才能被监控

  ```xml
          <!-- actuator 监控信息完善 -->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-actuator</artifactId>
          </dependency>
  ```

- 被监控服务必须加上`@EnableCircuitBreaker`注解,才能被hystrix dashboard管理

- 在Springboot 2.x 以上中需要在被监控服务的启动类中加入以下代码

  ```java
      @Bean
      public ServletRegistrationBean getServlet() {
              HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
              ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
              registrationBean.setLoadOnStartup(1);
              registrationBean.addUrlMappings("/hystrix.stream");
              registrationBean.setName("HystrixMetricsStreamServlet");
              return registrationBean;
          }
  ```
