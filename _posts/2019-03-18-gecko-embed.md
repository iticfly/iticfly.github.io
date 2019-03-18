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

```java
父母物色对象
儿子要求:肤白貌美大长腿
双方同意交往,确立交往

```

在上述场景中,父亲作为执行者，儿子作为被代理者，对外是父亲(执行者)帮儿子(代理对象)相亲，保护了儿子(代理对象)**不对外暴露**，同时也对儿子(代理对象)的相亲行为做了前置增强和后置增强(**功能增强**)。

对于分布式业务场景中,我们通常会对数据库进行分库分表，在对数据进行操作之前便可以进行前置设置数据源。



