---
layout: post
title: Spring浅谈：声明式事务原理
categories: java之web开发 java之web框架 数据库之mysql
tags: java spring 切面 面向切面编程 AOP AspectJ 面向切面 编译 装饰器设计模式 声明式事务 IoC IoC容器 spring-context spring-aspects pom Maven 增强器 通知方法 代理对象 Cglib代理 事务 数据库 声明式事务 MySQL MyBatis 嵌套事务 @Transactional @EnableTransactionManagement TransactionManagementConfigurationSelector AutoProxyRegistrar ProxyTransactionManagementConfiguration 方法拦截器 
---

在上一篇中讲到了使用@Transactional 注解事务运行效果，以及各种嵌套情况下的不同现象，本文就试着去研究一下背后的原理

上一篇讲到为了开启声明式事务功能，需要在配置类上添加@EnableTransactionManagement 注解，是不是有点熟悉，开启AOP 功能的时候，是不是要先@EnableAspectJAutoProxy，都是@Enable...

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {

    //...

}
```

显然这个注解为容器注入了TransactionManagementConfigurationSelector 这个Bean。继续看后者的实现，其为容器中注入AutoProxyRegistrar、ProxyTransactionManagementConfiguration 组件

这只暂时只考虑AdviceMode 为PROXY 的情况（因为默认就是PROXY）

```java
public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

    /**
     * 重写这个方法，用于表示具体注入什么组件
     */
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

AutoProxyRegistrar 为IoC 容器注入InfrastructureAdvisorAutoProxyCreator，后者是一个PostProcessor，所以这个注入的动作主要是利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用。和AOP 的基本原理一致！

ProxyTransactionManagementConfiguration 会给容器中注入事务增强器，这个是声明式事务的关键所在！

## ProxyTransactionManagementConfiguration组件

```java
@Configuration
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {

    @Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
    @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
    public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
        BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
        advisor.setTransactionAttributeSource(transactionAttributeSource());
        advisor.setAdvice(transactionInterceptor());
        if (this.enableTx != null) {
            advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
        }
        return advisor;
    }

    @Bean
    @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
    public TransactionAttributeSource transactionAttributeSource() {
        return new AnnotationTransactionAttributeSource();
    }

    @Bean
    @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
    public TransactionInterceptor transactionInterceptor() {
        TransactionInterceptor interceptor = new TransactionInterceptor();
        interceptor.setTransactionAttributeSource(transactionAttributeSource());
        if (this.txManager != null) {
            interceptor.setTransactionManager(this.txManager);
        }
        return interceptor;
    }
}
```

很明显ProxyTransactionManagementConfiguration 被标注为一个配置类，实现了为IoC 容器注入事务增强器BeanFactoryTransactionAttributeSourceAdvisor

显然在注入事务增强器的代码中，为事务增强器设置了事务属性、Advice

```java
@Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
    BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
    advisor.setTransactionAttributeSource(transactionAttributeSource());
    advisor.setAdvice(transactionInterceptor());
    if (this.enableTx != null) {
        advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
    }
    return advisor;
}
```

看一下事务属性的注入函数：`advisor.setTransactionAttributeSource(transactionAttributeSource());`

```java
@Bean
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public TransactionAttributeSource transactionAttributeSource() {
    return new AnnotationTransactionAttributeSource();
}
```

显然这个是从IoC 容器中获取一个AnnotationTransactionAttributeSource 组件，当然接下来就是看这个组件了

## AnnotationTransactionAttributeSource的实现

```java
public AnnotationTransactionAttributeSource(boolean publicMethodsOnly) {
    this.publicMethodsOnly = publicMethodsOnly;
    if (jta12Present || ejb3Present) {
        this.annotationParsers = new LinkedHashSet<>(4);
        this.annotationParsers.add(new SpringTransactionAnnotationParser());
        if (jta12Present) {
            this.annotationParsers.add(new JtaTransactionAnnotationParser());
        }
        if (ejb3Present) {
            this.annotationParsers.add(new Ejb3TransactionAnnotationParser());
        }
    }
    else {
        this.annotationParsers = Collections.singleton(new SpringTransactionAnnotationParser());
    }
}
```

这个组件实现各种注解解析器，包括SpringTransactionAnnotationParser、JtaTransactionAnnotationParser、Ejb3TransactionAnnotationParser

比如SpringTransactionAnnotationParser 的解析属性的方法如下，用于解析@Transactional 注解中每个属性，比如propagation、isolation、timeout……

```java
protected TransactionAttribute parseTransactionAnnotation(AnnotationAttributes attributes) {
    RuleBasedTransactionAttribute rbta = new RuleBasedTransactionAttribute();

    Propagation propagation = attributes.getEnum("propagation");
    rbta.setPropagationBehavior(propagation.value());
    Isolation isolation = attributes.getEnum("isolation");
    rbta.setIsolationLevel(isolation.value());
    rbta.setTimeout(attributes.getNumber("timeout").intValue());
    rbta.setReadOnly(attributes.getBoolean("readOnly"));
    rbta.setQualifier(attributes.getString("value"));

    List<RollbackRuleAttribute> rollbackRules = new ArrayList<>();
    for (Class<?> rbRule : attributes.getClassArray("rollbackFor")) {
        rollbackRules.add(new RollbackRuleAttribute(rbRule));
    }
    for (String rbRule : attributes.getStringArray("rollbackForClassName")) {
        rollbackRules.add(new RollbackRuleAttribute(rbRule));
    }
    for (Class<?> rbRule : attributes.getClassArray("noRollbackFor")) {
        rollbackRules.add(new NoRollbackRuleAttribute(rbRule));
    }
    for (String rbRule : attributes.getStringArray("noRollbackForClassName")) {
        rollbackRules.add(new NoRollbackRuleAttribute(rbRule));
    }
    rbta.setRollbackRules(rollbackRules);

    return rbta;
}
```

