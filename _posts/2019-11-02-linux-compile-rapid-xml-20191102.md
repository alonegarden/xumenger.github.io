---
layout: post
title: rapidxml：一个高效的xml库
categories: c/c++之基础语法 c/c++之预处理 c/c++之面向对象 c/c++之函数 c/c++之指针与内存
tags: Linux g++ rapidxml XML 编译 tinyxml
---

>测试环境：Ubuntu 16.04 LTS、C++11

>rapidxml是一个快速的xml库，官方网站：[http://rapidxml.sourceforge.net/](http://rapidxml.sourceforge.net/)，根据manual 看到，竟然比tinyxml 快了50-100倍

编写测试代码如下

```c++
#include "rapidxml.hpp"
#include "rapidxml_utils.hpp"
#include "rapidxml_print.hpp"
#include "rapidxml_iterators.hpp"

int main()
{
    return 0;
}
```

编译的时候直接报错如下

![](../media/image/2019-11-02/01.png)

这些错误是因为 rapidxml_iterators.hpp 这个头文件导致的。在常规的XML 解析中，其实这个头文件并不是必须的！

所以暂时的解决方法就是先不包含这个头文件！

>[https://stackoverflow.com/questions/3830822/compile-rapidxml-under-linux-with-g](https://stackoverflow.com/questions/3830822/compile-rapidxml-under-linux-with-g)

## 加载XML 文件

用来测试的XML 文件内容如下（有没有发现上面最开始给出的XML 格式是有问题的！！）

```xml
<?xml verson="1.0" encoding="utf-8"?>

<classroom>
    <person>
        <name>Tom</name>
        <age>18</age>
        <address>England</address>
    </person>
    <person>
        <name>John</name>
        <age>19</age>
        <address>England</address>
    </person>
    <person>
        <name>Lily</name>
        <age>18</age>
        <address>American</address>
    </person>
</classroom>
```

编写测试程序如下

```c++
#include "rapidxml/rapidxml.hpp"
//#include "rapidxml/rapidxml_iterators.hpp"
#include "rapidxml/rapidxml_print.hpp"
#include "rapidxml/rapidxml_utils.hpp"

#include <iostream>

int main ()
{
    rapidxml::file<> xmlfile("./classroom.xml");

    std::cout << "----begin" << std::endl;
    std::cout << "----length: " << xmlfile.size() << std::endl;
    std::cout << xmlfile.data() << std::endl;
    std::cout << "----end" << std::endl;

    return 0;
}
```

编译运行

![](../media/image/2019-11-02/02.png)

## 解析XML文件

xml 文件现在加了一些属性

```xml
<?xml verson="1.0" encoding="utf-8"?>

<classroom>
    <person id = "1" gender = "male">
        <name>Tom</name>
        <age>18</age>
        <address>England</address>
    </person>
    <person id = "2" gender = "male">
        <name>John</name>
        <age>19</age>
        <address>England</address>
    </person>
    <person id = "3" gender = "female">
        <name>Lily</name>
        <age>18</age>
        <address>American</address>
    </person>
</classroom>

```

编写代码如下

```c++
#include "rapidxml/rapidxml.hpp"
#include "rapidxml/rapidxml_print.hpp"
#include "rapidxml/rapidxml_utils.hpp"

#include <iostream>

int main ()
{
    rapidxml::file<> xmlfile("./classroom.xml");

    rapidxml::xml_document<> doc;
    doc.parse<0>(xmlfile.data());


    // fitst node of DOM Tree
    rapidxml::xml_node<> *root = doc.first_node();
    std::cout << "root: " << root << std::endl;
    // *root 会输出整个xml 内容
    // std::cout << *root << std::endl;
    std::cout << "root name: " << root->name() << std::endl;
    std::cout << "root name_size: " << root->name_size() << std::endl;


    std::cout << std::endl << "------------------------------------" << std::endl;
    rapidxml::xml_node<> *root_first = root->first_node();
    std::cout << "first node of root: " << std::endl << *root_first << std::endl;


    std::cout << std::endl << "------------------------------------" << std::endl;
    root_first = root->first_node("person");
    rapidxml::xml_attribute<> *attr;
    attr = root_first->first_attribute();
    std::cout << "attr_name: " << attr->name() << std::endl;
    std::cout << "attr_value: " << attr->value() << std::endl;

    attr = attr->next_attribute();
    std::cout << "attr_name: " << attr->name() << std::endl;
    std::cout << "attr_value: " << attr->value() << std::endl;


    std::cout << std::endl << "------------------------------------" << std::endl;
    for (; root_first!=NULL; root_first=root_first->next_sibling())
    {
        std::cout << "sib: " << *root_first << std::endl;
    }

    return 0;
}
```

编译运行（如果不加 -fpermissive，会出现编译报错）

>g++ testxml.cpp -o testxml -fpermissive

![](../media/image/2019-11-02/03.png)

## 参考资料

* [compile rapidxml under linux with g++](https://stackoverflow.com/questions/3830822/compile-rapidxml-under-linux-with-g)
* [c++ rapidxml node_iterator example?](https://stackoverflow.com/questions/2104523/c-rapidxml-node-iterator-example)
* [RAPIDXML Manual](http://rapidxml.sourceforge.net/manual.html)
* [rapidxml,一个快速的xml库](https://www.cnblogs.com/lancidie/archive/2013/04/13/3019527.html)
* [c++开源库rapidxml介绍与示例](https://blog.csdn.net/v_xchen_v/article/details/75634273)
* [编译链接错误及解决方法记录](https://blog.csdn.net/woxiwangxuehaocpp/article/details/43764811)