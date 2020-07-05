# 获取各种路径


<!--more-->

## 获取spring boot 可执行jar包所在路径
```java
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);

		ApplicationHome home = new ApplicationHome(DemoApplication.class);
		home.getDir();    // returns the folder where the jar is. This is what I wanted.
		home.getSource(); // returns the jar absolute path.
//输出如下：
//C:\Users\foo\IdeaProjects\code-snippet\spring-boot\demo\target
//C:\Users\foo\IdeaProjects\code-snippet\spring-boot\demo\target\demo-0.0.1-SNAPSHOT.jar
	}
}
```

## 未完待续

## References
* [JAVA获取文件路径](https://www.jianshu.com/p/1c9714622a4f)
* [How to get spring boot application jar parent folder path dynamically?](https://stackoverflow.com/questions/46657181/how-to-get-spring-boot-application-jar-parent-folder-path-dynamically)
