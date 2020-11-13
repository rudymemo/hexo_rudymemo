---
title: spring-boot源码分析
date: 2019-01-11
mp3: /music/本当の优しさとは… - 渡辺俊幸.mp3
cover: /img/wallhaven-294675.jpg
typora-root-url: ..\..\themes\diaspora\source
---

- spring-boot 初始化

  通过调用SpringApplication.run()方法来启动spring-boot

  首先看构造方法

  ```java
  public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
          this.sources = new LinkedHashSet();
          this.bannerMode = Mode.CONSOLE;
          this.logStartupInfo = true;
          this.addCommandLineProperties = true;
          this.addConversionService = true;
          this.headless = true;
          this.registerShutdownHook = true;
          this.additionalProfiles = new HashSet();
          this.isCustomEnvironment = false;
          this.resourceLoader = resourceLoader;
          Assert.notNull(primarySources, "PrimarySources must not be null");
          this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
      	/**
      	* 推断项目类型
      	* 主要有三种类型
      	* NONE:非web项目
      	* SERVlET:标准的servlet web项目
      	* 还有REACTIVE:WEBFLUX等
      	**/
          this.webApplicationType = WebApplicationType.deduceFromClasspath();
         /**
         * 判断完成项目类型
         * 调用getSpringFactoriesInstances
         * 自动配置
         **/
      this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
          this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
          this.mainApplicationClass = this.deduceMainApplicationClass();
      }
  ```

  然后run方法调用SpringApplicationRunListeners的starting()方法利用观察者设计模式，通过各种监听器来完成spring-boot的初始化，比如spring-boot日志初始化等。

- spring-boot 如何内嵌tomcat然后启动的

  首先tomcat是通过JAVA语言开发的，tomcat发行了很多jar包版本，spring-boot通过集成jar包把tomcat依赖到项目中（可通过maven库搜索）。

  启动tomcat，在idea中找到TomcatWebServer类，然后断点调试如下图：

  ![](/img/springboot/tomcat_20190306114739.png)

  可以一直往上找，找到getWebServer方法可以看到tomcat的实例化写法，如下图：

  ![](/img/springboot/tomcat_20190306115059.png)

  可以看到tomcat是通过new Tomcat()来进行实例化的。tomcat得到之后再来看initialize()方法，如下图：

  ![](/img/springboot/tomcat_20190306115400.png)

  然后调用this.tomcat.start()方法来启动tomcat。

- 