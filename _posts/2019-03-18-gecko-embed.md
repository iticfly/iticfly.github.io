---
layout: post
title: 设计模式-代理模式
categories: Gecko
description: 代理模式简述
keywords: winEmbed, Mozilla
---



**代理(Proxy)**是一种设计模式,提供了对目标对象另外的访问方式;即通过代理对象访问目标对象.这样做的好处是:可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能.
这里使用到编程中的一个思想:不要随意去修改别人已经写好的代码或者方法,如果需改修改,可以通过代理的方式来扩展该方法。

**应用场景**：为其他对象提供一种代理以控制对这个对象的访问。从结构上来看和 Decorator 模式类似，
但 Proxy 是控制，更像是一种对功能的限制，而 Decorator 是覆盖或者增加职责。
Spring 的 Proxy 模式在 AOP 中有体现，比如 JdkDynamicAopProxy 和 Cglib2AopProxy。

| 归类       | 特点                                                         | 穷举                                                 |
| ---------- | ------------------------------------------------------------ | :--------------------------------------------------- |
| 结构型模式 | 执行者、代理人<br>对于被代理人来说，这件事一定要做的。但是我自己又不想做或者没时间做<br>对于代理人而言，需要获取到被代理人的资料，只是参与整个过程的某个或几个环节 | 租房中介、售票黄牛、婚介、事务代理、非侵入式日志监听 |

使用代理模式主要有两个目的**:一是保护目标对象，二是增强目标对象。**



#### **静态代理**

举个例子：人到了适婚年龄，父母总是迫不及待希望早点抱孙子。而现在社会的人在各
种压力之下，都选择晚婚晚育。于是着急的父母就开始到处为自己的子女相亲，比子女
自己还着急。这个相亲的过程，就是一种我们人人都有份的代理。来看代码实现：

顶层接口 Person：

```java
public interface Person {
    /**
     * 相亲
     */
    void findLove();
}
```

儿子要找对象,实现Son类:

```java
public class Son implements Person {
    @Override
    public void findLove() {
        System.out.println("儿子要求:肤白貌美大长腿");
    }
}
```

父亲要帮儿子相亲,实现Father(可怜天下老父亲-_-)

```java
public class Father implements Person {
    private Son son;

    public Father(Son son) {
        this.son = son;
    }

    @Override
    public void findLove() {
        System.out.println("父母物色对象");
        son.findLove();
        System.out.println("双方同意交往,确立交往");
    }
}
```

来看测试代码:

```java
    public static void main(String[] args) {
        Father father = new Father(new Son());
        father.findLove();
    }
```

运行结果:

```
父母物色对象
儿子要求:肤白貌美大长腿
双方同意交往,确立交往
```

在上述场景中,父亲作为执行者，儿子作为被代理者，对外是父亲(执行者)帮儿子(代理对象)相亲，保护了儿子(代理对象)**不对外暴露**，同时也对儿子(代理对象)的相亲行为做了前置增强和后置增强(**功能增强**)。

对于分布式业务场景中,我们通常会对数据库进行分库分表，在对数据进行操作之前便可以进行前置设置数据源。



#### 动态代理

动态代理和静态对比基本思路是一致的，只不过动态代理功能更加强大，随着业务的扩
展适应性更强。如果还以找对象为例，使用动态代理相当于是能够适应复杂的业务场景。
不仅仅只是父亲给儿子找对象，如果找对象这项业务发展成了一个产业，进而出现了媒
婆、婚介所等这样的形式。那么，此时用静态代理成本就更大了，需要一个更加通用的
解决方案，要满足任何单身人士找对象的需求。我们升级一下代码，先来看 JDK 实现方
式：

创建媒婆(婚介)JDKMeipo类:

```java
public class JDKMeipo implements InvocationHandler {
    private Person target;

    public Object getInstance(Person target) {
        this.target = target;
        Class<? extends Person> clazz = target.getClass();
        return Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object invoke = method.invoke(this.target, args);
        after();
        return invoke;
    }

    private void before() {
        System.out.println("我是媒婆,我已经拿到你的需求了...");
        System.out.println("开始物色...");
    }

    private void after() {
        System.out.println("如果合适的话,就准备办事...");
    }
}
```



创建客户Customer类:

```java
public class Customer implements Person {
    @Override
    public void findLove() {
        System.out.println("我是客户,我的要求是白富美...");
    }
}
```

测试类:

```java
public class DynamicProxyTest {
    public static void main(String[] args) {
        Person person = (Person) new JDKMeipo().getInstance(new Customer());
        person.findLove();
    }
}
```

测试输出:

```
我是媒婆,我已经拿到你的需求了...
开始物色...
我是客户,我的要求是白富美...
如果合适的话,就准备办事...
```



用户只要是进行相亲行为,都可以在媒婆(JDKMeipo)处创建代理对象,执行者便可以对代理对象的行为进行增强操作。

#### 高仿真JDK Proxy手写实现

既然 jdk 动态代理功能如此强大，那么它如何实现呢？我们来探究一下原理，并模仿  JDK Proxy 手写一个属于自己的动态代理。JDK  Proxy 采用字节重组，重新生成的对象来替换原始的对象以达到动态代理的效果。JDK Proxy生成对象的步骤如下:

1. 拿到被代理对象的引用，并且通过反射的方式获取到它所实现的所有接口。
2. JDK Proxy 重新生成一个新类，同时新类要实现被代理类所实现的所有接口。
3. 动态生成 Java 代码，把新加的业务逻辑方法由一定的逻辑代码去维护。
4. 编译新生成的 Java 代码.class。
5. 再重新加载到 JVM 中运行。

以上这个过程就叫字节码重组。JDK 中有一个规范，在 ClassPath 下只要是以 $  开头的 class 文件一般是自动生成的。 



