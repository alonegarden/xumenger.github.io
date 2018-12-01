---
layout: post
title: 搭建Spring Boot源码环境
categories: 大型系统架构 java之web开发 java之web框架 
tags: Java Web Flask Python Servlet Eclipse Spring SpringBoot maven gradle 
---

>本文基于Spring Boot-2.0.3

首先使用maven（不是gradle） 构建Spring Boot 项目（为了更快的构建，跳过测试用例）

```shell
$ ./mvnw clean install -DskipTests -Pfast
```

然后等待依赖包下载、等待编译

![](../media/image/2018-12-01/01.png)

执行下面的命令

```shell
$ ./mvnw eclipse:eclipse
```

然后将项目导入到Eclipse 中

![](../media/image/2018-12-01/02.png)

编写测试代码如下

```java
package example;

import java.util.Arrays;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;

@RestController
@SpringBootApplication
public class RunServer {

    public static void main(String[] args) {
        SpringApplication.run(RunServer.class, args);
    }

    @RequestMapping("/")
    public String index() {
            return "This is Index Page";
    }
    
    @RequestMapping("/about")
    public String about() {
        return "This is About Page";
    }
    
    @RequestMapping(value="/user/{name}", method= RequestMethod.GET)
    public String user(@PathVariable("name") String name) {
        return "Hello " + name;
    }
    
    @Bean
    public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
        return args -> {
            System.out.println("来看看 SpringBoot 默认为我们提供的 Bean：");
            String[] beanNames = ctx.getBeanDefinitionNames();
            Arrays.sort(beanNames);
            Arrays.stream(beanNames).forEach(System.out::println);
        };
    }
}
```

但是报错

![](../media/image/2018-12-01/03.png)

解决办法是，右键当前项目->【Properties】，引入spring-boot-autoconfigure 项目

![](../media/image/2018-12-01/04.png)

