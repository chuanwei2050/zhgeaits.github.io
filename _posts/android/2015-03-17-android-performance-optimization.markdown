---
layout: post
title:  "Android的应用性能优化学习笔记!"
date:   2015-03-17 10:06:03
categories: android
type: android
---

这是读《Android的应用性能优化》这本书的笔记和心得。

**代码优化**

减少创建对象的数量。  
优化算法，少使用递归，使用迭代。  
使用缓存效率更高。Android定义了SparseArray类，当key是整数时，它比hashmap效率更高，因为hashmap用的是Integer对象，而SparseArray使用的是int基本类型。使用LruCache是不错的选择，缓存也可以先丢弃那些重建开销很小的项目。  
优化的基本原则是保持应用的持续响应，让主线程尽可能快地完成任务。  
Android提供了工具StrictMode来检测不良行为。开发时候用这个很不错的哦。

**图形**  
优化布局原则：减少创建对象数量，消除不必要的对象，或推迟创建对象。  
使用<merge/>标签和合并布局。因为Activity的顶是FrameLayout。  

**方法数、属性数优化**  
虽然android程序用java来写，但是android跑的不是java虚拟机，java代码编译成的class文件不用直接用，还需要编译成为dex文件，如下图所示。  
网上很多相关的资料，主要是因为android对于方法和field都是使用short类型来建立索引，所以只能2^16是最大值，很容易就超了。  
dx和dexdump命令都在android-sdk-windows\build-tools\21.1.2目录下。  
把jar包转换成dex文件的命令如下：  
dx --dex --verbose --no-strict --output=temp.dex D:\dfm-0.3.1\classes.jar  

统计dex文件里面的方法数，属性数  
dexdump.exe -f D:\android-sdk-windows\build-tools\21.1.2\temp.dex | find "method_ids_size"  
dexdump.exe -f D:\android-sdk-windows\build-tools\21.1.2\temp.dex | find "field_ids_size"  

公司的项目方法数达到了5W，属性数达到了3W，但是用idea编译的时候，却奇怪了，方法数增加，属性数也增加，而且属性数增加得离谱，多了几万，一下子就超了65535，想了很久都不知道为什么，只知道是idea的bug，我也还是菜鸟一个，然后公司的一个高手就分析dex文件，对比idea编出来的包和maven命令编出来的包，用dexdump命令来分析数据，排序，过滤，对比，发现多处的是R文件来的属性，只要发现子项目没有r文件，就从application项目复制一个r文件来编译，这样导致属性数就多了。所以大家都想到，搞一个r文件给子项目就好了，然后就随便搞一个string.xml，就会生成r文件了，那样就O了。主要是学习怎么分析dex文件，这个比较重要。。

![alt build_dex](/image/build_dex.png "build_dex") 

**内存检测**  
使用eclipse的DDMS，看heap就可以。点击“Cause GC”按钮相当于向虚拟机请求了一次gc操作，当内存使用信息第一次显示以后，无须再不断的点击“Cause GC”，Heap视图界面会定时刷新，在对应用的不断的操作过程中就可以看到内存使用的变化。

Heap视图中部有一个Type叫做data object，也就是我们的程序中大量存在的类类型的对象。在data object一行中有一列是“Total Size”，其值就是当前进程中所有Java数据对象的内存总量，一般情况下，这个值的大小决定了是否会有内存泄漏。  
a) 不断的操作当前应用，同时注意观察data object的Total Size值  
b) 正常情况下Total Size值都会稳定在一个有限的范围内，也就是说由于程序中的的代码良好，没有造成对象不被垃圾回收的情况，所以说虽然我们不断的操作会不断的生成很多对象，而在虚拟机不断的进行GC的过程中，这些对象都被回收了，内存占用量会会落到一个稳定的水平；  
c) 反之如果代码中存在没有释放对象引用的情况，则data object的Total Size值在每次GC后不会有明显的回落，随着操作次数的增多Total Size的值会越来越大，直到到达一个上限后导致进程被kill掉。  

**内存分析**  
使用idea或者android studio可以dump出内存*.hprof文件，同样使用eclipse也可以dump出来。  
点击左下角的Android  
选中你的程序的包名  
点击 initiates garbage collection on selected vm  
点击 dump java heap for selected client  

然后使用MAT来分析，安装：  
可以选择eclipse插件的方式安装  
http://download.eclipse.org/mat/1.3/update-site/  
也可以选择单独MAT程序下载安装  
http://www.eclipse.org/mat/downloads.php

用eclipse安装以后，直接file然后打开既可以。

分析方法：  
点击OQL图标，在窗口输入select * from instanceof android.app.Activity并按Ctrl + F5或者!按钮，看看多少activity是泄露没有被回收的。点击一个activity对象，右键选中Path to GC roots，再选中exclude weak/soft references，就可以看到是哪些引用导致activity没有被回收。

一般是handler的runnable对外面有引用，所以runnable要写成静态内部类static inner class，然后runnable对外面的引用要用弱引用。基本能解决，另外可以使用开源的weekhanlder解决。还有一个简单方法的就是在onDestory的时候remove这个runnable就可以了。