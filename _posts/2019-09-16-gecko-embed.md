---
layout: post
title: 等待通知机制之wait()/notify()
categories: Gecko
description: jdk中Object类中notify/wait
keywords: wait
---

# 等待通知机制之wait()/notify()

## 1.wait()/notify官方定义

![1568276705093](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1568276705093.png)

wait()/notify是Object类中方法，且定义为final级别(不可修改).

jdk8中对wait()的注释为

```java
/ ** *使当前线程等待，直到另一个线程为此对象调用* {@link java.lang.Object＃notify（）}方法或* {@link java.lang.Object＃notifyAll（）}方法。 *换句话说，此方法的行为就像它只是*执行调用{@code wait（0）}一样。 * <p> *当前线程必须拥有此对象的监视器。线程*释放此监视器的所有权并等待，直到另一个线程*通过调用{@code notify}方法或* {@code notifyAll}方法通知等待此对象监视器的线程唤醒*。然后线程等待，直到它可以*重新获得监视器的所有权并继续执行。 * <p> *与在一个参数版本中一样，中断和虚假唤醒是*可能的，并且此方法应始终在循环中使用：* <pre> * synchronized（obj）{* while（<condition not hold> ）* obj.wait（）; * ... //执行适合条件的操作*} * </ pre> *此方法只能由作为此对象监视器的所有者*的线程调用。有关线程可以成为监视器所有者的方式的*描述，请参阅{@code notify}方法。 * * @throws IllegalMonitorStateException如果当前线程不是对象监视器的所有者。 * @throws InterruptedException如果任何线程在当前线程*等待通知之前或当前线程中断*当前线程。当抛出此异常时，将清除当前线程的<i>中断*状态</ i>。 * @see java.lang.Object＃notify（）* @see java.lang.Object＃notifyAll（）* /
```

jdk8中对notify()的注释为

```java
/ ** *唤醒正在等待此对象的*监视器的单个线程。如果任何线程正在等待此对象，则选择其中一个*被唤醒。选择是任意的，由*实施自行决定。线程通过调用{@code wait}方法之一等待对象的*监视器。 * <p> *唤醒的线程将无法继续，直到当前*线程放弃对此对象的锁定。唤醒的线程将以通常的方式与可能*主动竞争同步此对象的任何其他线程竞争;例如，*唤醒线程在*下一个锁定此对象的线程中没有可靠的特权或劣势。 * <p> *此方法只能由作为此对象监视器的所有者*的线程调用。线程以三种方式之一成为*对象监视器的所有者：* <ul> * <li>通过执行该对象的同步实例方法。 * <li>通过执行在对象上同步的{@code synchronized}语句*的主体。 * <li>对于{@code Class，}类型的对象，通过执行该类的* synchronized静态方法。 * </ ul> * <p> *一次只能有一个线程拥有对象的监视器。 * * @throws IllegalMonitorStateException如果当前线程不是此对象监视器的所有者。 * @see java.lang.Object＃notifyAll（）* @see java.lang.Object #wait（）* /
```

jdk8中对notifyAll()的注释为

```java
/ ** *唤醒等待此对象监视器的所有线程。 A *线程通过调用* {@code wait}方法之一等待对象的监视器。 * <p> *唤醒的线程将无法继续，直到当前*线程放弃此对象上的锁定。唤醒的线程*将以通常的方式与可能*主动竞争同步此对象的任何其他线程竞争;例如，*被唤醒的线程在下一个锁定此对象的线程中没有可靠的特权或劣势。 * <p> *此方法只能由作为此对象监视器的所有者*的线程调用。有关线程可以成为监视器所有者的方式的*描述，请参阅{@code notify}方法。 * * @throws IllegalMonitorStateException如果当前线程不是此对象监视器的所有者。 * @see java.lang.Object＃notify（）* @see java.lang.Object #wait（）* /
```



## 2.wait()和sleep()的区别

-wait是object类的方法，sleep是Thread的方法。

-wait方法必须在同步上下文中使用(如synchronized)，会使当前线程释放对象的内置锁并等待，使得当前线程阻塞，直到其他线程唤醒当前线程。sleep方法会使当前线程休眠，是基于Thread的方法，并不会释放调用对象的锁，直到休眠结束重新获得对象锁。

## 3.notify()/notifyAll()的作用

notify是通知一个线程获取对象内置锁，notifyAll是唤醒等待此对象内置锁的所有线程，如果当前线程不是此对象内置锁的所有者，会抛出IllegalMonitorStateException。

被唤醒的线程无法立即执行，必须等待唤醒线程释放对象内置锁。

## 4.等待通知之交叉打印奇偶数

```java
public class TwoThreadWaitNotify extends Object {

    private int start = 1;

    private boolean flag = false;

    public static void main(String[] args) throws Exception {
        TwoThreadWaitNotify number = new TwoThreadWaitNotify();
        Thread t1 = new Thread(new OuNum(number));
        t1.setName("偶数");
        Thread t2 = new Thread(new JiNum(number));
        t2.setName("奇数");

        t1.start();
        t2.start();
    }

    public static class OuNum implements Runnable {
        private TwoThreadWaitNotify number;

        public OuNum(TwoThreadWaitNotify number) {
            this.number = number;
        }

        public void run() {
            while (number.start < 100) {
                synchronized (TwoThreadWaitNotify.class) {
                    System.out.println("偶数线程抢到锁了");
                    if (number.flag) {
                        System.out.println(Thread.currentThread().getName() + "+-+偶数" + number.start);
                        number.start++;

                        number.flag = false;
                        TwoThreadWaitNotify.class.notify();
                    } else {
                        try {
                            System.out.println("偶数线程开始阻塞");
                            TwoThreadWaitNotify.class.wait();
                            System.out.println("偶数线程阻塞结束");
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }

    public static class JiNum implements Runnable {
        private TwoThreadWaitNotify number;

        public JiNum(TwoThreadWaitNotify number) {
            this.number = number;
        }

        public void run() {
            while (number.start < 100) {
                synchronized (TwoThreadWaitNotify.class) {
                    System.out.println("奇数线程抢到锁了");
                    if (!number.flag) {
                        System.out.println(Thread.currentThread().getName() + "+-+奇数" + number.start);
                        number.start++;

                        number.flag = true;
                        TwoThreadWaitNotify.class.notify();
                    } else {
                        try {
                            System.out.println("奇数线程开始阻塞");
                            TwoThreadWaitNotify.class.wait();
                            System.out.println("奇数线程阻塞结束");
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }

}
```

输出结果如下:

![1568598140211](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1568598140211.png)

## 5.小结

notify以及wait方法作为基类object原生方法，和jvm中对象管理机制有一定关系。实例对象内部会含有一个内置锁(在jdk注释处可能描述为monitor)，jvm控制内置锁与线程之前的绑定关系，控制线程的执行与等待。所以，要想深入了解其中的运行原理，还需对jvm有一定的了解与深入。