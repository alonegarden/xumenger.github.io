---
layout: post
title: OpenResty实践
categories: lua之基础 大型系统架构 
tags: nginx web http lua OpenResty Mac c c++ 日志 
---

## 部署OpenResty环境

首先安装OpenResty依赖的库`brew install pcre openssl`

到OpenResty的[官网下载页面](http://openresty.org/cn/download.html)下载源码。我选择的是openresty-1.9.15.1.tar.gz

下面就是编译、安装的流程

```
$ tar -xzvf openresty-1.9.15.1.tar.gz
$ cd openresty-1.9.15.1
$ ./configure \
   --with-cc-opt="-I/usr/local/opt/openssl/include/ -I/usr/local/opt/pcre/include/" \
   --with-ld-opt="-L/usr/local/opt/openssl/lib/ -L/usr/local/opt/pcre/lib/" \
   -j8
$ make
$ sudo make install
```

安装的目录是/usr/local/openresty

![](../media/image/2018-06-13/01.png)

## 最简单的测试例子

OpenResty 安装之后就有配置文件及相关的目录的，为了工作目录与安装目录互不干扰，并顺便学下简单的配置文件编写，我们另外创建一个 OpenResty 的工作目录来练习，并且另写一个配置文件。我选择在当前用户目录下创建 OpenResty 目录，并在该目录下创建 logs 和 conf 子目录分别用于存放日志和配置文件

![](../media/image/2018-06-13/02.png)

**创建配置文件**

在conf下创建配置文件为nginx.conf，内容为

```nginx
# nginx worker 数量
worker_processes  1;

# 指定错误日志文件路径
error_log /Users/xumenger/Desktop/code/OpenResty/logs/error.log;

events {
    worker_connections 1024;
}


http {
    server {
        # 监听端口，若你的6699端口已经被占用，则需要修改
        listen 6699;
        location / {
            default_type text/html;

            # 使用lua的代码输出应答体内容
            content_by_lua_block {
                ngx.say("HelloWorld")
            }
        }
    }
}
```

然后我们用下面的命令启动Nginx

>/usr/local/openresty/nginx/sbin/nginx -c ./conf/nginx.conf -p .

![](../media/image/2018-06-13/03.png)

然后在浏览器上输入`http://127.0.0.1:6699/`，访问效果如下

![](../media/image/2018-06-13/04.png)

另外，我们可以在logs目录下看到该次HTTP请求的日志

![](../media/image/2018-06-13/05.png)

>详细的日志是帮助我们分析用户行为、Web攻击等必不可少的东西！



## 参考资料

* [《使用Nginx+Lua(OpenResty)开发高性能Web应用》](http://jinnianshilongnian.iteye.com/blog/2280928)
* [《跟我学OpenResty(Nginx+Lua)》](http://jinnianshilongnian.iteye.com/blog/2190344)
* [《OpenResty最佳实践》](https://moonbingbing.gitbooks.io/openresty-best-practices/content/)
* [《深入 Nginx：我们是如何为性能和规模做设计的》](http://blog.jobbole.com/88766/)
* [《使用Nginx和uWSGI部署Flask程序》](http://www.xumenger.com/nginx-flask-python-20180331/)
* [《Lua简明教程》](http://www.xumenger.com/lua-20180126/)
* [《Lua与C/C++混合编程》](http://www.xumenger.com/lua-c-cpp-20180202/)
