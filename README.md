### 分享背景 ###

当前我们App测试主要集中在功能测试方面，缺乏性能测试。二者都很重要，但是针对性及要求不同。

功能代表着业务逻辑，代表着app向用户提供的服，功能测试要求逻辑清晰，思维覆盖面广，注重细节

性能代表着质量，与用户体验息息相关，性能测试会要求一些专业的测试工具，有时涉及一些代码层次的东西

## App 性能测试 ##

当前移动设备越来越多地涌现在我们日常生活中，通过移动设备可以帮助人们更便捷高效的完成很多事，同时越来越多的需求也希望通过移动设备来完成。

相比于传统PC，移动设备有其自身的特点，如屏幕小、移动网络复杂且需要收费、电量有限等。因此，在完成用户一系列需求的背后，我们也面临一系列的问题。比如说，如何能保证开发的App内存开销低？如何保证App在功能不变的情况下足够省电？如何做到页面滑动流畅顺滑？如何保证网络开销尽可能的低？等等

核心性能测试维度大致有内存、电量、流量、流畅度、网络、安装包大小等

#### 一：内存优化 ####

What？ Why？ How？

What：

手机内存这个东西大家都不会陌生，在这里，再解释一下

手机内存一般分为：RAM和ROM。

RAM：运行内存

ROM：机身存储

如：三星note4，RAM 3G，ROM 16G

这里内存优化是指RAM优化

Java内存机制之Stack栈和Heap堆

<img src="./HeapAndStack.png">

1、对象实例数据

实际上是保存对象实例的属性，属性的类型和对象本身的类型标记等，但是不保存实例的方法。实例的方法是属于数据指令，是保存在Stack里面，也就是上面表格里面的类方法。

对象实例在Heap中分配好以后，会在stack中保存一个4字节的Heap内存地址，用来查找对象的实例。因为在Stack里面会用到Heap的实例，特别是调用实例的时候需要传入一个this指针。

2、方法内部变量

类方法的内部变量分为两种情况：简单类型保存在Stack中；对象类型在Stack中保存地址，在Heap 中保存值。

3、非静态方法和静态方法

非静态方法有一个隐含的传入参数，这个参数是dalvik虚拟机传进去的，这个隐含参数就是对象实例在Stack中的地址指针。因此非静态方法（在Stack中的指令代码）总是可以找到自己的专用数据（在Heap 中的对象属性值）。当然非静态方法也必须获得该隐含参数，因此非静态方法在调用前，必须先new一个对象实例，获得Stack中的地址指针，否则dalvik虚拟机将无法将隐含参数传给非静态方法。

静态方法没有隐含参数，因此也不需要new对象，只要class文件被ClassLoader load进入JVM的Stack，该静态方法即可被调用。所以我们可以直接使用类名调用类的方法。当然此时静态方法是存取不到Heap 中的对象属性的。

4、静态属性和动态属性

静态属性是保存在Stack中的，而不同于动态属性保存在Heap 中。正因为都是在Stack中，而Stack中指令和数据都是定长的，因此很容易算出偏移量，所以类方法(静态和非静态)都可以访问到类的静态属性。也正因为静态属性被保存在Stack中，所以具有了全局属性。

