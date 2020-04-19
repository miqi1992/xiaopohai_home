# Java中静态常量与静态变量的区别

入下：测试Java中静态常量与静态变量区别的样例，表名两者加载时的区别。  

StaticClass类中定义了静态常量FIANT_VALUE和静态变量VALUE,静态代码块的打印语句表示类被加载

``` java
public class StaticClass {
    static {
        System.out.println("StaticClass loading....");
    }
    public static String VALUE = "static value loading";
    
    public static final String FIANT_VALUE = "final value loading";
}
```

StaticClassLoadTest类用于测试静态变量的加载：  

``` java
public class StaticClassLoadTest {
    public static void main(String[] args) {
        System.out.println("StaticClassLoadTest....");
        printStaticVar();
    }
    
    private static void printStaticVar() {
        System.out.println(StaticClass.FIANT_VALUE);
        System.out.println(StaticClass.VALUE);
    }
}
```

输出：

``` java
StaticClassLoadTest...
final value loading
StaticClass loading...
static value loading
```

输出显示在打印静态常量时，StaticVar类并没有加载，在输出静态变量之前才打印类的加载信息。这表明类在未加载的情况下也能引用其静态常量信息，**原因是因为常量值存储在JVM内存中的常量区中，在类不加载时即可访问。**

> 注： 经过编译优化，静态常量FIANT_VALUE已经存到NotInit类自身常量池中，不会加载StaticClass

Java中的常量池，实际上分为两种形态：**静态常量池和运行时常量池**

所谓**静态常量池**，即*.class文件中的常量池，class文件中的常量池不仅仅包含字符串(数字)字面量，还包含类、方法的引用，占用class文件中大部分空间。这种常量池主要用于存放两大类常量：  字面量(Literal)和符号引用量(Symbolic Reference)，字面量相当于Java语言层面常量的概念，如文本字符串，声明为final的常量值等，符号引用则属于编译原理方面的概念，包括了如下三种类型的常量：  

+ 类和接口的的全限定名
+ 字段名称的描述
+ 方法名称和描述符

而**运行时常量池**，则是jvm虚拟机在完成类装载操作后，将class文件中的常量池载入到内存中，并保存到**方法区**中，我们常说的常量池，就是指方法区中的运行时常量池。  

运行时常量池相对于Class文件常量池的另外一个重要特征是**具备动态性**，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法去运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用比较多的就是String类的intern()方法。  

**String的intern()方法会查找在常量池中是否存在一份equals相等的字符串，如果有则返回该字符串的引用，如果没有则添加自己的字符串进入常量池。**

#### 常量池的好处

常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。  

**例如字符串常量池，在编译阶段就把所有的字符串文字放到一个常量池中**

(1) 节省内存空间： 常量池中所有相同的字符串常量被合并，只占用一个空间。  

(2)节省运行时间：比较字符串时， ==与equals()快。对于两个引用变量，只用 ==判断引用是否相等，也就可以判断实际值是否相等。  

![img](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/20171010170747727)

> 但是不能说所有的静态常用访问都不需要类的加载，这里还要判断这个常量是否属于"编译器常量"，即在编译器即可确定常量池。如果常量值必须在运行时才能确定，如常量值是一个随机值，也会引起类的加载，如下： 
>
> ``` java
> public static final int FIANT_VALUE_INT = new Random(66).nextInt();
> ```

接下来我们引用一些网络上流行的常量池例子，然后借以讲解

``` java
String s1 = "Hello";
String s2 = "Hello";
String s3 = "Hel" + "lo";
String s4 = "Hel" + new String("lo");
String s5 = new String("Hello");
String s6 = s5.intern();
String s7 = "H";
String s8 = "ello";
String s9 = s7 + s8;

System.out.println(s1 == s2);  // true
System.out.println(s1 == s3);  // true
System.out.println(s1 == s4);  // false
System.out.println(s1 == s9);  // false
System.out.println(s4 == s5);  // false
System.out.println(s1 == s6);  // true
```

