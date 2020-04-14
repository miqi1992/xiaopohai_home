> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://mp.weixin.qq.com/s/UnQjP1xVsA-dcGMof1q1vw

前言
--

之前阅读了 JDK 常用容器的源码本章就开始阅读 Mybatis 源码。不过在阅读之前我们首先搭建一下源码阅读环境，这样有利于我们后面的阅读，更加可以一边写注释一边的 Debug。

MyBatis 介绍
----------

首先我们先介绍一下 MyBatis 是什么，如果你比较熟悉的话可以跳过前面的部分。本章主要概念

*   什么是 Mybatis
    
*   为什么要用 Mybatis
    
*   如何使用 mybatis
    
*   mybatis 源码环境搭建
    

### 介绍 

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。同时 Mybatis 前身是 iBatis，iBatis 是 Apache 软件基金会下的一个开源项目。2010 年该项目从 Apache 基金会迁出，并改名为 MyBatis。同期，iBatis 停止维护。知道了这些之后我们就有一个问题，到底为什么要使用`MyBatis`。

为什么用 Mybatis
------------

在实际的开发中我们可以使用 JDBC 来访问数据库，还可以使用 Hibernate 来访问我们的数据库，除此之外还有很多方案可以选择，那么我们为什么要使用 Mybatis 呢？我们使用 JDBC 和 Mybatis 拿来举个例子，来看一下各自的应用场景。本次项目地址‘’‘’‘’‘’‘’‘’‘’‘’‘’, 这是已经整合的一套源码阅读环境，卡下来导入 IDEA 即可使用，同时本文中的例子也在项目中。

### JDBC 连接

创建数据库表

```
create table test.student (
        id int(10),
        name varchar(20),
        primary key (id)
  )
```

insert 添加语句这里就不再贴了，读者可以自行添加数据。![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SRfbKPnrMjEq8TdnQ9EzQRrO6xEaicxc30wP3CdBtKBZnKwK4CxSQicWw/640?wx_fmt=png)使用 JDBC 我们就不使用 Model 类进行演示了，然后链接数据库执行 sql 语句

```
@Test
  public void testJDBC(){
    Connection conn = null;
    Statement stmt = null;
    try {
      //加载驱动
      Class.forName("com.mysql.jdbc.Driver");
      conn = DriverManager.getConnection("jdbc:mysql://miniapp.lqcoder.com:3306/test","root","123456");
      // 执行查询
      stmt = conn.createStatement();
      String sql;
      sql = "select * from test.student";
      ResultSet rs = stmt.executeQuery(sql);
      // 展开结果集数据库
      while(rs.next()){
        // 通过字段检索
        int id  = rs.getInt("id");
        String name = rs.getString("name");
        // 输出数据
        System.out.print("ID: " + id);
        System.out.print("名称: " + name);
        System.out.print("\n");
      }
      // 完成后关闭
      rs.close();
      stmt.close();
      conn.close();
    } catch (ClassNotFoundException | SQLException e) {
      e.printStackTrace();
    }finally{
      // 关闭资源
      try{
        if(stmt!=null) stmt.close();
      }catch(SQLException se2){
      }// 什么都不做
      try{
        if(conn!=null) conn.close();
      }catch(SQLException se){
        se.printStackTrace();
      }
    }
  }
```

输出结果：

> ID: 1 名称: xiaomingID: 2 名称: xiaozhang

### 使用 Mybatis

由于我们这里是在源码中使用的查询所以不需要添加 Mybatis 依赖。首先创建 mybatis 的配置文件，新建一个`mybatis-config.xml`文件，添加以下配置`配置后期会详细解释，这里只做一个演示`，在下篇文章会详细的讲解整合`SpringBoot`![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SjessLD9MibcvxkibRRoh1Y4GOibhxwiaSRcNm7ZNibSIMB2KAxzENEiaEeyQ/640?wx_fmt=jpeg)然后创建`model`类，这里使用了`lombok`来简化开发，可以自行了解一下。![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SVE8mFRlURzCicHpnTaRGZphUHScBW9EHibrVIl8aAOhsX7HefeKrDO4w/640?wx_fmt=jpeg)创建`mapper`文件![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SGQBz2uIBdI8XM9Hf2bicv5Zo6shKhs7wiakB7icgotqLOGxArx3DvZhww/640?wx_fmt=png)创建 `dao`接口![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SwkibaYZoibG5cuc0jeKcPfvRmVrxmMibry4Nfk3wNvMViadru6RKYKCU9g/640?wx_fmt=png)然后在 `mybatis-config.xml`配置文件中加入`Mapper`Xml 文件的路径![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SnV5zI7Fr5D5bFdl8SHqC29ZlOeHLrE3GK3X6Z4UoD05HyNeqRt7Uug/640?wx_fmt=jpeg)完整配置![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91Sby9s0aH2MNAibFU7A5QZ3DX8icAq5CTibkrDsIjbyUz9L5wAVttxXb0Pg/640?wx_fmt=jpeg)然后创建测试方法![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SYTPdPXtqd7VXL5v22hynUFNbj5vWRbYIakKGG7U6kmS7FY74XBsREg/640?wx_fmt=png)查询结果：![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SIl86ibXnuGCIHVCU3ebib58PeYVb0dVhibr0lWGzkj4oM3Oxcicbiblz3jQ/640?wx_fmt=png)

