# java构造函数


<!--more-->


## 构造方法的作用

我们创建一个类的对象必须要调用构造方法，但构造方法的作用其实并不是为了创建对象，而是为了 **"初始化对象的内部状态"**, 就是为了给对象的各个属性赋初始值。

## 无参构造方法

当且仅当没有显式为类定义构造方法时,编译器会在编译的时候给这个类去自动添加一个无参数的构造方法;
并且其中会隐式的带有一个语句;

## 构造方法重载

一个类可以定义多个构造方法, 完成不同的初始化工作, 所以可以重载.

## 构造方法内调用构造方法

一个构造方法内可以调用另外一个构造方法, 但是使用this关键字来调用, 而不是类名.
```java
public class Test {

    public Test() {
    }

    public Test(int i) {
        //调用另一个构造方法,且必须在第一行
        this();
        i = 3;
    }
}
```

## 带继承关系的构造方法

在调用子类的构造方法时, jvm默认会先调用父类的构造方法.

```java
public class Test {
    public static void main(String[] args) {
        new Son();
    }

    static class Father {
        public Father() {
            System.out.println("Father");
        }
    }

    static class Son extends Father{
        public Son() {
            System.out.println("Son");
        }
    }
}
```

输出如下:

```text
Father
Son
```

 即使程序员没有显式的这样编码, 编译器也会把调用父类构造方法的语句添加到子类的构造方法中. 所以编译过后, 应该时这样的.

 ```java
class Son extends Father {
    public Son() {
        super();//如果没有显式添加, 则编译器会添加, 且必须在第一行.
        System.out.println("Son");
    }
}
 ```

### 设计原因
无论程序员是否愿意，子类在它的构造方法当中必须要调用父类的构造方法, 这个设计与面向对象理念有关.

一般来讲，子类都会比父类拥有更多的属性, 且一部分属性是从父类那里继承过来的. 那么在创建一个子类对象之后, 可以调用父类的构造方法对继承来的属性进行初始化, 自有的属性可以在子类自身构造方法中初始化.

```java
public class Test {
    public static void main(String[] args) {
        new Son("tony", 18);
    }

    static class Father {
        public String name;

        public Father(String name) {
            this.name = name;
        }
    }

    static class Son extends Father {

        public Integer age;

        public Son(String name, Integer age) {
            // 调用父类构造方法,初始化继承来的属性
            super(name);
            // 再初始化子类自身的属性
            this.age = age;
        }
    }
}
```

## References

* [如何理解java的构造方法? - 穆哥学堂的回答 - 知乎](https://www.zhihu.com/question/358104936/answer/940588216)
