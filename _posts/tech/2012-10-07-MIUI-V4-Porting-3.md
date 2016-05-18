---
layout: post
title: MIUI V4移植经验分享 （三）—— Smali代码注入 
categories: tech
tags:  MIUI Porting 移植 底包 ROM Smali 代码注入
---

适逢中秋、国庆双节假期，闲暇之余总结了过去一段时间的工作经验，分享给大家，希望对有志于从事miui移植工作的同学有所帮助。  
今天内容比较多，所以调侃的语言少一些，干货多一些，请喜欢轻松幽默的ROMER多担待些。  
以下的内容是对官方[MIUI V4移植教程](http://www.miui.com/thread-430751-1-1.html)的补充，其中一些工具的使用就不在这里赘述，请大家参考官方教程。  
好的，话不多说，进入正题。  

<!--more-->

## 应用场景

Smali代码注入只能应对函数级别的移植，对于类级别的移植是无能为力的。具体的说，如果你想修改一个类的继承、包含关系，接口结构等是非常困难的。但对于修改类成员变量访问控制权限，类方法实现，Smali代码注入的方法是可以实现的。这主要是因为Samli级代码的灵活性已经远低于java源代码，而且经过编译优化后，更注重程序的执行效率。

#### Smali代码注入

本质上讲，Smali代码注入就是在已有APK或JAR包中插入一些Dalvik虚拟机的指令，从而改变原来程序执行的路径或行为。  
这个过程大致分为五步——确定需要注入的Samli代码，确定注入位置，注入Smali代码，编译Smali代码，调试Smali代码。  
总体流程如下图：

![总览]({{ site.url }}/assets/miui_porting_3/overview.jpg)

下面详细说明：  

#### 确定需要注入的Smali代码

首先，确定基线文件——待移植的APK包或JAR包，使用APKTOOL反汇编，生成原始的Samli文件。  

其次，修改对应的APK包或JAR包的java源代码，使用编译系统重新生成新的APK包或JAR包，并用APKTOOL反汇编，生成包含修改后的Samli文件。   

最后，使用比较工具（例如BeyondCompare）比较两次Smali文件，即可提取出需要注入的Smali代码。  

例如下图红色区域就是需要注入的Smali代码

![注入1]({{ site.url }}/assets/miui_porting_3/smali_inject_1.jpg)

#### 确定注入位置

这一步的看似简单，实际工作中有很多难点，主要是有些注入位置比较难确定，需要不断的尝试。使用APKTOOL反汇编待注入的APK或JAR包后，

首先需要确认需要注入的Smali文件是哪个。这个主要是针对含有匿名内部类的Java文件而言。例如，移植PhoneWindowManager.java文件的修改时，反汇编之后会有很多PhoneWindowManager$1.smali, PhoneWindowManager$2.smali...类似的文件。这些文件就是匿名内部类的Smali代码，由于没有名字，所以编译后只能用$XXX来区分。如果带注入的Smali代码是从PhoneWindowManager$5.smali提取的，一般不能够直接将其注入到目标机型的PhoneWindowManager$5.smali文件中，因为不同机型的匿名内部类顺序不同，实现不同，Smali文件也不同。一般需要通过逐个比较PhoneWindowManager$5.smali附近的几个文件的Smali代码，看看其函数调用，函数名字，类继承关系是否相同来确定注入哪个文件。当然对于没有匿名内部类的Java文件可以直接使用对应的Smali文件注入即可。 
 
其次，确定了注入文件之后，就需要进一步确认待注入区域。由于Smali代码中的每个变量的类型是不固定的，再加上编译器的优化，导致不同ROM的APK或JAR包反汇编后，会有很多不同。这个并不影响我们的工作，我们重点关注Smali代码的“行为”——函数调用顺序，逻辑判断顺序，类成员变量访问顺序，即可大致确定注入区域。另外，对于新增的Smali代码区域可以随意些，新增变量直接追加在变量声明尾部即可，新增函数直接增加在文件尾部。总的来说这个工作还是非常经验化的，需要长时间的反复尝试才能更准确的确定注入区域。  

最后，继续上面例子，如图：  

![注入2]({{ site.url }}/assets/miui_porting_3/smali_inject_2.jpg)

图中有很多红色的不同，其中蓝框是我们刚才确定的需要注入的Samli代码。通过上下文匹配，可以发现绿框的位置是Samli代码需要注入的区域。尽管上下有很多指令和变量不同，但是这并不影响我们的工作。  

#### 注入代码

首先，将待注入的Smali代码注入对应的区域。  

其次，对注入的Smali代码进行“本地化”——修改变量、跳转标号、逻辑判断标号等，使之符合当前的Smali代码实现，完成“嫁接”工作。当然，如果情况很复杂，需要重写对应的Smali代码或者重构java源代码，来完成最终的代码注入。Dalvik 虚拟机每条指令含义请[参见这里](http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html)。   

最后，继续刚才的例子，如图：  

![注入3]({{ site.url }}/assets/miui_porting_3/smali_inject_3.jpg)

其中蓝框内是最终移植的代码，可以看到其中修改了move-result-object v2和invoke-virtual {v2, v0, v1, v15}, Lmiui/net/FirewallManager;->onStartUsingNetworkFeature(III)V的变量，这主要是因为invoke-virtual {v2, v0, v1, v15}, Lmiui/net/FirewallManager;->onStartUsingNetworkFeature(III)V在新的Smali代码中v15变量有其他的用途，我们需要找一个上下文无关的变量完成函数调用时的变量传递，所以这里将move-result-object v15改为move-result-object v2。并且修改了onStartUsingNetworkFeature()函数参数列表。  
另外，移植中还有一类关于资源ID的代码注入比较特殊，这里举例说明一下：  

![注入4]({{ site.url }}/assets/miui_porting_3/smali_inject_4.jpg)

现在我们需要将蓝框内的代码注入到绿框的位置，但是其中const v6, 0x10403c1 阻挡了前进的步伐。我们不能鲁莽的将蓝框代码合入绿框，这样会导致资源无法找到运行时错误。我们需要使用反汇编原始ROM的framework-res.apk和目标ROM的framework-res.apk，根据0x10403c1 ID值找到原始ROM framework-res.apk中对应的资源名字，再根据资源名字到目标ROM的framework-res.apk中查找对应的资源ID值。而后将其替换为目标ROM中的资源ID值。所以最终移植后的代码如下图：  

![注入5]({{ site.url }}/assets/miui_porting_3/smali_inject_5.jpg)

需要说明的是0x1开头的资源都是framework-res.apk中的资源，0x2开头的一般是厂商自己的资源，例如摩托的是moto-res.apk,HTC的是com.htc.resources.apk。miui自己的资源是0x6开头，位于framework-miui-res.apk中。

#### 编译Smali 代码

Smali编译过程相对简单，使用apktool b XXX XXX.apk 即可将Smali代码编译成apk或jar包。但是当遇到编译错误时，apktool工具给出的错误信息少之又少，以至于我们只能手动查找哪个文件Samli代码移植错误。  
这里，我总结了一些Smali代码移植时可能遇到的编译错误。希望对各位有用。   

<pre><code>1.函数调用(invoke-virtual等指令)的参数只能使用v0~v15，使用超过v15的变量会报错。修复这个问题有两种方法：  
  A.使用invoke-virtual/range {p1 .. p1}指令，但是这里要求变量名称需要连续。  
  B.增加move-object/from16 v0, v18类似指令，调整变量名，使之小于等于v15。  
2.函数调用中p0相当于函数可用变量值+1，pN相当于函数可用变量值+N。例如函数.local值为16，表明函数可用变量值为v0~v15，则p0相当于v16，p1相当于v17。  
</code></pre>

例如，下图左侧蓝框所在代码编译不过，后来检查了代码所在的函数.local为33，p0相当于v33，所以编译不过，修改为右侧绿框才正常。  

![注入6]({{ site.url }}/assets/miui_porting_3/smali_inject_6.jpg)

	3.跳转标号重叠。这里主要是指出现了两个相同的标号的情况，例如cond _ 11等，导致无法编译过。解决方法就是修改冲突的标号以及相关跳转语句。其实这个标号叫什么都无所谓，你甚至可以叫ABCD _ XXX,只要可以与对应的goto语句呼应即可。  
	4.使用没有定义的变量  

每个函数可以使多少变量都在函数体内的第一句.local中声明，例如.local 30表明这个函数可以使用v0~v29，如果使用v30就会编译错误。  

#### 调试Smali 代码

调试Smali代码主要任务是解决注入代码后导致的运行时错误。具体的说，就是使注入后的Smali代码通过dalvik虚拟机的字节码校验。获取错误的方法相对简单，使用下面两条命令即可：  

```
  adb logcat | grep dalvikvm  
  adb logcat | grep VFY
```

其中VFY的信息会给出Smali代码出错的文件、函数以及错误原因，dalvikvm的信息可以给出调用栈，以及上下文执行过程，都比较贴心。  

这里总结一下主要的运行时错误：  

<pre><code>1.函数变量列表与声明不同，这个主要体现在下面两个方面：
  A.函数调用的变量类型与函数声明不同。通过追踪变量在上下文的赋值动作来解决。  
  B.函数变量列表中变量少于或者多于函数声明的变量。通过核对函数声明来解决。  
2.函数调用方式不正确。例如：public和包访问函数使用invoke-virtual调用，private函数使用invoke-director调用，接口函数使用invoke-interface调用。如果使用错误，会导致运行时错误。需要调整相关的Smali代码。  
3.类接口没有实现。主要是由于增加了新的子类没有实现原有父类接口导致的，只需增加空实现即可修复。  
4.签名不正确。可以通过adb logcat | grep mismatch命令确认哪个package签名不正确。只需对签名不正确的包重新签名即可。当然如果有很多签名不一致的错误，建议大家对所有的APK重新签名。  
5.资源找不到。  
</code></pre>


这个问题的原因有很多种，我这里列举一些常见的原因：  
  
	A.系统资源文件签名不正确，导致没有加载系统资源，进而无法找到相应的资源。其中，系统资源文件是指system/framework/目录下的apk文件。  
	B.Smali代码中的资源ID移植错误，无法在系统资源中找到对应的资源。  
	C.资源相关的类移植存在问题，导致无法加载相关资源。  

另外，调试时，大家可能需要要追踪代码执行路径，但又苦于无法Debug。我这里分享一些简单的追踪方法，希望对大家有用。

<pre><code>1.增加简单的Smali日志信息：  
  A.修改函数的.local变量，在原来基础上增加两个变量，例如v11,v12。  
  B.在需要打印日志的地方增加如下Smali代码   
    const-string v11, "@@@@"  
    const-string v12, "interceptPowerKeyDown enter"  
    invoke-static {v11, v12}, Landroid/util/Log;->e(Ljava/lang/String;Ljava/lang/String;)I   
  如果增加的变量为v28和v29，则需要使用下面的语句。   
    invoke-static/range {v28 .. v29}, Landroid/util/Log;->e(Ljava/lang/String;Ljava/lang/String;)I  
2.打印程序调用栈的方法：  
  A.修改函数的.local变量，在原来基础上增加一个变量，例如v11。  
  B.在需要打印调用栈的地方增加如下Smali代码  
    new-instance v1 Ljava/lang/Exception;  
    invoke-direct {v1, Ljava/lang/Exception;-><init>()V   
    invoke-virtual {v1, Ljava/lang/Exception;->printStackTrace()V
</code></pre>

今天先分享到这里，希望我的这些经验对大家有所帮助。
