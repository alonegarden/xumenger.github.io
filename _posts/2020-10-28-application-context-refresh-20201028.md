---
layout: post
title: ApplicationContext 的初始化过程
categories: java之web开发 java之web框架 Spring之IoC 
tags: java bean IoC 循环依赖 Spring 容器 constructor-arg 设计模式 模板模式 BeanDefinition ApplicationContext getBean() 依赖注入 
---

一直提到，在Spring 的ApplicationContext 中，refresh() 方法是至关重要的，理解Spring 容器的初始化过程，这个函数是毋庸置疑的切入点！

先看一下ApplicationContext 的继承树

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
        MessageSource, ApplicationEventPublisher, ResourcePatternResolver 

public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable

public abstract class AbstractApplicationContext extends DefaultResourceLoader
        implements ConfigurableApplicationContext, DisposableBean 
```

其中refresh() 方法就是在AbstractApplicationContext 中定义的

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        // 这里是在子类中启动refreshBeanFactory() 的地方，refreshBeanFactory() 调用loadBeanDefinitions()
        // loadBeanDefinitions() 完成<bean> 解析为BeanDefinition 的过程
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            // 设置BeanFactory 的后置处理
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            // 调用BeanFactory 的后置处理器，这些后置处理器是在Bean 定义中向容器中注册的
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            // 注册Bean 的后处理器，在Bean 创建过程中调用
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            // 对上下文中的消息源进行初始化
            initMessageSource();

            // Initialize event multicaster for this context.
            // 初始化上下文中的事件机制
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            // 初始化其他的特殊Bean
            onRefresh();

            // Check for listener beans and register them.
            // 检查监听Bean 并且将这些Bean 向容器注册
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            // 实例化所有的non-laze-init 的单例Bean，具体通过调用getBean() 方法
            // getBean() 方法也是理解Spring 框架的重中之重
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            // 发布容器事件，结束Refesh 过程
            finishRefresh();
        }

        catch (BeansException ex) {
            // Destroy already created singletons to avoid dangling resources.
            // 为防止Bean 资源占用，在异常处理中，销毁已经在前面过程中生成的单例Bean
            destroyBeans();

            // Reset 'active' flag.
            // 重置active 标志
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }
    }
}
```

@Autowired、AOP、声明式事务、集成MyBatis、SpringBoot集成Tomcat、SpringMVC等功能都分别是在哪个环节实现的？

在refresh() 中，obtainFreshBeanFactory() 中调用loadBeanDefinitions() 解析得到BeanDefinition，在finishBeanFactoryInitialization() 中通过调用getBean() 实现对于Bean 的依赖注入

那么在obtainFreshBeanFactory() 和finishBeanFactoryInitialization() 之间的一系列方法调用都做了什么？

* prepareBeanFactory()
* invokeBeanFactoryPostProcessors()
* registerBeanPostProcessors()
* initMessageSource()
* initApplicationEventMulticaster()
* onRefresh()
* registerListeners()