对比 
---

接下来总结一下上面两种方式。因为在 Java 中我们访问数据库，有多种选择方案最原始的就是使用  `JDBC`或是通过`Spring` 提供的`JdbcTemplate` 访问数据库也可以使用`Hibernate`, 但是为什么要使用`Mybatis`呢? 我们可以看一下上面使用`JDBC`访问数据库的步骤：

*   加载驱动类
    
*   创建数据库连接
    
*   创建 Statement
    
*   执行 SQL
    
*   处理 SQL 执行结果
    
*   关闭连接
    

但是核心步骤就只有两个分别是执行 SQL 和处理 SQL 结果，从开发来说我们只需要关心这两个步骤，一个简单的查询就需要写一大堆的代码来进行连接，处理什么的特别麻烦，有人会说了那为啥不把上面重复的代码进行封装，对外仅暴露 SQL 执行和处理结果呢？这样当然是可以的，但是对比`Mybatis`来说并不是因为比较繁琐才不去使用`JDBC`的，刚刚也看到了使用`Mybatis`的话也是比较麻烦的需要进行配置，还要创建 XML 文件，还要进行绑定。同样如果一个大型的项目使用`Mybatis`的话有多少 Dao 接口就要有多少 XML 文件，维护起来也很麻烦。

### 真正的问题

第一个问题就是我们使用 JDBC 的时候是需要对 SQL 进行拼接的，很容易就会导致拼接错误，比如多个符号少个逗号的问题第二个问题也是最知名的问题，就是要把 SQL 写到代码中，这样如果想要改动一下 SQL 就需要修改代码，同时也会降低代码的可读性，可维护性，从任何角度来说都非常致命。拼接 SQL 的问题也是有解决的方案使用`PreparedStatement`可以解决拼接问题，同时也可以解决 SQL 注入问题，但是后者是解决不了的。同时现在看一下 JDBC 在处理查询结果的时候，需要手动从 ResultSet 中取出数据然后在进行处理，上方的例子仅仅使用了两个属性，如果是有很多属性想想就头大![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SKNnP1zbCAhNOSoicmsvboibvFbyRHq4KOFgodhZhXCROnkrSu60Tiad3A/640?wx_fmt=jpeg)

为什么使用 Mybatis
-------------

接下来就是为什么要使用`mybatis`，因为使用`mybatis`也是同样的麻烦，Dao 层多了也是存在维护性相关的问题。想要维护的话需要付出很大的代价。想要查询数据库同样也是需要很多步骤![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SaIibA4sL5AiaKU7q8xpicfUVcLV2CLoFaAiajIB5AibCR1L5Eib6qQ56cN8Q/640?wx_fmt=jpeg)首先看一下`mybatis`的步骤

*   读取配置文件
    
*   创建 SqlSessionFactoryBuilder 对象
    
*   创建 SqlSessionFactory
    
*   通过 SqlSessionFactory 创建 SqlSession
    
*   为 Dao 接口生成代理类
    
*   调用接口方法访问数据库
    

如果每次都需要执行这些步骤的话那它和 JDBC 没啥区别，而且一点优势都没有。然而并不是，`这里参考官方文档``SqlSessionFactoryBuilder` 和 `SqlSessionFactory` 以及 `SqlSession` 等对象 的作用域和生命周期是不一样的。`SqlSessionFactoryBuilder`主要用于构建`SqlSessionFactory`工厂类，用完就可以抛弃。但是`SqlSessionFactory`一旦被创建 就应该在应用运行期间一直存在，不应该丢弃或重建。同时 SqlSession 不是线程安全的，不能在多线程中共享，用完即销毁按照自己的需求创建。以上步骤中，读取配置和创建`SqlSessionFactoryBuilder`，构建`SqlSessionFactory`只需要进行一次第 4 和第 5 步需要进行多次创建。最后一步是必须的。同时也把 SQL 语句和代码进行了拆分，提高看可读性，同时`mybatis`对查询结果进行了映射，无需在手动处理`ResultSet`。

总结
--

对于 JDBC 来说每一步都是必须的，至于 MyBatis，它是构建在 JDBC 技术之上的， 对访问数据库的操作进行了简化，方便用户使用。JDBC 可以看作是一种基础服务，`MyBatis`则是在基础服务上面的框架他们的目标不同。使用`Mybatis`不仅简化了传统 JDBC 访问繁琐的问题，还解决了 SQL 语句与代码的高耦合的问题，同样的也有就是对结果集处理的问题 (**Mybatis 对结果进行了映射**)

搭建源码环境
------

上面介绍了`mybatis`是什么、为什么使用、和怎么使用接下来就开始搭建源码环境，同时上面的例子只是简单的使用一下`mybatis`下文将对`mybatis`进行整合看一下在项目中怎么使用。首先我们打开仓库地址

