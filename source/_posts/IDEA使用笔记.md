---
title: IDEA使用笔记
date: 2023-01-24 14:05:56
tags: IDEA
---

# IDEA使用笔记

## 设置调整类

### 代码提示大小写不敏感

![大小写不敏感.jpg](https://i.loli.net/2020/03/12/qvG8rSt5zBPObT6.jpg)



### java注释风格

这里的注释是指通过`Ctrl+/`快捷键自动注释，如果不修改，默认会将`//`添加到行首，看着很别扭，我希望`//`能和前后的代码保持相同缩进。

![注释风格.jpg](https://i.loli.net/2020/03/12/IvmROoJjEuAK9Ly.jpg)



### 调整补全建议唤出的快捷键

![补全建议替换.jpg](https://i.loli.net/2020/03/12/oJGt2jlUVmD63dO.jpg)



### 启动时不自动打开上一次项目

![启动时.jpg](https://i.loli.net/2020/03/12/xgc8Pk1LhAm49ZG.jpg)




### java代码自动换行

![自动换行.jpg](https://i.loli.net/2020/03/12/pxbtqB1o3gOfuSG.jpg)

这里 Ensure right margin is not exceeded 是关键，但因为我使用了`Material Theme UI`工具，样式有了一些变化，所以又调整了一下 Hard wrap at 缩小了页面一行容许容纳的字符数，保证不超出视界。

![缩进.png](https://i.loli.net/2020/03/12/fKM4BZ9LRzqIei1.png)

注意这张图右边黄色框里被箭头指着的很细很细几乎看不见的竖线，就是界限了。如果代码超过了这个界限，在按下`ctrl + alt + L`即要求代码自动格式化后，IDEA 就会尽可能地将代码放回到界限以内。



### 为 Lombok 开启注解预处理

![注解预处理.png](https://i.loli.net/2020/03/12/xQ4bZmKL2aGSPgR.png)



### 代理设置

![proxy1.png](https://i.loli.net/2020/03/12/RT9xVE2GhLcoYq4.png)

如图所示，其中选用 SOCKS 还是和 HTTP 导致的唯一可能不同在于端口（Prot Number），如图所示。HTTP 的端口就是下图中没有前缀的 Port 的端口 10809。 Host name 是固定的 127.0.0.1，这是IP协议规定的表达本机的IP地址。

![proxy2.png](https://i.loli.net/2020/03/12/pEZfm2bQIuiVXtY.png)

要代理，自己首先要有提供代理服务的主机和代理用的软件。代理服务我是找服务商买的，在买的时候服务商又有让设置机场的名字和密码，即这里 Proxy authentication 中的 Login 和 Password。



### git 相关快捷键设置

![gitee相关快捷键.png](https://i.loli.net/2020/03/12/IY2GQzWBaRAT6DK.png)

如图，将 git push 快捷键设定为 `Ctrl + Shift + K`。别看这里好像有很多个地方要设置，其实设置一处其它地方就都同步改了。

善用搜索，add 和 commit 用搜索搜出来改即可。 add  对应的操作全名是 Add to VCS，Commit 对应的是 Comit... ，不带 File， 不是 Commit File。

使用时建议先用 `alt + 9` 切换到 版本控制选项卡下，在 add commit push 一键三连。

### 光标所在行颜色调整

![光标与光标行颜色.png](https://i.loli.net/2020/04/09/pcjs4nPEkOGaUhL.png)

如图所示，其中 Caret 代表光标，即那个一闪一闪的横线，提示着接下来输入的内容会在什么位置。

而 Caret row 就是指光标所在的行的颜色。

除此之外还有许多其它设置可以调整从而使代码看起来更加醒目。

### DEBUG时显示null

![效果图](https://s1.ax1x.com/2020/04/22/JYTYQK.png)

如图所示，如果不在下方的设置中设置的话，这个`null`并不会显式地出现。

![设置方式](https://s1.ax1x.com/2020/04/22/JYTtsO.png)

## 使用的插件

### .ignore

[.ignore](https://plugins.jetbrains.com/plugin/7495--ignore/) 可以帮助快速创建 git 要忽略的文件和文件夹的匹配规则

### Free MyBatis plugin

[Free MyBatis plugin](https://plugins.jetbrains.com/plugin/8321-free-mybatis-plugin/) 免费的加强IDEA对mybatis的支持的插件

### key-promoter-x

[key-promoter-x](https://plugins.jetbrains.com/plugin/9792-key-promoter-x/) 每当用鼠标点击用快捷键就能做到的按钮时，会自动提示你应该用什么快捷键。对多次用鼠标点击的操作，会提示你是否要为它设置快捷键。

### lombok

[Lombok](https://plugins.jetbrains.com/plugin/6317-lombok/) 结合 Lombok 的jar包，可以通过在类上加`@Getter` `@Setter` `@ToString` 这样的注解使类自动生成相关的方法而不用手写。

### Material Theme UI

[Material Theme UI](https://plugins.jetbrains.com/plugin/8006-material-theme-ui/) 对IDEA的布局做了一些调整，并有一些更漂亮的配色主题，使IDEA更好看。

### Rainbow Brackets

[Rainbow Brackets](https://plugins.jetbrains.com/plugin/10080-rainbow-brackets/) 不同层级的括号有着相差相当大的颜色，从而更好确认块的范围。

### Code Glance

[Code Glance](https://plugins.jetbrains.com/plugin/7275-codeglance/versions)  在编辑器右侧开辟一块区域放置当前文件代码的小地图，从而方便大型文件里的移动。

### Grep Console

[Grep Console](https://plugins.jetbrains.com/plugin/7125-grep-console) 通过在插件中设置包含什么关键词的输出在IDEA控制台上的记录要用什么颜色（这一整条都会变色），而使IDEA控制台上产生的信息颜色多样从而更可读。通过设置如`INFO`,`WARN`这种表明日志记录的等级的，可以使可读性大大提升。

![grepconsole.png](https://i.loli.net/2020/03/12/gPdxGsWnC6YoEO2.png)

### IDEALog

[IDEALog](https://plugins.jetbrains.com/plugin/index?xmlId=com.intellij.ideolog) 用IDEA打开`.log`文件后，它会在样式和格式上起到一些帮助。

### Maven Helper

[Maven Helper](https://plugins.jetbrains.com/plugin/index?xmlId=MavenRunHelper) 在 pom.xml 文件下给出一个选项卡，点进去后可以更可视化地方便地管理依赖。

![mavenhelper.png](https://i.loli.net/2020/03/12/RDQt4ChybaSpnGW.png)

### mybatis log plugin

[mybatis log plugin](https://plugins.jetbrains.com/plugin/10065-mybatis-log-plugin/)  在控制台，运行状态等等的底部选项卡中又多了这个插件提供的展示台，里面会记录软件运行后发生的调用 mybatis 对 mysql 的 CRUD 与原生 mysql 语句相近的语句。

### RestfulToolkit

[RestfulToolkit](https://plugins.jetbrains.com/plugin/10292-restfultoolkit) （猜测是通过统计和管理`@RequestMapping`注解）在侧边栏多提供一个该插件提供的展示台，里面包含项目（有意义且仅对web项目有意义）中所有对外的接口。

## 值得记背的快捷键

### 切换底部选项卡

`alt + 1` 切换到文件视图

`alt + 4` 软件运行时的输出控制台 springboot 启动后会自动切换到此选项卡

`alt + 9` 切换到版本控制（git）试图

`alt + F12` 切换到cmd，powershell之类的系统的终端。

### 智能提示继续写什么和智能补完

由于[调整补全建议唤出的快捷键](#调整补全建议唤出的快捷键)小节中的设置，在我使用时为`Ctrl + ,`和`Shift + Ctrl + ,`。

### 获取正在调用的方法的详细信息 和 参数表

比如使用`System.out.println("hello")`时突然忘记了`println`是个什么用处的方法了，那么把光标放到`println`内部，按下`Ctrl + Q`就会展现该方法上面的注释和参数表。光标停留在`println(...)`的参数表中即`...`部分上时，按下`Ctrl + P`就会显示可用的参数表。

### 获取光标在其方法体中的方法的签名

比如在主类的`main`方法里写`System.out.println("Hello")`，假设光标停在`println`里，此时按下`Alt + Q`,会开个小窗飘在左上角展示`main`方法的签名。

### 关心一个『方法/类/接口/成员』在哪里被使用

先把光标放在『方法/类/接口/成员』中间，然后按下`Alt + F7`，下方就会弹出一块展板对应展现其在整个项目中被使用的地方。而在一个类里关心一个成员变量或一个方法里一个变量被使用的位置，则光标在其中后按下`Shift +  Ctrl + F7`,会使使用其的所有地方高亮。在这之后可以用按下`F3`和`Shift + F3`在其中向前向后移动。

### 在当前文件中的语法错误中来回移动

下一个：`F2`，上一个：`Shift + F2`。

### 快速跳转到具体实现的代码那边

按住`Ctrl`键后用鼠标单击感兴趣具体实现的目标，就会跳转到方法的具体实现的代码里。点击一个接口时`Ctrl + 鼠标单击`在还会智能推断这里最可能是用哪个实现而跳转到实现类的代码。如果按的是成员变量之类的不存在实现的，则会跳转到最接近的使用它的地方。

`Ctrl + B`没有智能推断的能力，只会列出来让你选。但明确具体实现在哪里的话，用它效果一样。

### 回到发生跳转前/回到 『回到发生跳转前』 前

`Ctrl + Alt`打底，前者再加`←`方向键，后者再加`→`方向键。

### 列出当前类的方法和成员变量

`Ctrl + F12`，列出后还能使用搜索并从中选择后快速跳转。

### git相关

按照[git 相关快捷键设置](#git 相关快捷键设置)中的设置。

### 展开文件夹/packages

想快速一次性展开整个`src`文件夹，可以使用数字键盘中的`*`一次性展开到底，也可以使用方向键的右`→`展开一层。与之相反的则是数字键盘的`-`号，全部收回，以及方向键的左`←`收起一层。

