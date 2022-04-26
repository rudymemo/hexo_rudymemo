---
title: spring-aop原理
date: 2019-03-06
mp3: /music/Love Theme - Henry Mancini.mp3
cover: 'https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn@main/20220424/YeMo1qPZhxwL9nf.2bgxv6y6hn40.jpg'
typora-root-url: ..\..\themes\diaspora\source
---

- OOP——AOP 

  OOP（Object Oriented Programming，面向对象编程）核心思想是将客观存在的不同事物抽象成相互独立的类，然后把与事物相关的属性和行为封装到类里，并通过继承和多态来定义类彼此间的关系，最后通过操作类的实例来完成实际业务逻辑的功能需求。处理逻辑是自上而下，产生一些横切性的问题，比如说权限验证、日志的记录、事务的开启与关闭等等，这样会产生了很多散落性的代码，由此衍生了AOP。

- AOP（Aspect Oriented Programming，面向切面编程）核心思想是将业务逻辑中与类不相关的通用功能切面式的提取分离出来，让多个类共享一个行为，一旦这个行为发生改变，不必修改类，而只需要修改这个行为即可。

  AOP相较于OOP来说是一个全新的编程思想，可以解决OOP产生的横切性的问题，解决代码散落的问题。在spring中	AOP实现主要有两种方式：

  1. 一种是通过JDK动态代理。JDK动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。
  2. 一种是CGLIB动态代理。CGLIB动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

  spring使用哪种代理方式？

  - 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP 
  - 如果目标对象实现了接口，可以强制使用CGLIB实现AOP 

  - 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

- AOP demo

  下面手写一下AOP的实现，首先是配置config文件，[看官网文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop)

  添加配置文件：

  To enable @AspectJ support with Java `@Configuration`, add the `@EnableAspectJAutoProxy` annotation

  ```java
  package com.config;
  
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.EnableAspectJAutoProxy;
  
  @Configuration
  @ComponentScan("com")
  @EnableAspectJAutoProxy(proxyTargetClass = true)
  /**
   *  我的目标对象是实现了接口的
   *  proxyTargetClass默认为false(jdk动态代理)
   *  为true时强制使用cglib代理
   */
  public class AppConfig {
  }
  ```

  接口类：

  ```java
  package com.dao;
  
  public interface IndexDao {
      public String query(String name);
  }
  ```

  实现类：

  ```java
  package com.dao;
  
  import org.springframework.stereotype.Component;
  
  @Component("dao")
  public class IndexDaoImpl {
      public String query(String name) {
          return "return：" + name;
      }
  }
  ```

  切面类：

  ```java
  package com.config;
  
  import org.aspectj.lang.ProceedingJoinPoint;
  import org.aspectj.lang.annotation.*;
  import org.springframework.stereotype.Component;
  
  @Component
  @Aspect
  public class RudyAspect {
  
      @Pointcut("execution(* com.dao.*.*(..))")
      public void pointCut() {
  
      }
      @Before("pointCut()")
      public void before() {
          System.out.println("before--------");
      }
      @Around("pointCut()")
      public void around(ProceedingJoinPoint pjp) throws Throwable {
          String className = pjp.getTarget().getClass().getName();
          String methodName = pjp.getSignature().getName();
          Object[] args = pjp.getArgs();
          System.out.println(className);
          System.out.println(methodName);
          for (Object arg : args) {
              System.out.println("获取参数：" + arg);
          }
          Object result = pjp.proceed(args);
          System.out.println(result);
      }
      @After("pointCut()")
      public void after() {
          System.out.println("after--------");
      }
  }
  ```

  测试结果如下图：

  ![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261110868.png)

  可以看到横切的结果，已经实现。

- 下面介绍下各种注解的意思

  @Pointcut		顾名思义切入点，可配置多个  

  @Before			执行切入点之前执行某个操作  

  @After			执行切入点之后不管是抛出异常或者正常退出都会执行  

  @AfterReturning	后置增强，方法正常退出时执行  

  @AfterThrowing	异常抛出增强  

  @Around  环绕增强

  这些注解可以自行实验，就不在此贴代码了。

  @Pointcut()参数有必要解释一下：

  - execution：用于匹配方法执行的连接点；
  - within：用于匹配指定类型内的方法执行；
  - this：用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；        
  - target：用于匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；
  - args：用于匹配当前执行的方法传入的参数为指定类型的执行方法；
  - @within：用于匹配所以持有指定注解类型内的方法；
  - @target：用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；
  - @args：用于匹配当前执行的方法传入的参数持有指定注解的执行；
  - @annotation：用于匹配当前执行方法持有指定注解的方法；

- 应用场景：

  - 比如redis缓冲
  - 自定义日志记录
  - 等等

