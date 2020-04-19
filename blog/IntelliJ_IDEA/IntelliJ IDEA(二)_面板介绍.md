> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://www.cnblogs.com/jajian/p/7994126.html

**_1_**|**_0_** **一、面板说明**
==========================

IDEA 面板的全貌如下图

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170811355-1484626138.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170811355-1484626138.png)

**_2_**|**_0_**** 二、菜单栏**
=========================

下面会简单介绍下一些常用的部分菜单使用，如有疑问或补充欢迎留言。

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170831355-115731291.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170831355-115731291.png)

### (1)、File 文件

 [![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170849824-949216683.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170849824-949216683.png)

1. New：新建一个工程

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170912511-882340968.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170912511-882340968.png)

可以新建 project，导入已存在的资源 project，从版本控制库导入工程，新建 Module，导入已存在的资源 Module，新建文件（JS，DB，JSP，Java，CSS……），新建 FMXL 文件。

2. Open：打开本地的文件或工程

3. Open URL：

4. Open Recent：打开最近已导入过的工程

5. Close Project：关闭工程

7. Setting：IDEA 配置文件

8. Project Structure：显示当前工程结构

9. Other Setting：全局默认配置

```
Default Settings…，Default Project Structure…
```

IDEA 在 Setting 中某些配置是 For 当前 project 的，也就是意味着你新打开的一个 project 并不能够默认通用这些配置，你需要另外重新配置。你可以在 DefaultSetting 中进行一些全局通用配置。例如：maven 的安装路径，maven 仓库地址，git.exe 地址等。

10. Import Settings：导入 Settins 文件

你可以将自己以前保存过的 settings 文件导入进来，也可以导入外来的 settings 文件，例如换主题皮肤。

11. Exoort Settings：导出 Settings 文件

将自己习惯的 settings 文件导出到本地或云盘，下次在新的地点使用时可以直接导入使用。

……

### (2)、Edit 编辑

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170932667-500494130.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170932667-500494130.png)

1.Undo：撤销

2.Redo Duplicate Line or Selection：重新复制行或选择。（返回撤销之前）

3.Cut：剪切

4.Copy：复制

5.Copy：复制文件路径

6.Copy Reference

7.Paste：粘贴

8.Paste from History…：从剪切板中选择历史复制的内容粘贴

9.Paste：

10.Delete：删除

11.Find：

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170953355-731585101.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209170953355-731585101.png)

……

### (3)、View 视图

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171018027-720619837.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171018027-720619837.png)

1. Tool Windows：一些工具窗口

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171040714-1074338834.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171040714-1074338834.png)

2. Recent Files：最近打开过的文件（Crtl + E）

3. Recently Changed Files：最近做过修改过的文件