首先说明一点，在java中，直接使用==操作符，比较的是两个字符串的引用地址，并不是比较内容，比较内容请用String.equals()。

s1 == s2这个很好理解，s1、s2在赋值时，均使用的字符串字面量，说白话点，就是直接把字符串写死，在编译期间，这种字面量会直接放入class文件的常量池中，从而实现复用，载入运行时常量池后，s1、s2指向的是同一个内存地址，所以相等。  

s1 == s3这个地方有个坑，s3虽然是动态拼接出来的字符串，但是所有参与拼接的部分都是已知的字面量，在编译期间，这种拼接会被优化，编译器直接帮你拼好，因此String s3 = "Hel" + "lo";在class文件中被优化成String s3 = "Hello"，所以s1 == s3成立，**只有使用引号包含文本的方式创建的String对象的String对象之间使用"+"连接产生的新对象才会被加入字符串池中**。

s1 == s4当然不相等， s4虽然也是拼接出来的，但new String("lo")这部分不是已知字面量，是一个不可预料的部分，编译器不会优化，必须等到运行时才可以确定结果，结合**字符串不变**定理。鬼知道s4被分配到哪去了，所以地址肯定不同。**对于所有包含new方式新建对象(包括null)的"+"连接表达式，它所产生的新对象都不会被加入字符串池中。**

![java字符串不变](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/082306277835446.jpg)

s1 == s9也不相等， 道理差不多，虽然s7、s8在赋值的时候使用的字符串字面量，但是拼接成s9的时候，s7、s8作为两个变量，都是不可预料的，编译器毕竟是编译器，不可能当解释器用，不能在编译期被确定，所以不做优化，只能等到运行时，在堆中创建s7、s8拼接成的新字符串，在堆中地址不确定，不可能与方法区常量池中的s1地址相同。  

![jvm常量池，堆，栈内存分布](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/082307486111786.png)

s4 == s5已经不用解释了，绝对不相等，二者都在堆中，但地址不同。  

s1 == s6这两个相等完全归功于intern方法，s5在堆中，内容为hello, intern方法会尝试将Hello字符串添加到常量池中，并返回其在常量池中的地址，因为常量池装那个已经有了Hello字符串，所以intern方法直接返回地址；而s1在编译期就已经指向了常量池了，因此s1和s6指向同一地址，相等。  

``` java
public static final String A = "ab";  // 常量A
public static final String B = "cd";   //常量B
public static void main(String[] args) {
    String s = A + B;  //将两个常量用+连接对s进行初始化
    String t = "abcd";
    if(s == t) {
        System.out.println("s等与t, 他们是同一对象");
    } else {
        System.out.println("s不等于t, 他们不是同一对象");
    }
}

s等与t, 他们是同一对象
```

A和B都是常量，值是固定的，因此s的值也是固定的，它在类编译时就已经确定了。也就是说： String s = A + B;等同于： String s = "ab" + "cd";

``` java
public static final String A;  // 常量A
public static final String B;  // 常量B
static {
    A = "ab";
    B = "cd";
}

public static void main(String[] args) {
    //将连个常量用+ 连接对s进行初始化
    String s = A + B;
    String t = "abcd";
    if(s  == t) {
        System.out.println("s等与t, 他们是同一个对象");
    } else {
        System.out.println("s不等于t, 他们不是同一个对象");
    }
}

s不等于t, 他们不是同一个对象
```

A和B虽然被定义为常量，但是他们都没有马上被赋值，在运算出s的值之前，他们何时被赋值，以及被赋予什么样的值，都是个变数。因此A和B在被赋值之前，性质类似于一个变量。那么s就不能在编译期被确定，而只能在运行时被创建了。  

至此，我们可以得出三个非常重要的理论：  

