---
layout: post
title: Java服务端开发：Java注解
categories: 大型系统架构 java之多线程 java之面向对象
tags: Java服务端开发 Java 服务端 服务器 注解 装饰器 Python Spring-Boot Flask 
---

在[《Spring Boot开发Web程序》](http://www.xumenger.com/java-springboot-20180322/)讲到Spring Boot 在做URL 路由的时候使用到了**注解**

```java
package com.example.demo;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
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
}
```

在看[《使用Flask进行简单Web开发》](http://www.xumenger.com/python2-flask-20170701/)中讲到Flask 在做URL 路由的时候是这样的

```python
# -*- coding: utf-8 -*-
from flask import Flask
app = Flask(__name__)

# 主页
@app.route('/')
def index():
    return '<h1>Hello World</h1>'

# 用户URL
@app.route('/user/<name>')
def user(name):
    return '<h1>Hello, %s</h1>' % name

if __name__ == '__main__':
    app.run(debug = True)
```

是不是很像？！那到底Java 的注解和Python 的装饰器

