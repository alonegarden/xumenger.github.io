---
layout: post
title: Spring浅谈：声明式事务原理
categories: java之web开发 java之web框架 数据库之mysql
tags: java spring 切面 面向切面编程 AOP AspectJ 面向切面 编译 装饰器设计模式 声明式事务 IoC IoC容器 spring-context spring-aspects pom Maven 增强器 通知方法 代理对象 Cglib代理 事务 数据库 声明式事务 MySQL MyBatis 嵌套事务 @Transactional @EnableTransactionManagement TransactionManagementConfigurationSelector AutoProxyRegistrar ProxyTransactionManagementConfiguration 
---

在上一篇中讲到了使用@Transactional 注解事务运行效果，以及各种嵌套情况下的不同现象，本文就试着去研究一下背后的原理

上一篇讲到为了开启声明式事务功能，需要在配置类上添加@EnableTransactionManagement 注解，是不是有点熟悉，开启AOP 功能的时候，是不是要@@EnableAspectJAutoProxy，都是@Enable...

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {

    //...

}
```

显然这个注解为容器注入了TransactionManagementConfigurationSelector 这个Bean

```java
public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

    // 重写这个方法，用于表示具体注入什么组件
    @Override
    protected String[] selectImports(AdviceMode adviceMode) {
        switch (adviceMode) {
            case PROXY:
                // 如果是PROXY（默认值），则导入 AutoProxyRegistrar、 ProxyTransactionManagementConfiguration
                return new String[] {AutoProxyRegistrar.class.getName(),
                        ProxyTransactionManagementConfiguration.class.getName()};
            case ASPECTJ:
                // 如果是ASPECTJ，则导入 determineTransactionAspectClass() 方法返回的Bean
                return new String[] {determineTransactionAspectClass()};
            default:
                return null;
        }
    }

    private String determineTransactionAspectClass() {
        return (ClassUtils.isPresent("javax.transaction.Transactional", getClass().getClassLoader()) ?
                TransactionManagementConfigUtils.JTA_TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME :
                TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME);
    }

}
```

...