Java 的堆是一个运行时数据区,类的(对象从中分配空间。这些对象通过new、newarray、anewarray和multianewarray等指令建立，它们不需要程序代码来显式的释放。堆是由垃圾回收来负责的，堆的优势是可以动态地分配内存大小，生存期也不必事先告诉编译器，因为它是在运行时动态分配内存的，Java的垃圾收集器会自动收走这些不再使用的数据。但缺点是，由于要在运行时动态分配内存，存取速度较慢。

栈的优势是，存取速度比堆要快，仅次于寄存器，栈数据可以共享。但缺点是，存在栈中的数据大小与生存期必须是确定的，缺乏灵活性。栈中主要存放一些基本类型的变量（,int, short, long, byte, float, double, boolean, char）和对象句柄。 

由此可见，GC是针对于Heap(堆)的，对于Stack(栈)来说，由于他FILO的结构，可以自动释放。

Android内存优化也主要是针对于Heap(堆)的优化。

WHY？

还么是以三星note4为例，一部崭新的手机在开机之后，手机内存(RAM，后面不特殊说明都是指RAM)并不是像它所宣传的那样，可用内存并没有3G，系统内存往往会占用一部分，而剩下的才是能为我们所用的

理论上来说，空闲内存越大，手机才会越流畅，因此才会有很多安全软件提供清理已打开的APP，来释放内存的功能。

随着近些年的安卓手机硬件快速升级，手机的内存也越来越大，从最初的512M到1GB，2GB，到现在的3GB，4GB等

但是即便如此，目前APP在各方面要求提高，以及打开APP数量的不断增加，我们的手机内存也会被迅速充满

其实我们在用安卓手机的时候不用太在意剩余内存，这源于Android系统机制，当内存不足时，会进行内存调度。这里有一个阀值，只有低于这个值时系统才会按照优先级关闭一些用户不用的进程，释放内存，供需要的程序使用。

Android进程的优先级

- 前台进程
- 可见进程
- 服务进程
- 后台进程
- 空进程

那为什么内存少的时候运行程序会慢呢，原因是：在内存剩余不多时运行程序时会触发系统自身的进程调度策略，这是十分消耗系统资源的操作，特别是在一个程序频繁向系统申请内存的时候。这种情况下系统并不会关闭所有打开的进程，而是选择性关闭，频繁的调度自然会拖慢系统。

对于越低端的手机触发系统调度策略的时候自然越慢，这也就是手机卡的原因。

而对于不同的手机对于每一个应用所分配内存上限有所不同，若超出该上限，则会引发OOM，使用如下命令可以查看该上限

`adb shell getprop dalvik.vm.heapgrowthlimit` 获取极限堆的大小

`getprop`命令可以获取android系统属性，其中是以key，value形式，储存的，后面跟参数key，可以查询到对应的value值

如：
三星note3 128M；三星note4 256M

一般情况下，越低端的手机该上限越低。

使用getprop命令可以查很多信息

`adb shell cat build.prop`  getprop命令获取的值，与该文件中所存储的值对应

`dalvik.vm.heapstartsize`  堆分配的初始大小，这个值越小，RAM消耗越慢，但是，使用内存较多的应用，会引发gc和进程调度策略，应用反应反而更慢。反之，这个值越大，系统RAM消耗越快，但是程序更流畅

`dalvik.vm.heapgrowthlimit`  极限堆大小，dvm heap是可以增长的，但是正常情况下不会超过heapgrowthlimit，如果超过将引发oom

`dalvik.vm.heapsize`  使用大堆时，极限堆大小(使用大堆，需要在manifest中指定android:largeHeap=true)如三星note4，该值为512M

而我们做内存优化就是要减少系统的进程调度，减少GC，防止手机卡顿，还有防止OOM。

GC(Gabage Collection)：垃圾回收

OOM(Out Of Memory)：内存溢出

HOW？

最严重的内存问题OOM通常是由于内存泄露引起的，在这里介绍一个简单，实用，有效的检查方法。

工欲善其事必先利其器

使用工具

- Android Studio/Android Device Monitor(DDMS Dalvik Debug Monitor Server)
- MAT(Memory Analyzer Tool)

工具的下载

[Android Studio下载](https://developer.android.google.cn/studio/index.html)

[MAT下载](https://www.eclipse.org/mat/)

其中MAT工具可能会由于一些问题，导致安装后不能使用，这边的解决方案就是安装eclipse，并安装MAT插件 安装地址：http://archive.eclipse.org/mat/1.2/update-site/

工具使用

Android Studio主界面  

-->  Android Monitor

-->  Monitors

<img src="./WelcomeActivity.png">

-->  Memory

<img src="./AndroidMonitor.png">

在这里我们可以看到程序运行时的内存变化，下面就具体介绍一下其中的详细内容

<img src="./AndroidMonitor1.png" width=100%>

1.Free：表示空闲的内存

2.Allocated：已分配的内存

Free + Allocated 即为当前应用程序向系统申请到的总内存

<img src="./AndroidMonitor2.png" width=100%>

3.启动与关闭Memory监测按钮

4.手动触发GC按钮

5.dump java heap 按钮,点击Android Studio就开始干活了，成功后会自动打开 hprof文件。

6.start(stop) allocation tracking按钮先点击一次，然后会看到Memory Recorder开始转动，然后自己开始在APP上面做相应的操作。在合适的时间再点一次，结束记录。

7.系统释放不需要的内存空间

8.应用程序向系统申请内存

特殊情况

<img src="./OOM.png">

该图内存走势不断向上，没有减少，基本可以断定是有内存泄露。

从操作上看，可以记录是页面跳转或是某个操作导致该内存泄露，当然，我们也有更专业的方法。

Monitor中dump java heap按钮可以帮助我们生成某一段操作的hprof文件，没错，就是和DDMS生成hprof功能一样，点击一下开始记录，再次点击，会生成该段时间内操作的hprof文件。在操作之前记得手动GC，如此可确保我们得到的内存使用情况更准确。

<img src="./hprof.png">

生成hprof之后，studio会直接打开该文件，但是现在生成的文件不能直接在MAT中进行分析，需要进行转化

(生成路径：在项目的Captures文件夹下，或者在主窗口中点击Captures按钮，或者选择 View > Tools Windows > Captures，打开Captures窗口也可查看)

我们可以在命令行中使用hprof-conv命令将其转化为MAT可识别的文件

`hprof-conv -z <inputFile> <outputFile>`

(-z：exclude non-app heaps, such as Zygote  由于系统的资源类占据了很大一部分的内存，而其余的前几名也往往是系统类。为了除去这部分对分析的干扰，我们在Android SDK提供的hprof-conv转换时增加这个参数)

图中展示比较混乱，解释一下列属性的含义

App heap：当前app使用的堆

Image heap：当前app在硬盘上的内存映射

Zygote heap：zygote 复制时继承来的库、运行时类和常量的数据集。zygote空间设备启动时创建，从不分配这里的空间。

Class List View

Package Tree View

Total Count：该类的实例总数

Heap Count：所选择的堆中该类的实例的数量

Sizeof：单个实例所占空间大小（如果每个实例所占空间大小不一样则显示0）

Shallow Size：该对象本身占有的内存大小

Retained Size：释放该对象本身占有的内存大小

(Shallow Size：对象自身占用的内存大小，不包括它引用的对象。 
针对非数组类型的对象，它的大小就是对象与它所有的成员变量大小的总和。当然这里面还会包括一些java语言特性的数据存储单元。 
针对数组类型的对象，它的大小是数组元素对象的大小总和。 )

(Retained Size：Retained Size=当前对象大小+当前对象可直接或间接引用到的对象的大小总和。(间接引用的含义：A->B->C, C就是间接引用) 
换句话说，Retained Size就是当前对象被GC后，从Heap上总共能释放掉的内存。 
不过，释放的时候还要排除被GC Roots直接或间接引用的对象。他们暂时不会被被当做Garbage。 )

depth：GC根节点到所选实例的最短路径的深度

Shallow Size：所选实例的大小

Dominating Size：所选实例所支配的内存大小

测试流程

代码：

通常用来进行内存测试的版本是纯净版本，不应该附加多余的Log和调试试用组件。他们可能会分配临时内存，引起更多的GC，导致应用出现缓慢、卡顿现象。

测试场景：

通常分两类：

一类是当前有新开发或改动的某项功能，需要对该功能进行性能测试。此类场景针对该功能组织，包括功能开启前、运行、结束后等测试点。

另一类是整体性能，考察的应用场景，在综合使用情况下的性能指标。测试场景应当包括启动后待机，切换到后台，执行主要功能，以及反复执行各功能后。

各类场景中，经常作为测试重点的有：

- 包含了图片显示的界面
- 网络传输大量数据
- 需要缓存数据的场景

场景转换成用例：

- 结合场景比较操作前后或不同版本的内存变化
- 显示多张图片的前台进程
- 多个场景来回切换
- 长时间运行进程的内存增长

合适的场景配合对应的测试手段就会发现相应的问题。

本文旨在介绍性能测试之内存优化的一些概念知识，更深层次的实践则还需要慢慢研究。

附

自动化测试工具：

1. Monkey
	
	使用adb命令即可启动，并可以设置一系列参数
	
	$ adb shell monkey -p your.package.name -v 500

	附教程链接：[Monkey工具](http://www.cnblogs.com/yyangblog/archive/2011/03/10/1980068.html)


2. AS2.2 Record Espresso + Instrumentation

	Android Studio提供的自动化测试工具，配置好环境之后可以录制操作生成相应代码。

	附教程链接：[Android 傻瓜式自动测试 利用AS2.2 Record Espresso + Instrumentation](http://blog.csdn.net/qq_28195645/article/details/51971452)

参考：

1. 《移动App性能评测与优化》TMQ专项测试团队编著
2. [Android studio Android Monitor介绍](https://my.oschina.net/u/1463920/blog/611737?p=1)
3. [getprop 获取android系统属性](https://my.oschina.net/han21912/blog/134211)
4. [Android进程的内存管理分析](http://blog.csdn.net/gemmem/article/details/8920039)
5. [Android性能优化-内存篇 WIKI](http://99.48.58.205/mediawiki/index.php/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96-%E5%86%85%E5%AD%98%E7%AF%87)
6. [Android内存机制分析1——了解Android堆和栈](http://www.cnblogs.com/scud001/archive/2013/08/15/3258998.html)
