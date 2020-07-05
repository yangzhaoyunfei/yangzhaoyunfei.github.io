# spring 中几个常见注解


<!--more-->
## @ConfigurationProperties 【SpringBoot 提供】

我们需要取 N 个配置项，通过 @Value 的方式一个一个去取很麻烦。我们可以使用 @ConfigurationProperties.

被 @ConfigurationProperties 标识的类的所有属性和配置文件中相关的配置项进行绑定, 前提是被标示类具有setter方法。（默认从全局配置文件中获取配置值），绑定之后我们就可以通过这个类去访问全局配置文件中的属性值了。

如:

配置文件中有如下属性:
```text
person.name=kundy
person.age=13
person.sex=male
```
这里 @ConfigurationProperties 有一个 prefix参数，主要是用来指定该配置项在配置文件中的前缀。

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private Integer age;
    private String sex;

    // getter & setter...

}
```

## @EnableConfigurationProperties
形如 @ConfigurationProperties(prefix = "com.cxytiandi"), 单独使用 @ConfigurationProperties 注解，被注解类不会被注册到ioc容器中，也就无法通过被注解类获取到配置文件中的属性。

还需要使用其他注解将被注解类注册到容器中，如@Componet,@Configuration，@EnableConfigurationProperties(MyConfig.class)等

```java
@ConfigurationProperties(prefix = "com.cxytiandi")
@Configuration
public class MyConfig {
	private String name;
    // getter & setter & constructor
}
```

## @Import 【Spring 提供】

@Import注解支持导入普通 java 类，并将其声明成一个bean。主要用于将多个分散的 java config 配置类融合成一个更大的 config 类

**@Import 三种使用方式**

* 直接导入普通的 Java 类。
* 配合自定义的 ImportSelector 使用。
* 配合 ImportBeanDefinitionRegistrar 使用。

### 导入普通的 Java 类

1. 创建一个普通的 Java 类。
```java
public class Circle {
    public void sayHi() {
        System.out.println("Circle sayHi()");
    }
}
```
2. 创建一个配置类，里面没有显式声明任何的 Bean，然后将刚才创建的 Circle 导入。
```java
@Import({Circle.class})
@Configuration
public class MainConfig {
}
```
3. 测试
```java
public static void main(String[] args) {

ApplicationContext context = new AnnotationConfigApplicationContext(MainConfig.class);
    Circle circle = context.getBean(Circle.class);
    circle.sayHi();
}
```
4. 运行结果
```text
Circle sayHi()
```

可以看到我们顺利的从 IOC 容器中获取到了 Circle 对象，证明我们在配置类中导入的 Circle 类，确实被声明为了一个 Bean。

### 配合自定义的 ImportSelector 使用

ImportSelector是一个接口，该接口中只有一个 selectImports 方法，用于返回完整类名的数组。所以利用该特性我们可以给容器动态导入 N 个Bean。

1. 创建普通 Java 类 Triangle。
```java
public class Triangle {
    public void sayHi(){
        System.out.println("Triangle sayHi()");
    }
}
```
2. 创建 ImportSelector 实现类，selectImports 返回 Triangle 的全类名。
```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{"com.company.Triangle"};
    }
}
```
3. 创建配置类，在原来的基础上还导入了 MyImportSelector。
```java
@Import({Circle.class,MyImportSelector.class})
@Configuration
public class MainConfigTwo {
}
```

4. 创建测试类
```java
public static void main(String[] args) {

ApplicationContext context = new AnnotationConfigApplicationContext(MainConfigTwo.class);
    Circle circle = context.getBean(Circle.class);
    Triangle triangle = context.getBean(Triangle.class);
    circle.sayHi();
    triangle.sayHi();
}
```

### 配合 ImportBeanDefinitionRegistrar 使用

ImportBeanDefinitionRegistrar也是一个接口，它可以手动注册bean到容器中，从而我们可以对类进行个性化的定制。(需要搭配 @Import 与 @Configuration 一起使用。）

1. 创建普通 Java 类 Rectangle。
```java
public class Rectangle {
    public void sayHi() {
        System.out.println("Rectangle sayHi()");
    }
}
```

2. 创建 ImportBeanDefinitionRegistrar 实现类，实现方法直接手动注册一个名叫 rectangle 的 Bean 到 IOC 容器中
```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {

    RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Rectangle.class);
        // 注册一个名字叫做 rectangle 的 bean
        beanDefinitionRegistry.registerBeanDefinition("rectangle", rootBeanDefinition);
    }

}
```

3. 创建配置类，导入 MyImportBeanDefinitionRegistrar 类。
```java
@Import({Circle.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
@Configuration
public class MainConfigThree {
}
```

4. 创建测试类。
```java
public static void main(String[] args) {

    ApplicationContext context = new AnnotationConfigApplicationContext(MainConfigThree.class);
        Circle circle = context.getBean(Circle.class);
        Triangle triangle = context.getBean(Triangle.class);
        Rectangle rectangle = context.getBean(Rectangle.class);
        circle.sayHi();
        triangle.sayHi();
        rectangle.sayHi();

    }
}
```

5. 运行结果
```text
Circle sayHi()
Triangle sayHi()
Rectangle sayHi()
```
嗯对，Rectangle 对象也被注册进来了。

## @Conditional 【Spring提供】

@Conditional 注解可以实现只有在特定条件满足时才启用一些配置。

1. 创建普通 Java 类 ConditionBean，该类主要用来验证 Bean 是否成功加载
```java
public class ConditionBean {
    public void sayHi() {
        System.out.println("ConditionBean sayHi()");
    }
}
```

2. 创建 Condition 实现类，@Conditional 注解只有一个 Condition 类型的参数，Condition 是一个接口，该接口只有一个返回布尔值的 matches() 方法，该方法返回 true 则条件成立，配置类生效。反之，则不生效。在该例子中我们直接返回 true。
```java
public class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        return true;
    }
}
```

3. 创建配置类，可以看到该配置的 @Conditional 传了我们刚才创建的 Condition 实现类进去，用作条件判断。
```java
@Configuration
@Conditional(MyCondition.class)
public class ConditionConfig {
    @Bean
    public ConditionBean conditionBean(){
        return new ConditionBean();
    }
}
```
4. 编写测试方法
```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(ConditionConfig.class);
    ConditionBean conditionBean = context.getBean(ConditionBean.class);
    conditionBean.sayHi();
}
```

5. 结果分析

因为 Condition 的 matches 方法直接返回了 true，配置类会生效，我们可以把 matches 改成返回 false，则配置类就不会生效了。

## @Conditional 扩展注解

除了自定义 Condition，Spring 还为我们扩展了一些常用的 Condition。

* @ConditionalOnBean : 容器中存在指定 Bean，则生效
* @ConditionalOnMissingBean : 容器中不存在指定 Bean，则生效
* @ConditionalOnClass : classpath中有指定的类，则生效
* @ConditionalOnMissingClass : classpath中没有指定的类，则生效
* @ConditionalOnProperty : 系统中指定的属性有指定的值, 则生效
* @ConditionalOnWebApplication : 当前是web环境，则生效

## @Primary 和 @Qualifier

>https://blog.csdn.net/sinat_32023305/article/details/90718687

当一个接口有2个不同实现时,使用@Autowired注解时会报org.springframework.beans.factory.NoUniqueBeanDefinitionException异常信息

* @Qualifier 选择一个对象的名称,通常比较常用
* @Primary 可以理解为默认优先选择,不可以同时设置多个

## @Resource 和 @Autowired 和 @Inject 

### @Resource
@Resource注释是JSR-250注释集合的一部分，位于 javax.annotation.Resource，并打包在Jakarta EE中。该注释有以下执行路径，按优先级列出:
* Match by Name
* Match by Type
* Match by Qualifier

这些执行路径适用于setter和字段注入。

### @Autowired
@Autowired注解的行为与@Inject注解相似。唯一的区别是@Autowired注解是Spring框架的一部分。
此注释与@Inject注释有相同的执行路径，按优先级顺序列出:
* Match by Type
* Match by Qualifier
* Match by Name
这些执行路径适用于setter和字段注入。

### @Inject
@Inject注释属于JSR-330注释集合。该注释有以下执行路径，按优先级列出:
* Match by Type
* Match by Qualifier
* Match by Name
这些执行路径适用于setter和字段注入。为了使用@Inject注释， 必须在Gradle或Maven中添加 javax.inject 相关依赖。

[具体使用场景](https://www.baeldung.com/spring-annotations-resource-inject-autowire#5-discussion-summary)

## References
* [这样讲 SpringBoot 自动配置原理，你应该能明白了吧](https://zhuanlan.zhihu.com/p/90399818)
