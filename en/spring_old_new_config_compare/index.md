# Spring 两种配置文件对比


<!--more-->


## IOC容器配置文件

配置文件是IOC容器的Bean定义来源.

### xml形式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--bean定义-->
</beans>
```

xml形式下, 对注解的支持:
```xml
<context:annotation-config/>
```

### java形式
@Configuration 将一个类标识为JavaConfig形式的Spring Ioc容器的配置类.
```java
@Configuration
public class SpringConfig{
    //bean定义
}
```

然后, 使用配置文件初始化spring IOC 容器:

```java
public class MyApp {
    public static void main(String[] args) {
        // xml文件形式
        ApplicationContext context =
                new ClassPathXmlApplicationContext("spring-config.xml");
        // java代码形式
        ApplicationContext ctx =
                new AnnotationConfigApplicationContext(SpringConfig.class);
        MyBean myBean = (MyBean) context.getBean("myBean");
    }
}
```

## Bean定义

### xml形式

```xml
<bean id="myBean" class="com.company.myapp.MyBean">
    ...
</bean>
```

### java形式
@Bean 常与 @Configuration 搭档使用, 其将一个方法标识为Ioc容器中Bean的定义来源, 方法返回值将作为一个bean定义注册到Spring的IoC容器，方法名将默认成该bean定义的id.
```java
@Configuration
public class SpringConfig{
    @Bean
    public MyBean myBean(){
        return new MyBean();
    }
}
```

## 依赖注入
表达bean与bean之间的依赖关系

### xml形式
```xml
<bean id="myBean" class="com.company.myapp.MyBean">
   <propery name ="dependencyService" ref="myBean2" />
</bean>
<bean id="myBean2" class="com.company.myapp.MyBean2"></bean>
```

### java形式

```java
@Configuration
public class SpringConfig{
    @Bean
    public MyBean myBean(){
        return new MyBean(myBean2());
    }

     @Bean
    public MyBean2 myBean2(){
        return new MyBean2();
    }
}
```

## ComponentScan
自动扫描并加载指定路径下符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean注册到IoC容器中。
可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描.

### xml形式
```xml
<context:component-scan base-package="com.company.myapp`"/>
```

### java形式
```java
@ComponentScan(basePackages = "com.company.myapp")
```

## bean属性值

### xml形式
```xml
<bean class="Person">
<property name ="name" value="i am name"></property>
</bean>
```

### java形式
@Value就相当于传统 xml 配置文件中的 value 字段。
```java
@Component
public class Person {
    @Value("i am name")
    private String name;
}
```

value 的取值可以是：

* 字面量
* 通过 ${key}方式从环境变量中获取值
* 通过 ${key}方式全局配置文件中获取值
* #{SpEL}

## References

* [这样讲 SpringBoot 自动配置原理，你应该能明白了吧](https://zhuanlan.zhihu.com/p/90399818)
