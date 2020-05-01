> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://www.cnblogs.com/vipstone/p/12803528.html

if else 是我们写代码时，使用频率最高的关键词之一，然而有时过多的 if else 会让我们感到脑壳疼，例如下面这个伪代码：  
![](https://user-gold-cdn.xitu.io/2020/4/22/171a0bc1f0e1fcba?w=943&h=853&f=png&s=311699)  
是不是很奔溃？虽然他是伪代码，并且看起来也很夸张，但在现实中，当我们无数次 review 别人代码时，都会发现类似的场景，那么我们本文就来详细聊聊，有没有什么方法可以让我们避免来写这么多的 if else 呢？

我们本文提供了 9 种方法来解决掉那些 “烦人” 的 if else，一起来看吧。

1. 使用 return
------------

我们使用 `return` 去掉多余的 else，实现代码如下。

优化前代码：

```
if ("java".equals(str)) {
    // 业务代码......
} else {
    return;
}
```

优化后代码：

```
if (!"java".equals(str)) {
    return;
}
// 业务代码......
```

这样看起来就会舒服很多，虽然相差只有一行代码，但真正的高手和普通人之间的差距就是从这一行行代码中体现出来的。

**「勿以善小而不为，勿以恶小而为之」「千里之堤，溃于蚁穴」**，说的都是同样的道理。

2. 使用 Map
---------

使用 Map 数组，把相关的判断信息，定义为元素信息可以直接避免 if else 判断，实现代码如下。

优化前代码：

```
if (t == 1) {
    type = "name";
} else if (t == 2) {
    type = "id";
} else if (t == 3) {
    type = "mobile";
}
```

我们先定义一个 Map 数组，把相关判断信息存储起来：

```
Map<Integer, String> typeMap = new HashMap<>();
typeMap.put(1, "name");
typeMap.put(2, "id");
typeMap.put(3, "mobile");
```

之前的判断语句可以使用以下一行代码代替了：

```
type = typeMap.get(t);
```

3. 使用三元运算符
----------

三元运算符也叫三元表达式或者三目运算符 / 表达式，不过代表的都是一个意思，优化代码如下。

优化前代码：

```
Integer score = 81;
if (score > 80) {
    score = 100;
} else {
    score = 60;
}
```

优化后代码：

```
score = score > 80 ? 100 : 60;
```

4. 合并条件表达式
----------

在项目中有些逻辑判断是可以通过梳理和归纳，变更为更简单易懂的逻辑判断代码，如下所示。

优化前代码：

```
String city = "西安";
String area = "029";
String province = "陕西";
if ("西安".equals(city)) {
    return "xi'an";
}
if ("029".equals(area)) {
    return "xi'an";
}
if ("陕西".equals(province)){
    return "xi'an";
}
```

优化后代码：

```
if ("西安".equals(city) || "029".equals(area) || "陕西".equals(province)){
    return "xi'an";
}
```

5. 使用枚举
-------

JDK 1.5 中引入了新的类型——枚举（enum），我们使用它可以完成很多功能，例如下面这个。

优化前代码：

```
Integer typeId = 0;
String type = "Name";
if ("Name".equals(type)) {
    typeId = 1;
} else if ("Age".equals(type)) {
    typeId = 2;
} else if ("Address".equals(type)) {
    typeId = 3;
}
```

优化时，我们先来定义一个枚举：

```
public enum TypeEnum {
    Name(1), Age(2), Address(3);
    public Integer typeId;
    TypeEnum(Integer typeId) {
        this.typeId = typeId;
    }
}
```

之前的 if else 判断就可以被如下一行代码所替代了：

```
typeId = TypeEnum.valueOf("Name").typeId;
```

6. 使用 Optional
--------------

从 JDK 1.8 开始引入 Optional 类，在 JDK 9 时对 Optional 类进行了改进，增加了 ifPresentOrElse() 方法，我们可以借助它，来消除 if else 的判断，使用如下。

优化前代码：

```
String str = "java";
if (str == null) {
    System.out.println("Null");
} else {
    System.out.println(str);
}
```

优化后代码：

```
Optional<String> opt = Optional.of("java");
opt.ifPresentOrElse(v -> 
	System.out.println(v), () -> System.out.println("Null"));
```

> 小贴士：注意运行版本，必须是 JDK 9+ 才行。

7. 梳理优化判断逻辑
-----------

和第 4 点比较类似，我们可以通过分析 if else 的逻辑判断语义，写出更加易懂的代码，例如以下这个嵌套判断的优化。

优化前代码：

```
// 年龄大于 18
if (age > 18) {
    // 工资大于 5000
    if (salary > 5000) {
        // 是否漂亮
        if (pretty == true) {
            return true;
        }
    }
}
return false;
```

优化后代码：

```
if (age < 18) {
    return false;
}
if (salary < 5000) {
    return false;
}
return pretty;
```

我们需要尽量把表达式中的包含关系改为平行关系，这样代码可读性更高，逻辑更清晰。

8. 使用多态
-------

继承、封装和多态是 OOP（面向对象编程）的重要思想，本文我们使用多态的思想，提供一种去除 if else 方法。

优化前代码：

```
Integer typeId = 0;
String type = "Name";
if ("Name".equals(type)) {
    typeId = 1;
} else if ("Age".equals(type)) {
    typeId = 2;
} else if ("Address".equals(type)) {
    typeId = 3;
}
```

使用多态，我们先定义一个接口，在接口中声明一个公共返回 `typeId` 的方法，在添加三个子类分别实现这三个子类，实现代码如下：

```
public interface IType {
    public Integer getType();
}

public class Name implements IType {
    @Override
    public Integer getType() {
        return 1;
    }
}

public class Age implements IType {
    @Override
    public Integer getType() {
        return 2;
    }
}

public class Address implements IType {
    @Override
    public Integer getType() {
        return 3;
    }
}
```

> 注意：为了简便我们这里把类和接口放到了一个代码块中，在实际开发中应该分别创建一个接口和三个类分别存储。

此时，我们之前的 if else 判断就可以改为如下代码：

```
IType itype = (IType) Class.forName("com.example." + type).newInstance();
Integer typeId = itype.getType();
```

有人可能会说，这样反而让代码更加复杂了，此可谓 “杀鸡焉用宰牛刀” 的典型范例了。这里作者只是提供一种实现思路和提供了一些简易版的代码，以供开发者在实际开发中，多一种思路和选择，具体用不用需要根据实际情况来定了。**灵活变通，举一反三，才是开发的上乘心法**。

9. 选择性的使用 switch
----------------

很多人都搞不懂 switch 和 if else 的使用场景，但在两者都能使用的情况下，可以尽量使用 switch，因为 switch 在常量分支选择时，switch 性能会比 if else 高。

if else 判断代码：

```
if ("add".equals(cmd)) {
    result = n1 + n2;
} else if ("subtract".equals(cmd)) {
    result = n1 - n2;
} else if ("multiply".equals(cmd)) {
    result = n1 * n2;
} else if ("divide".equals(cmd)) {
    result = n1 / n2;
} else if ("modulo".equals(cmd)) {
    result = n1 % n2;
}
```

switch 代码：

```
switch (cmd) {
    case "add":
        result = n1 + n2;
        break;
    case "subtract":
        result = n1 - n2;
        break;
    case "multiply":
        result = n1 * n2;
        break;
    case "divide":
        result = n1 / n2;
        break;
    case "modulo":
        result = n1 % n2;
        break;
}
```

在 Java 14 可使用 switch 代码块，实现代码如下：

```
// java 14
switch (cmd) {
    case "add" -> {
        result = n1 + n2;
    }
    case "subtract" -> {
        result = n1 - n2;
    }
    case "multiply" -> {
        result = n1 * n2;
    }
    case "divide" -> {
        result = n1 / n2;
    }
    case "modulo" -> {
        result = n1 % n2;
    }
}
```

总结
--

业精于勤荒于嬉，行成于思毁于随。编程是一门手艺，更是一种乐趣，哈佛最受欢迎的幸福课《幸福的方法》一书中写到「**让我们能感到快乐和幸福的方法，无非是全身心的投入到自己稍微努力一下才能完成的工作中去**！」是啊，太简单的事情通常无法调动起我们的兴趣，而太难的工作又会让我们丧失信心，只有那些看似很难但稍微努力一点就能完成的事情，才会给我们带来巨大的快乐。

#### 最后的话

原创不易，如果觉得本文对你有用，请随手点击一个「**赞**」，这是对作者最大的支持与尊重，谢谢你。

参考 & 鸣谢

[https://www.tuicool.com/wx/2euqQvZ](https://www.tuicool.com/wx/2euqQvZ)

[http://www.blogjava.net/xzclog/archive/2006/10/16/75399.html](http://www.blogjava.net/xzclog/archive/2006/10/16/75399.html)