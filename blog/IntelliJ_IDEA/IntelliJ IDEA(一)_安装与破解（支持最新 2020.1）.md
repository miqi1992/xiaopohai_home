**_1_**|**_0_** **前言**
======================

我是从 eclipse 转 IDEA 的，对于习惯了 eclipse 快捷键的我来说，转 IDEA 开始很不习惯，IDEA 快捷键多，组合多，记不住，虽然可以设置使用 eclipse 的快捷键，但是总感觉怪怪的。开始使用的时候自己也在网络上收集各种 IDEA 使用的教程，但是很多都不全，东说一点西说一点，因此我想在这里整理一份全而整的使用教程系列，不定时更新。

**_2_**|**_0_** **官网下载**
========================

一般都会去官网下载，官网地址 [IntelliJ IDEA](https://www.jetbrains.com/idea/download/#section=windows)，官网上对于不同的操作系统（windows，macOS，Linux）都有两个版本可供下载

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205194610628-1721397153.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205194610628-1721397153.png)

**Ultimate** 即为旗舰版，功能全面，插件丰富，但是收费，按年收费。如果非要比较的话类似于 myEclipse。

**Community** 即为社区版，免费试用，功能相对而言不是很丰富，但是不影响开发使用。如果非要比较的话类似于 eclipse。

如果有经济实力的话还是建议购买 Ultimate 版使用，但是不是终身的而是一年一付；但是网络上也有破解版的，各位相较而选。

**_3_**|**_0_** **安装**
======================

下载好了安装包，确认已经安装好了 JDK ，双击开始安装。

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200348222-2064563648.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200348222-2064563648.png)

 选择安装路径

 [![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200527425-1461784901.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200527425-1461784901.png)

1. 根据你的操作系统类型选择，32 位或 64 位的桌面快捷方式。

2. 选择是否根据文件后缀名关联相应的文件，例如勾选了 Java 以后打开 java 文件默认是以 IDEA 打开，可以不勾选。

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200554003-1282727364.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200554003-1282727364.png)

点击 Install，就开始安装了

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200628863-1602142211.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171205200628863-1602142211.png)

**_4_**|**_0_** **目录说明**
========================

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171206122455566-1188233198.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171206122455566-1188233198.png)

*   Bin：容器，执行文件和启动参数等。
*   Help：快捷键文档和其他帮助文档
*   Jre64:64 位 java 运行环境
*   Lib：idea 依赖的类库
*   License：各插件许可
*   Plugin：插件

**_5_**|**_0_** **破解方式**
========================

**1. [注册码](http://idea.lanyus.com/)激活（已废弃）**

[![](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171206100851706-229737583.png)](https://images2017.cnblogs.com/blog/1162587/201712/1162587-20171206100851706-229737583.png)

 但是这种破解是有有效期的，过了有效期又需要重新破解，下面介绍一个永久破解的方式 - 补丁破解。

**2. 破解补丁**

*   2017.2    版本补丁下载：[https://files.cnblogs.com/files/jajian/JetbrainsCrack-2.6.10-release-enc.jar.zip](https://files.cnblogs.com/files/jajian/JetbrainsCrack-2.6.10-release-enc.jar.zip)
*   2018.3.1 版本补丁地址：[https://files.cnblogs.com/files/jajian/JetbrainsIdesCrack-3.4-release-enc.jar.zip](https://files.cnblogs.com/files/jajian/JetbrainsIdesCrack-3.4-release-enc.jar.zip)
*   2018.3.5 版本补丁地址：[https://files.cnblogs.com/files/jajian/JetbrainsIdesCrack-4.2-release.zip](https://files.cnblogs.com/files/jajian/JetbrainsIdesCrack-4.2-release.zip)

> **最新补丁下载**：
> 
> 这里需要注意：不同的版本，补丁不同。因此这里不能列出所有的补丁文件，如果你是官网最新的 Idea 版本，关注公众号后回复 “IDEA” 可获取最新破解方法。

1.  先下载压缩包解压后得到 jetbrains-agent.jar，把它放到你认为合适的文件夹内。
2.  启动你的 IDEA，如果上来就需要注册，选择：试用（`Evaluate for free`）进入 IDEA，如果之前已经有过破解，请先 remove License。
3.  点击你要注册的 IDEA 菜单：`Configure` 或 `Help` -> `Edit Custom VM Options …`，如果提示是否要创建文件，请点 "Yes"。
4.  在打开的 vmoptions 编辑窗口末行添加：`-javaagent:/absolute/path/to/jetbrains-agent.jar，**这里需要填写你自己的路径**`。
5.  重启你的 IDE。
6.  点击 IDE 菜单 `Help` -> `Register…` 或 `Configure` -> `Manage License…`，支持两种注册方式：License server 和 Activation code。
  
    1). 选择 License server 方式，地址填入：http://fls.jetbrains-agent.com （网络不佳的用第 2 种方式）  
    2). 选择 Activation code 方式离线激活，请使用：ACTIVATION_CODE.txt 内的注册码激活  
    如果激活窗口一直弹出（error 1653219），请去 hosts 文件里 “移除”jetbrains 相关的项目  
    License key is in legacy format == Key invalid，表示 agent 配置未生效。
    

**注意事项：**

一定要自己确认好路径 (不要使用中文路径)，填错会导致 IDE 打不开！！！最好使用绝对路径。一个 vmoptions 内只能有一个 - javaagent 参数。

_示例:_

```
1 mac: -javaagent:/Users/jetbrains/jetbrains-agent.jar
2 linux: -javaagent:/home/jetbrains/jetbrains-agent.jar 
3 windows: -javaagent:C:\Users\jetbrains\jetbrains-agent.jar
```

如果还是填错了，参考这篇文章编辑 vmoptions 补救：[intellij-support](https://intellij-support.jetbrains.com/hc/en-us/articles/206544519).

点击 OK，破解完成，此时在 Help -> About 中可看到有效期至 2089 年结束，够用了吧。

![](https://img2020.cnblogs.com/i-beta/1162587/202003/1162587-20200304110536717-2092796239.png)