> https://github.com/mybatis/mybatis-3

![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SL9bsicIJjiavggLm6oFuDsQ4c928HZiaQLYibpAFz1sJcPsumZuYqjbxmA/640?wx_fmt=jpeg) 把项目 clone 下来，然后在拉取`parent`项目`parent`中主要就是一些环境依赖。

> https://github.com/mybatis/parent.git

![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SUdhGNOiaicWp0Y9AAzGHGJG48AuWUN1hBjXiaickZ0z6BYzWMWmFhKIw0A/640?wx_fmt=jpeg) 创建文件夹`mybatissource`，其中包含我们刚刚拉取的项目![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91S2kRq2B3fAWSLZmp1JQZybgjFLibWIYHNicxZicOJibibua8VWfnLCJnssIw/640?wx_fmt=jpeg)然后打开`IDEA`点击`File>open`然后找到`mybatissource`文件夹📁，选择`mybatissource`点击打开![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91S7eOATocGibBj5NKLXaVDuB0vibjNZQmS619XURQwDsL2ObQkzvUbdtGQ/640?wx_fmt=jpeg)找到 Pom 文件![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91S8g1DvEQt289ibFMISn8icJiaDPSgkT6usR9P08O0Im6vQ9eHYenvrSVWg/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91Sg3kEa6aMLgLcvf9jsaYwffria9thdHAepz7Te1qBCegM4d6H6aibMhHg/640?wx_fmt=jpeg)等待依赖加载完毕即可完整项目目录![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91S1LOIpsuauuSqpbicI6iadGrianhwzfTU6oCdgxwc8OZC2x7fCWXDqh5Lw/640?wx_fmt=jpeg)然后打开 Mybatis-3 项目在项目 src 下可自行创建目录![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91SlxZLr3R5wo2qVicdLKk4urh0vPLFicphMW4m8RRvIOvotIlnzTSUbdYg/640?wx_fmt=jpeg)**注意测试需要使用 junit 进行测试**![](https://mmbiz.qpic.cn/mmbiz_jpg/5CmNGOozaFIb9uP5ib5mibbMic4bFvRb91S0MEQdP9jkl24ElwyosIl7wqzhM5l6jdb3WwGMpdVFfzwQsHlBtKibyQ/640?wx_fmt=jpeg)以上就搭建好了源码阅读环境，你也可以自行发挥。同时不建议直接在实际项目中进行 debug 查询，最好搭建一个源码环境，这样可以一遍 debug 一遍写笔记。便于后期复习

### 参考

*   一本小小的 mybatis 源码分析书
    
*   Mybatis3 源码深度解析
    
*   https://mybatis.org/mybatis-3/zh/index.html
    
*   百度百科
    

### 推荐阅读

[![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFL0BeGQicib0B5aBXibzxamMM7PbFElGZV06lwy0uJVybH1cyuB35tzRFfaetXalAaK7dAd6kKeHuCTg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg3NDA4MjYyOQ==&mid=2247484202&idx=1&sn=6c29a2e222d545ad89305f39f41aaab0&chksm=ced77b00f9a0f21699399b5b5b5d7da7064d958664339af50c46b9d8cfe184a58787b28c992e&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFLUlQCFNp2GIMdWQO1zhQ8VXKicBSr7bh1icibI5ialFYPsMzrknjvFHeCODCIyUmFIQALGgFnllAvmOw/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg3NDA4MjYyOQ==&mid=2247484219&idx=1&sn=37e163ac7ddf052c16e9c5bf5a687448&chksm=ced77b11f9a0f2070b419a0029640af8fa5b01f391e35e7c44a09e17fdd5088e7e06e762b503&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFLhYpvgdTlQXZNpZ3oosfRgCCib41zqbJYcdicmYjUeyRqZAgbXGsvUDGbYP2Ml1PL3fo70kmYBQibgg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg3NDA4MjYyOQ==&mid=2247484195&idx=1&sn=0fdf8ccc39784647a3a124401835515d&chksm=ced77b09f9a0f21f21ed605cdb75f06a58775233f30bfa54ab5ddb1ce8fac343dc2310e2ead9&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFLXFxyGatkcvlRMjb5a1xmoyY0x9ibLaFrRYFKyGibDYNvKEBp5CN7ZDUu4IHSqTdPg94oMtkyq0Gzg/640?wx_fmt=png)](http://mp.weixin.qq.com/s?__biz=Mzg3NDA4MjYyOQ==&mid=2247484187&idx=1&sn=aea0895fa38c6b20e3fc12b25a5ce03b&chksm=ced77b31f9a0f227e550dba44537da44760b26c5e857bc5c7bf9b710e84b5cab644c86ec5f82&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/5CmNGOozaFJ3HsXur6nnGlLPtlpeRaMkukpy2phqicCFO48Wpn8H88zYeSddfFPia12AYLfVm89s2Ky7SHj2JRNg/640?wx_fmt=png)