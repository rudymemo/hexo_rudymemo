---
title: spring-ioc原理
date: 2019-03-07
mp3: /music/낡은 추억의 상자를 열었습니다 - Piano i.mp3
cover: 'https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn@main/20220424/HX2ZuOofYNwq31W.2h83x9ihsdk0.jpg'
typora-root-url: ..\..\themes\diaspora\source
---

- 最初java调用

  ```java
  package com.test;
  
  public class UserService {
      public void add(){};
  }
  ```

  要调用UserService类中的add()方法：

  ```java
  package com.test;
  
  public class UserServlet {
      public void addUser() {
          UserService user = new UserService();
          user.add();
      }
  }
  ```

  其他类用到，也是同样的写法，这是最初java调用某个类的方法的写法，如果类的方法或者类名要改定，那么必须把调用到改类的地方全部都改写一遍，这样耦合度太高，于是使用工厂模式来进行解耦合，我们可以这样写：

  ```java
  package com.test;
  
  public class UserFactory {
      public static UserService getService() {
          return new UserService();
      }
  }
  ```

  在UserServlet中调用工厂的静态类：

  ```java
  package com.test;
  
  public class UserServlet {
      public void addUser() {
          UserService user = UserFactory.getService();
          user.add();
      }
  }
  ```

  这样在servlet与factory也存在耦合度。

- IOC配置文件的实现xml

  ```xml
  <!-- xml文件中配置bean -->
  <bean id="userService" class="test.UserService"/>
  ```

  配置完成bean之后：

  ```java
  package com.test;
  
  public class UserFactory {
      public static UserService getService() throws ClassNotFoundException, IllegalAccessException, InstantiationException {
          // 利用dom4j解析xml可以获得id与class
          String className = "读取到的class的值";
          // 利用反射创建对象
          Class clazz = Class.forName(className);
          UserService userService = (UserService) clazz.newInstance();
          return userService;
      }
  }
  ```

  但是这样也不算是彻底解耦合，xml文件也需要不断改动，于是注解形式就出现了。

- IOC注解配置

  比如我们有一个类：

  ```java
  package com.rudy.bean;
  
  public class Color {
  
  }
  ```

  想把他加入到bean中可以这样配置：

  ```java
  package com.rudy.conf;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Import;
  
  import com.rudy.bean.Color;
  
  @Import(Color.class)
  public class AppConfig {
  	/*@Bean("Color")
      public Color color() {
          
          return new Color();
      }*/
  }
  ```

  去掉@Import(Color.class)把下面注释打开也是一样，测试一下：

  ```java
  package com.rudy.test;
  
  import org.junit.Test;
  import org.springframework.context.annotation.AnnotationConfigApplicationContext;
  
  import com.rudy.conf.AppConfig;
  
  public class IOCTest {
      AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig2.class);
      @Test
      public void test() {
          AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
          String[] beanDefinitionNames = annotationConfigApplicationContext.getBeanDefinitionNames();
          for (String name : beanDefinitionNames) {
              System.out.println(name);
          }
      }
  }
  ```

  输出结果：

  ```
  org.springframework.context.annotation.internalConfigurationAnnotationProcessor
  org.springframework.context.annotation.internalAutowiredAnnotationProcessor
  org.springframework.context.annotation.internalRequiredAnnotationProcessor
  org.springframework.context.annotation.internalCommonAnnotationProcessor
  org.springframework.context.event.internalEventListenerProcessor
  org.springframework.context.event.internalEventListenerFactory
  appConfig
  com.rudy.bean.Color
  ```

  也可以根据条件是否加载bean：

  ```java
  package com.rudy.conf;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Conditional;
  
  import com.rudy.bean.Color;
  import com.rudy.condition.MyColor;
  
  public class AppConfig {
      
      /**
       * 根据条件自定义加载bean
       * @return
       */
      @Conditional(MyColor.class)
      @Bean
      public Color color() {
          
          return new Color();
      }
  
  }
  ```

  ```java
  package com.rudy.condition;
  
  import org.springframework.context.annotation.Condition;
  import org.springframework.context.annotation.ConditionContext;
  import org.springframework.core.type.AnnotatedTypeMetadata;
  
  public class MyColor implements Condition{
  
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          
          return false;
      }
  
  }
  ```

  查看测试运行结果：

  ```
  org.springframework.context.annotation.internalConfigurationAnnotationProcessor
  org.springframework.context.annotation.internalAutowiredAnnotationProcessor
  org.springframework.context.annotation.internalRequiredAnnotationProcessor
  org.springframework.context.annotation.internalCommonAnnotationProcessor
  org.springframework.context.event.internalEventListenerProcessor
  org.springframework.context.event.internalEventListenerFactory
  appConfig
  ```

  可以发现color没有被放入bean，因为我们自定义的condition返回了false。

  condition中可以获取很多信息来进行判断：

  ```java
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
      //1、获取IOC的beanFactory
      ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
      //2、得到类加载器
      ClassLoader classLoader = context.getClassLoader();
      //3、获取bean定义的注册类
      BeanDefinitionRegistry registry = context.getRegistry();
      //4、获取运行的环境
      Environment environment = context.getEnvironment();
      String property = environment.getProperty("os.name");
      if (property.contains("linux")) {
          return true;
      }
      return false;
  }
  ```



