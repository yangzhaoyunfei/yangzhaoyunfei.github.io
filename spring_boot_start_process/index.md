# Spring Boot启动流程


<!--more-->


## SpringApplication启动流程
以spring boot 2.2.5.RELEASE版本为例, 详细解析启动过程:

启动类, 即主配置类:
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// 1.创建一个SpringApplication对象
// 2.调用了SpringApplication对象的run()
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

### 创建SpringApplication对象

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        // 1.保存主配置类, 它是主要的bean来源, 也就是@SpringBootApplication注解的类
        this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        // 2.推断web应用类型
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
        // 3.从classpath下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；实例化并配置好它们
        setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        // 4.从classpath下找到ETA-INF/spring.factories配置的所有ApplicationListener;实例化并配置好它们
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        // 5.从堆栈信息中推断有main方法的主配置类, 也就是@SpringBootApplication注解的类
        this.mainApplicationClass = deduceMainApplicationClass();
    }
```

### 调用SpringApplication对象的run()方法

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    // 1.从类路径下META-INF/spring.factories获取SpringApplicationRunListeners
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 2.回调所有的获取SpringApplicationRunListener.starting()方法
    listeners.starting();
    try {
        // 3.封装命令行参数
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 4.准备环境，调用prepareEnvironment方法
        //方法中创建环境完成会后回调SpringApplicationRunListener#environmentPrepared()；表示环境准备完成
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        configureIgnoreBeanInfo(environment);
        // 5.打印Banner图（就是启动时的标识图）。
        Banner printedBanner = printBanner(environment);
        // 6.创建ApplicationContext,决定创建web的ioc还是普通的ioc  
        context = createApplicationContext();
        // 7.异常分析报告
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);
        // 8.准备上下文环境，将environment保存到ioc中
        // applyInitializers()：回调之前保存的所有的ApplicationContextInitializer的initialize方法 
        // listeners.contextPrepared(context):回调所有的SpringApplicationRunListener的contextPrepared()，
        // listeners.contextLoaded(context): prepareContext()最后会回调所有的SpringApplicationRunListener的contextLoaded（）
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        // 9.刷新容器,ioc容器初始化（如果是web应用还会创建嵌入式的Tomcat）
        // 扫描，创建，加载所有组件的地方,（配置类，组件，自动配置）
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        // 10.所有的SpringApplicationRunListener回调started方法
        listeners.started(context);
        // 11.从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调，
        // ApplicationRunner先回调，CommandLineRunner再回调
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        // 12.所有的SpringApplicationRunListener回调running方法
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    // 13.整个SpringBoot应用启动完成以后返回启动的ioc容器
    return context;
}
```

### 流程图

![pop](/images/202003/springboot13.svg)

## 自动配置原理

@EnableAutoConfiguration自动配置的魔法骑士就变成了：从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。

然后各种自动配置类依次执行,利用@condition注解,根据容器中的bean,classpath下的class, 配置文件中的属性,来综合决定给功能要不要配置, 如何配置.

我们知道, 自动配置的目的是配置好相应的bean放到容器中, 供我们使用.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}

```

## Reference
* [详解SpringBoot——启动原理及自定义starter](https://zhuanlan.zhihu.com/p/64309456)
* [SpringBoot 启动原理解析](https://zhuanlan.zhihu.com/p/99205565)
* [Spring源码-refresh方法](https://juejin.im/post/5d30410e518825276a6f9a1c)



