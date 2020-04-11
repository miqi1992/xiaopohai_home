### Object 相关概念

Object 是 java 中的顶级父类，它是所有类的超类，所有对象以及数组均会实现这个类提供的方法

JVM 在编译源码过程中，遇到没有继承 Object 的对象时，编译器会指定默认父类 Object

接口没有继承顶级父类，但会隐式的声明一套和 Object 中的方法签名完全一样的方法，这也就符合万物皆对象的面向对象思想，任何对象直接或间接的跟 Object 对象有关

### Object 类源码中的关键方法

```
public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }
    ···
}
```

**registerNatives()：**注册本地方法

它会注册除 `registerNatives`外的所有本地方法。当 java 程序需要调用本地方法时，jvm 会在加载的动态文件里定位并链接该本地方法，从而得以执行此方法。

在类被加载时就调用 `registerNatives()` 的用意是此时是程序主动将本地方法链接到调用方，当 java 程序需要调用本地方法时可直接调用，省去了 jvm 再去定位并链接的这一步，这样做的好处是：

1.  更加方便且提高了执行效率
2.  当本地方法在程序运行中有更新，调用 `registerNatives()` 可及时实现更新
3.  Java 程序需要调用一个本地应用提供的方法时，因为虚拟机只会检索本地动态库，因而虚拟机是无法定位到本地方法实现的，这个时候就只能使用 `registerNatives()` 进行主动链接
4.  通过 `registerNatives()` 在定义本地方法的实现时，可以不遵守 JNI 的命名规范

```
···
    public final native Class<?> getClass();
```

**getClass()：**返回此对象的运行时类

返回值是 Class 类型，通过返回的 Class 对象我们可以获取目标类中包含的所有方法、所有变量、构造函数等

```
···
    public native int hashCode();
```

**hashCode()：**返回此对象的存储地址。主要用于判断对象是否相同，可提高查询、存储操作的效率

**equals()：**比较。但它有不同的提供来源

```
···
    public boolean equals(Object obj) {
        return (this == obj);
    }
    ···
```

Object 类提供的 equals 方法实际上是比较两个对象的哈希值

```
···
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
    ···
```

String 类提供的 equals 方法也会比较哈希值，但并不仅仅之是比较哈希值

如果两个对象的哈希值相同就说明它们包含的内容一定是相同的，直接返回 true，但如果哈希值不同且传参进来的对象非 String 类型则直接返回 false

当两个对象均为 String 类型且长度一致时，则通过 while 循环逐个字符进行比对，并返回最终对比结果

```
···
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    ··
```

**toString()：**获取对象的字符串表现形式，组合方式为：类名 +@+ 十六进制哈希码。当然了，获取这样的数据实际意义不大，一般我们都是通过重写对象 toString() 来传递更多具体的数据，如：重写实体 Bean 的 toString() 观察数据是否正确或完整

```
···
    protected native Object clone() throws CloneNotSupportedException;
    ···
```

**clone()：**创建对象的副本并返回

提供克隆功能的目标对象需实现 Cloneable 接口并重写 clone()，实现此功能遇到以下场景时会涉及两个概念，依需而选：

如果此对象被复制的属性都是基本类型，那么只需要实现当前类的 Cloneable 机制就可以了，这种称之为浅拷贝。如果被复制对象的属性中包含其它实体类对象的引用，且这些实体类对象都需要实现 cloneable 接口并覆盖 clone() 方法，这种称之为深拷贝（其它实体类不实现 Cloneable 机制也可进行拷贝，但就是浅拷贝了，这时指针是指向此实体类原地址的，而非新建地址，因为它并未创建副本）

浅拷贝：被复制对象的所有值属性都含有与原来对象的相同，而所有的对象引用属性仍然指向原来的对象  
深拷贝：在浅拷贝的基础上，所有引用其它对象的变量也进行了 clone，并指向被复制过的新对象

```
···
    public final native void notify();
    ···
```

**notify()：**唤醒正在等待此对象的监视器

线程成为此对象监视器的方法有三种：通过执行此对象的 `Synchronized` 方法、通过执行属于此对象的 Synchronized 代码块、通过执行该类的静态 `Synchronized` 方法，如果该线程不是锁的持有者，则会抛出 `IllegalMonitorStateException` 异常

当唤醒发生时，如果有多个线程正在等待此对象，那么其中一个将会被唤醒，但选择是随机的（这取决于虚拟机中本功能的具体实现代码）

```
···
    public final native void notifyAll();
    ···
```

**notifyAll()：**唤醒正在等待此对象监视器的所有线程

但此时这些等待线程不会立即执行，它们需要等待调用 `notifyAll()` 的线程释放掉锁后才会执行。同样的，如果该线程不是锁的持有者，调用 `notifyAll()` 会抛出 `IllegalMonitorStateException` 异常

```
···
    public final native void wait(long timeout) throws InterruptedException;

    ···
    public final void wait() throws InterruptedException {
        wait(0);
    }

    ···
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }
    ···
```

**wait() / wait(long timeout) / wait(long timeout, int nanos)：**使调用该方法的线程释放共享资源锁，然后从运行状态退出，进入等待队列，直到被再次唤醒 或 定时等待 N 毫秒（如果没有通知就超时返回）

使用时首先要获得锁，需在 `synchronized` 方法或 `synchronized` 代码块中调用，由 `notify()` 或 `notifyAll()` 唤醒

```
···
    protected void finalize() throws Throwable { }
```

**finalize()：**资源回收

它会在 gc 启动，该对象被回收的时候调用

### 常见问题

##### final / finally() / finalize() 的区别

final 修饰符，可修饰属性、方法、类。修饰属性时表示常量，只可被赋值一次，修饰方法时表示方法锁定，以防止继承类对其进行更改，修饰类表示常量类不可被继承（final 类中所有的成员方法也都会隐式的定义为 final 方法）

finally 异常处理，它只能用在 try/catch 语句中，并且附带一个语句块，表示这段语句最终一定会被执行（不管有没有抛出异常），经常被用在需要释放资源的情况下

finalized() 资源回收，它会在 gc 启动，该对象被回收的时候调用，用于释放某些资源



## 参考

1. [java的锁池和等待池](https://www.cnblogs.com/tiancai/p/9371655.html)

2. [Object类九大方法之notify和notifyAll方法](https://blog.csdn.net/qq_44590469/article/details/99620813)