1. 必须要关注编译期的行为，才能更好的理解常量池。  
2. 运行时常量池中的常量，基本来源于各个class文件中的常量池。  
3. 程序运行时，除非手动向常量池中添加常量(比如调用internal方法)，否则jvm不会自动添加常量到常量池。  

以上所涉及仅字符串常量池，实际上还有整型常量池、浮点型常量池(java中基本类型的包装类的大部分都实现了常量池技术，即Byte/Short/Integer/Long/Character/Boolean；Float/Double并没有实现常量池技术)等等，但都大同小异，只不过数值类型的常量池不可以手动添加常量，程序启动时常量池中的常量就已经确定了，比如整型常量池中的常量范围： -128~127，(Byte、Short、Integer、Long、Character、Boolean)这5种包装 类默认创建了数值[-128, 127]的相应类型的缓存数据，但是超出此范围仍然会去创建新的对象。  

例如在自动装箱时，把int变成Integer的时候，是有规则的，当你的int的值在-128~IntegerCache.high(127)时，返回的不是一个新new出来的Integer对象，而是一个已经缓存在堆中的Integer对象，如果你要把一个int变成一个Integer对象，首先去缓存中找，找到的话直接返回引用给你就 行了，不必再新new一个），如果不在-128-IntegerCache.high(127) 时会返回一个新new出来的Integer对象。    

#### 实践

说了那么多理论，接下来让我们接触一下真正的常量池。  

前文提到过，class文件中存在一个静态常量池，这个常量池是由编译器生成的，用来存储java源文件中的字面量(本文仅仅关注字面量)，假设我们有如下的java代码

``` java
String s = "hi";
```

为了方便起见，就这么简单，没错！将代码编译成class文件后，用winhex打开二进制格式的class文件，如图： 

![二进制格式的class文件](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/082317394713776.png)

 简单讲解一下class文件的结构，开头的4个字节是class文件魔数，用来标识这是一个class文件，说白话点就是文件头，既：CA FE BA BE。

紧接着4个字节是java的版本号，这里的版本号是34，因为笔者是用jdk8编译的，版本号的高低和jdk版本的高低相对应，高版本可以兼容低版本，但低版本无法执行高版本。所以，如果哪天读者想知道别人的class文件是用什么jdk版本编译的，就可以看这4个字节。

接下来就是常量池入口，入口处用2个字节标识常量池常量数量，本例中数值为00 1A，翻译成十进制是26，也就是有25个常量，其中第0个常量是特殊值，所以只有25个常量。

常量池中存放了各种类型的常量，他们都有自己的类型，并且都有自己的存储规范，本文只关注字符串常量，字符串常量以01开头(1个字节)，接着用2个字节记录字符串长度，然后就是字符串实际内容。本例中为：01 00 02 68 69。

接下来再说说运行时常量池，由于运行时常量池在方法区中，我们可以通过jvm参数：-XX:PermSize、-XX:MaxPermSize来设置方法区大小，从而间接限制常量池大小。

 假设jvm启动参数为：-XX:PermSize＝2M -XX:MaxPermSize＝2M，然后运行如下代码：

``` java
//保持引用，防止自动垃圾回收
List<String> list = new ArrayList<String>();
        
int i = 0;
        
while(true){
    //通过intern方法向常量池中手动添加常量
    list.add(String.valueOf(i++).intern());
}
```

程序立刻会抛出：Exception in thread "main" java.lang.outOfMemoryError: PermGen space异常。PermGen space正是方法区，足以说明常量池在方法区中。

在jdk8中，移除了方法区，转而用Metaspace区域替代，所以我们需要使用新的jvm参数：-XX:MaxMetaspaceSize=2M，依然运行如上代码，抛出：java.lang.OutOfMemoryError: Metaspace异常。同理说明运行时常量池是划分在Metaspace区域中。具体关于Metaspace区域的知识，请自行搜索。

### 参考

《深入理解java虚拟机 jvm高级特性与最佳实践》