------



  如果是多个路径多个类加入bean可以利用各种注解加上扫描还有自定义规则等等。

  比如@Controller，@Service等

  ```java
  package com.rudy.controller;
  
  import org.springframework.stereotype.Controller;
  
  @Controller
  public class BookController {
  
  }
  ```

  ```java
  package com.rudy.service;
  
  import org.springframework.stereotype.Service;
  
  @Service
  public class BookService {
  
  }
  ```

  ```java
  package com.rudy.conf;
  
  import org.springframework.context.annotation.ComponentScan;
  
  @ComponentScan(value = "com.rudy")
  public class AppConfig {
  
  }
  ```

  运行测试类结果如下：

  ```
  org.springframework.context.annotation.internalConfigurationAnnotationProcessor
  org.springframework.context.annotation.internalAutowiredAnnotationProcessor
  org.springframework.context.annotation.internalRequiredAnnotationProcessor
  org.springframework.context.annotation.internalCommonAnnotationProcessor
  org.springframework.context.event.internalEventListenerProcessor
  org.springframework.context.event.internalEventListenerFactory
  appConfig
  bookController
  bookService
  ```



  还可以自定义扫描规则

  ```java
  package com.rudy.conf;
  
  import org.springframework.context.annotation.ComponentScan;
  import org.springframework.context.annotation.ComponentScan.Filter;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.FilterType;
  
  import com.rudy.filter.MyTypeFilter;
  
  @Configuration
  @ComponentScan(value = "com.rudy",includeFilters={
          /*@Filter(type=FilterType.ANNOTATION,classes={Controller.class}),*/
          /** Filter candidates using a given custom
           * {@link org.springframework.core.type.filter.TypeFilter} implementation.
           * 自定义过滤规则，实现TypeFilter接口
           */
          @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
  },useDefaultFilters = false)
  public class AppConfig {
  
  }
  ```

  ```java
  package com.rudy.filter;
  
  import java.io.IOException;
  
  import org.springframework.core.io.Resource;
  import org.springframework.core.type.AnnotationMetadata;
  import org.springframework.core.type.ClassMetadata;
  import org.springframework.core.type.classreading.MetadataReader;
  import org.springframework.core.type.classreading.MetadataReaderFactory;
  import org.springframework.core.type.filter.TypeFilter;
  
  public class MyTypeFilter implements TypeFilter{
  
      public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
              throws IOException {
          AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
          String className = annotationMetadata.getClassName();
          ClassMetadata classMetadata = metadataReader.getClassMetadata();
          Resource resource = metadataReader.getResource();
          System.out.println("--->" + className);
          if (className.contains("ller")) {
              return true;
          }
          return false;
      }
  
  }
  ```

  注解的好处可以看到，很多时候我们并不需要知道类的信息，只需要加上注解，然后通过配置文件自定义就可以加载对应的类到bean容器中。

- 最后放一张bean加载解析过程图片



![](https://cdn.jsdelivr.net/gh/rudymemo/picx.xpoet.cn/img/202204261402871.png)