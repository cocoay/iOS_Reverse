## 1、前言
提到iOS逆向，网上的文章铺天盖地，创作时间从2014年到2019年不等。绝大部分的工具或命令都过时了。书籍更不用说。

我刚刚结束了为期1月的零基础iOS逆向研究，现已经成功逆向了微信的一些复杂逻辑。现在分享一下我的一些学习路线和心得。

* _郑重声明：本次逆向仅限于对iOS底层知识的学习，无任何恶意行为_ 

## 2、工具篇
#### 2.1 MonkeyDev 逆向集成环境
_工欲善其事 必先利其器_
iOS后起之秀**AloneMonkey**在旧的逆向集成环境**iOSOpenDev**的基础上，进行了升级。变成了一款非越狱插件开发集成神器：**MonkeyDev**，它提供了我们一个像开发普通iOS应用一样简单的开发平台。
[源码地址](https://github.com/AloneMonkey/MonkeyDev)
[安装和使用方法请看这里](http://www.alonemonkey.com/2017/07/12/monkeydev-without-jailbreak/)
我们先看一眼项目的创建：
![跟创建其他iOS项目一样简单](https://img-blog.csdnimg.cn/20190912160625684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyNDE1NTI=,size_16,color_FFFFFF,t_70)
再来看一眼**project**结构目录：
![拖入无壳的app即可](https://img-blog.csdnimg.cn/20190912160724779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyNDE1NTI=,size_16,color_FFFFFF,t_70)
只需要拖入一个砸壳的应用即可。关于如何砸壳，这不是逆向的重点，所以本篇不教，有兴趣的同学可以自行查资料解决。可以先到**PP助手**的应用商店下载脱壳应用。

**该平台集成了一下hook工具：**

**CaptainHook：**
通过runtime hook OC函数. [CaptainHook基本原理](https://www.jianshu.com/p/8834b4ce8781)

**fishhook：** 
FaceBook出品，通过修改**Mach-O**外部函数地址的方式，达到hook的功能 [fishhook原理 & Mach-O动态加载过程](https://www.jianshu.com/p/4d86de908721)

**tweek：** 
基于越狱设备，让App去加载对应的dylib，然后：
1.修改目标函数前N字节，跳转到自定义函数入口；
2.备份目标函数前N个字节，跳转回目标函数。
[tweek基本原理](https://www.jianshu.com/p/00eb4eab36ef) 讲的比较浅，但是没找到详细的原理文章


#### 2.2 iOS基础篇
**A、初识 Mach-O**
[什么是Mach-O 以及查看工具](https://www.jianshu.com/p/7040dd1396f7)

**B、动态链接过程**
提到Mach-O就不得不提 _**动态链接**_
因为看完这些资料，你可能会越来越懵逼，Mach-O是被谁加载的呢？Mach-O怎么跟我们的函数怎么关联上的呢？静态库和动态库的区别是什么呢？
别急 回头看看 [fishhook原理篇](https://www.jianshu.com/p/4d86de908721) ，这些问题基本就可以解答了，
如果你还是不满足，想要更深入的理解动态链接过程，请看阿里iOSer [刘坤的这篇博客](https://blog.cnbluebox.com/blog/2017/10/12/dyld2/)

如果你想对Mach-O有更深入、全面的了解，可以看看这些：
[Mach-O分析：解析一个类](https://www.jianshu.com/p/ef8f7ed2e016) 该篇讲较基础，作为入门比也较合适
[Mach-O 内存分布](https://wenghengcong.com/posts/f13a5377/) 
[Mach-O 文件格式探索](https://github.com/Desgard/iOS-Source-Probe/blob/master/C/mach-o/Mach-O%20%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E6%8E%A2%E7%B4%A2.md)


#### 2.3 必要辅助工具篇
**1、class-dump** 分析mach-o 导出所有类的头文件（非开发中的头文件，它包括所有的属性、类方法、对象方法）
[同样，这里是它的安装和使用说明](https://www.jianshu.com/p/ec62d78fe859)
导出来之后 长这样：（建议用sublime打开，如果你用xcode打开该文件夹，你就会知道我这条建议有多么感人了）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912175527209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyNDE1NTI=,size_16,color_FFFFFF,t_70)
2、反汇编工具：**IDA** 或 **Hopper Disassembler**
**Hopper**可以试用大约20分钟？分析比IDA较快（不过也需要近半小时），但试用版不能保存分析结果，下次打开又要等待半小时。网上的破解版都不好用。
所以我选择购买了**IDA 破解版：**[购买地址](https://www.macdown.com/mac/734.html)
当然，有钱的童鞋，还是建议支持买正版。也就￥900多。
汇编指令和二进制机器码是一一对应的，所以有了Mach-O二进制文件，反汇编回汇编代码 也是可行的。
至于为什么要使用反汇编工具，看看下面的图片就懂了。
![1、函数列表](https://img-blog.csdnimg.cn/20190912181013399.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyNDE1NTI=,size_16,color_FFFFFF,t_70)
1、函数列表。不过看起来还不如我们**class-dump**导出的头文件包，别急，一旦双击某个函数名，就可以看到该函数的汇编代码了。见下图：
![doSearch函数汇编位置](https://img-blog.csdnimg.cn/2019091218132846.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyNDE1NTI=,size_16,color_FFFFFF,t_70)
这是搜索好友的doSearch函数，如果你觉得汇编看不太懂，想要看点能看懂的，按**F5**，**IDA**会帮你转成伪代码，如下图：
![这是之前用hopper工具转化的伪代码，IDA的差不多，就不另行截图了](https://img-blog.csdnimg.cn/20190912181555770.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIyNDE1NTI=,size_16,color_FFFFFF,t_70)
一个细节可以看出：函数地址在整个Mach-O中的偏移位置。



**好了，前期的工具和理论基础，先准备这些就够了，迫不及待的小伙伴们 你们可以开始动手试验了，请移步：** [iOS逆向（二）：实践篇](https://github.com/OPTJoker/iOS_Reverse/blob/master/iOS%E9%80%86%E5%90%91%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9A%E5%AE%9E%E8%B7%B5%E7%AF%87.md)
