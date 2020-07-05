# filter interceptor listenter


<!--more-->


## 前言

这几个概念其实都是设计模式你的概念, 其命名都跟真实场景密切相关, 理解了命名, 就了解了模式.

## Filter

当你有一堆东西的时候，你只希望选择符合某些要求的东西。定义这些要求的工具，就是过滤器。

应用场景:

* 过滤掉非法url请求（不是login.do的地址请求，如果用户没有登陆都过滤掉）,
* 在传入servlet或者 struts的action前统一设置字符集，
* 去除掉一些非法字符（聊天室经常用到的，一些骂人的话）。。。

```java
public class Test {
    public static void main(String[] args) {
        filter();
    }

    public static void filter() {
        // 填充100个带有随机字母标签的球
        List<String> array = new ArrayList<>();
        Random r = new Random();
        String[] balls = new String[]{"A", "B", "C"};
        for (int i = 0; i < 100; i++) {
            array.add(balls[r.nextInt(3)]);
        }

        System.out.println(array);
        // 只拿出B的来。不明白的自行学习Java 8
        array = array.stream().filter("B"::equals).collect(Collectors.toList());
        System.out.println(array);
    }
```

## Interceptor

在一个流程正在进行的时候，你希望干预它的进展，甚至终止它进行，这是拦截器做的事情。

应用场景:

进行权限验证，或者是来判断用户是否登陆，日志记录，或者限制时间点访问。
我自己用过拦截器，是用户每次登录时，都能记录一个登录时间。 （这点用拦截器明显合适，用过滤器明显不合适，因为没有过滤任何东西）

```java
interface Interceptor {
    void intercept(River river);
}

public class Test {
    public static void main(String[] args) {
        RiverController rc = new RiverController();
        Interceptor inter = new SomeInterceptor();

        // 这一步通常叫做控制反转或依赖注入，其实也没啥子
        rc.setInterceptor(inter);

        rc.flow(new River());
    }
}

class River {
    // 流量
    int volume;
    // 总鱼数
    int numFish;

    @Override
    public String toString() {
        return "River{" +
                "volume=" + volume +
                ", numFish=" + numFish +
                '}';
    }
}

class PowerGenerator {
    double generate(int volume) {
        // 假设每一百立方米水发一度电
        return volume / 100d;
    }
}

class SomeInterceptor implements Interceptor {
    PowerGenerator generator = new PowerGenerator();

    @Override
    public void intercept(River river) {
        // 消耗了1000立方米水来发电
        int waterUsed = 1000;

        // 水量减少了1000。
        river.volume -= waterUsed;

        // 发电
        generator.generate(waterUsed);

        // 拦截所有的鱼
        river.numFish = 0;

        System.out.println("passed interceptor");
    }
}

class RiverController {
    Interceptor interceptor;

    void setInterceptor(Interceptor interceptor) {
        this.interceptor = interceptor;
    }

    void flow(River river) {
        // 大坝前, 源头积累下来的水量和鱼
        river.volume += 100000;
        river.numFish += 1000;

        System.out.println(river);
        // 经过了大坝
        interceptor.intercept(river);

        System.out.println(river);

        // 下了点雨
        river.volume += 1000;
    }
}
```

## Listener

当一个事件发生的时候，你希望获得这个事件发生的详细信息，而并不想干预这个事件本身的进程，这就要用到监听器。

应用场景:

当你要触发一个事件，但这件事既不是过滤，又不是拦截，那很可能就是监听！ 联想到Windows编程里的，单击鼠标、改变窗口尺寸、按下键盘上的一个键都会使Windows发送一个消息给应用程序。监听器的概念类似于这些。

```java
// 监听器
interface BedListener {
    // 监听器在参数中收到了某个事件，而这个事件往往是只读的
    // 监听器的方法名通常以"on"开头
    void onBedSound(String sound);
}

public class Test {

    public static void main(String args[]) {
        Neighbor n = new Neighbor();
        n.setListener(sound -> generatePower());
        n.doInterestingStuff();
    }

    private static void generatePower() {
        // 根据当地法律法规，部分内容无法显示
    }
}

// 邻居
class Neighbor {
    BedListener listener;

    // 依然是所谓控制反转
    void setListener(BedListener listener) {
        this.listener = listener;
    }

    void doInterestingStuff() {
        // 根据当地法律法规，部分内容无法显示

        // 将事件发送给监听器
        listener.onBedSound("嘿咻");
        listener.onBedSound("oyeah");
    }
}

```

## Summary

约定：

* 过滤器：用于属性甄别，对象收集（不可改变过滤对象的属性和行为）
* 监听器：用于对象监听，行为记录（不可改变监听对象的属性和行为）
* 拦截器：用于对象拦截，行为干预（可以改变拦截对象的属性和行为）

能力逐渐增强：过滤器-->监听器-->拦截器

三者的用法完全不同，不存在什么交集。你也可以用拦截器做过滤器的工作，但是大材小用了，也不符合约定。

## References

* [请教一下关于过滤器，拦截器，监听器具体应用上的区别？ - Kangol LI的回答 - 知乎](https://www.zhihu.com/question/35225845/answer/61876681)
* [请教一下关于过滤器，拦截器，监听器具体应用上的区别？ - 支浩宇的回答 - 知乎](https://www.zhihu.com/question/35225845/answer/100703033)
* [请教一下关于过滤器，拦截器，监听器具体应用上的区别？ - 至尊七虾堡的回答 - 知乎](https://www.zhihu.com/question/35225845/answer/345131918)