## TransactionInterceptor事务增强器

ProxyTransactionManagementConfiguration 在注入BeanFactoryTransactionAttributeSourceAdvisor 的同时，还为其设置了事务增强器：`advisor.setAdvice(transactionInterceptor());`

```java
@Configuration
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {

    @Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
    @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
    public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
        BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
        advisor.setTransactionAttributeSource(transactionAttributeSource());
        advisor.setAdvice(transactionInterceptor());
        if (this.enableTx != null) {
            advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
        }
        return advisor;
    }

}
```

对应的代码实现如下所示

```java
@Bean
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public TransactionInterceptor transactionInterceptor() {
    TransactionInterceptor interceptor = new TransactionInterceptor();
    interceptor.setTransactionAttributeSource(transactionAttributeSource());
    if (this.txManager != null) {
        interceptor.setTransactionManager(this.txManager);
    }
    return interceptor;
}
```

按照惯例，当然就是去看一下TransactionInterceptor 事务增强器的实现了，可以看到它继承自TransactionAspectSupport，并实现了MethodInterceptor、Serializable

```java
public class TransactionInterceptor 
    extends TransactionAspectSupport 
    implements MethodInterceptor, Serializable 
```

实现了MethodInterceptor 接口，说明它是一个方法拦截器。上面针对AutoProxyRegistrar 的分析中，知道在声明式事务中，其实是在IoC 容器中放一个代理对象，当代理对象要执行目标方法的时候，方法拦截器就会工作

其工作方法就是invoke()，源码如下所示

```java
@Override
@Nullable
public Object invoke(MethodInvocation invocation) throws Throwable {
    // Work out the target class: may be {@code null}.
    // The TransactionAttributeSource should be passed the target class
    // as well as the method, which may be from an interface.
    Class<？> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

    // Adapt to TransactionAspectSupport's invokeWithinTransaction...
    return invokeWithinTransaction(invocation.getMethod(), targetClass, invocation::proceed);
}
```

其内部其实是调用了invokeWithinTransaction() 方法，该方法先获取事务的属性，再获取PlatformTransactionManager（回到[上文的例子](http://www.xumenger.com/spring-transactional-20200618/)中，特地说明了要往IoC 容器中加入一个PlatformTransactionManager）

关于事务开启、回滚、提交的逻辑就是在这个方法中实现的！详细看我在下面代码中的注释！

```java
@Nullable
protected Object invokeWithinTransaction(Method method, @Nullable Class<？> targetClass,
        final InvocationCallback invocation) throws Throwable {

    // If the transaction attribute is null, the method is non-transactional.
    TransactionAttributeSource tas = getTransactionAttributeSource();
    final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);
    final PlatformTransactionManager tm = determineTransactionManager(txAttr);
    final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);

    if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {

        /**
         * 如果有必要的话开启事务
         */
        // Standard transaction demarcation with getTransaction and commit/rollback calls.
        TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);

        Object retVal;
        try {

            /**
             * 执行目标方法
             */
            // This is an around advice: Invoke the next interceptor in the chain.
            // This will normally result in a target object being invoked.
            retVal = invocation.proceedWithInvocation();
        }
        catch (Throwable ex) {

            /**
             * 如果出现异常，则获取事务管理器，并利用事务管理器回滚操作！
             */
            // target invocation exception
            completeTransactionAfterThrowing(txInfo, ex);
            throw ex;
        }
        finally {
            cleanupTransactionInfo(txInfo);
        }

        /**
         * 如果一切正常，则提交事务！
         */
        commitTransactionAfterReturning(txInfo);
        return retVal;
    }

    else {
        final ThrowableHolder throwableHolder = new ThrowableHolder();

        // It's a CallbackPreferringPlatformTransactionManager: pass a TransactionCallback in.
        try {
            Object result = ((CallbackPreferringPlatformTransactionManager) tm).execute(txAttr, status -> {
                TransactionInfo txInfo = prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
                try {
                    return invocation.proceedWithInvocation();
                }
                catch (Throwable ex) {
                    if (txAttr.rollbackOn(ex)) {
                        // A RuntimeException: will lead to a rollback.
                        if (ex instanceof RuntimeException) {
                            throw (RuntimeException) ex;
                        }
                        else {
                            throw new ThrowableHolderException(ex);
                        }
                    }
                    else {
                        // A normal return value: will lead to a commit.
                        throwableHolder.throwable = ex;
                        return null;
                    }
                }
                finally {
                    cleanupTransactionInfo(txInfo);
                }
            });

            // Check result state: It might indicate a Throwable to rethrow.
            if (throwableHolder.throwable != null) {
                throw throwableHolder.throwable;
            }
            return result;
        }
        catch (ThrowableHolderException ex) {
            throw ex.getCause();
        }
        catch (TransactionSystemException ex2) {
            if (throwableHolder.throwable != null) {
                logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
                ex2.initApplicationException(throwableHolder.throwable);
            }
            throw ex2;
        }
        catch (Throwable ex2) {
            if (throwableHolder.throwable != null) {
                logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
            }
            throw ex2;
        }
    }
}
```

>同样，本文也只是粗略的过一遍，详细的还得是自己结合具体的各种案例，详细的进行Debug 和分析

>同样的，理解Java 的反射机制，对于理解这部分的代码也是极其关键的！！！