4. Recent Changes：最近修改记录

 [![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171347933-1527064196.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171347933-1527064196.png)

5. Quick Switch Scheme…：

6. Toolbar：工具栏（显示 / 关闭）

 [![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171417245-2079949646.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171417245-2079949646.png)

7. Tool Buttons：工具按钮（IDEA 左右和底部的工具框）

 [![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171456761-1429743346.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171456761-1429743346.png)

8. Status Bar：IDEA 右下角的状态栏

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171521402-1323329511.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171521402-1323329511.png)

9. Navigation Bar：

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171545792-1487939373.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171545792-1487939373.png)

……

### (4)、Navigate 导航 

[![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171631745-1574332773.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171631745-1574332773.png)

1. Class：查询类

2. File：查询文件

……

3. Jump to Navigation Bar：跳到导航栏

4.Declaration：进入光标所在的方法 / 变量的接口或是定义处

5.Implementations：方法的实现

6.Type Declaration：进入光标当前所在属性的类

……

7.Type Hierarchy：当前类的分层结构

……

### (5)、Code 编码

 [![](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171654152-945362368.png)](https://images2018.cnblogs.com/blog/1162587/201712/1162587-20171209171654152-945362368.png)

这都是些和编码相关的，重写方法，实现方法，环绕（try…catch，if…else，…），上面显示的快捷键基本都需要记住，因为是比较常用的。

### (6)、Analyze 分析

### (7)、Refactor 重构

这些在项目重构时会使用的加多，例如类名更改，可以通过 Rename(Shift + F6) 来快速替换所有使用该类的地方。

### (8)、Build 构建

构建项目相关的。

### (9)、Run 运行

启动项目相关的，Run，Debug，……

### (10)、Tools 工具

 文件作为模板保存，项目作为模板保存，生成 javaDoc，……

### (11)、VCS 版本控制

 版本控制相关的。

### (12)、Window 窗体

 将当前窗体格式作为默认窗体，激活工具窗体，编辑 Tabs，……

### (14)、Help 帮助

 IDEA 的使用帮助，注册，检查更新，……

**_3_**|**_0_** **三、工具栏**
=========================

 工具栏可通过 View -- Toolbar 来控制显示，如下：

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171221151721490-1347609835.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171221151721490-1347609835.png)

从左至右依次为：

1、打开文件（File -- Open）

2、保存全部（Ctrl + S）

3、同步：（Ctrl+Alt+Y）检测所有外部改变的文件并从磁盘加载

4、Undo：（Ctrl + Z）撤销

5、Redo：（Ctrl + Shift + Z）返回撤销前，防止误撤销

6、剪切：（Ctrl + X）

7、复制：（Ctrl + C）

8、粘贴：（Ctrl + V）

9、查找：（Ctrl + F）

10、替换：（Ctrl + R）

11、回退：（Ctrl + Alt + 向左箭头）

12、前进：（Ctrl + Alt + 向右箭头）

13、构建项目：（Ctrl + F9）

14、当前项目 (Run/Debug) 运行配置

15、运行项目

16、Debug 模式运行项目

17、代码覆盖率方式运行项目

何为 “代码覆盖率”？这里应用一下百度百科的，读者可以另寻资料。

> 代码覆盖（Code coverage）是软件测试中的一种度量，描述程式中源代码被测试的比例和程度，所得比例称为代码覆盖率。
> 
> 在做单元测试时，代码覆盖率常常被拿来作为衡量测试好坏的指标，甚至，用代码覆盖率来考核测试任务完成情况，比如，代码覆盖率必须达到 80% 或 90%。于是乎，测试人员费尽心思设计案例覆盖代码。用代码覆盖率来衡量，有利也有弊。

18、停止项目运行

19、AVD 管理器（Android 开发相关）

20、版本控制更新项目，需要项目加入了版本控制（Ctrl + T）

21、版本控制提交 (Commit) 项目（Ctrl + K）

22、当前文件与服务器上该文件最新版本的内容进行比较。如果当前编辑的文件没有修改，则是灰色不可点击。

23、版本控制，显示历史操作（commit，merge）

24、恢复代码，返回上一版本，可选择性恢复（Ctrl + Alt + Z）。

25、打开 Settings 配置界面（Ctrl + Alt + S）

26、项目结构设置（Ctrl + Alt + Shift + S）

27、SDK 管理器

28、IDEA 帮助文档

29、中英文翻译

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171221163007443-769972308.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171221163007443-769972308.png)

30、捕获内存快照。会在用户主目录下生成内存快照 (**hprof** 文件) 压缩包，用于分析内存。

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171221143919146-597816698.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171221143919146-597816698.png)

__EOF__

作　　者：**[JaJian` 博кē](https://www.cnblogs.com/jajian)**  
出　　处：[https://www.cnblogs.com/jajian/p/7994126.html](https://www.cnblogs.com/jajian/p/7994126.html)  
关于博主：沪漂一员，互联网架构方向，如有问题探讨可以[直接私信](http://msg.cnblogs.com/msg/send/jajian)我，亦可下方留言。  
版权声明：本博客的所有原创文章均会同步更新到我的公众号**【小菜亦牛】**上，喜欢的可以左边扫码关注。码字辛苦，尊重原创。  
声援博主：如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。您的鼓励是博主的最大动力！