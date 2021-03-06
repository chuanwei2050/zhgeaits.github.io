---
layout: post
title:  "Java GC垃圾回收机制学习笔记!"
date:   2013-06-02 16:00:03
categories: java
type: java
---

1.概念上讲，初始化和创建是彼此独立的，但是在JAVA中，初始化和创建捆绑在一起，两者不分离。在Object类中找不到initialize()方法。

2.每个重载方法都必须有一个独一无二的参数列表，包括顺序不一样，但是不能返回参数不一样，因为调用的时候不一定用到返回参数，所以不确定。

3.实际上，在调用一个对象的方法时，编译器会把修改方法的参数，加入该对象的自身引用进去。

4.构造器是隐式的静态，只有编译器才能调用，但是在构造器里面可以this调用前面位置的构造器，并且只能调用一个。其他地方不能调用构造器。

5.在static方法的内部不能调用非static方法，反过来就可以。

6.在C++中，对象一定会被销毁（如果程序没有缺陷），而在JAVA里的对象却并非总被垃圾回收。
  就是说，对象可能不被垃圾回收，垃圾回收不等于“析构”，垃圾回收只与内存有关。
  使用垃圾回收器的唯一原因是为了回收程序不再使用的内存。在Object里的finalize()方法是native方法，会调用c中的free函数，
  gc在清理对象是调用该对象的这个方法。但是，如果通过一些特殊的方式创建为对象分配了内存，如在屏幕绘制图像，则需要手动擦除这个图像。
  打开文件也需要手动关闭，这些系统资源就是java的内存泄漏原因之一，还有一个就是引用所占的内存，如果不清空回收，也会泄漏的。
  而且不能直接调用finalize方法，他的结果无法预料，是危险的。
  无论是回收还是终结，都不保证一定会发生。如果jvm并未面临内存耗尽的情形，它是不会浪费时间去执行垃圾回收以恢复内存的。
  
7.一种旧的回收机制：给每个对象一个引用计数器，当计数器为0时就回收对象，但是有缺陷，就是如果AB两个对象互相有对方引用时就产生循环，无法回收。
  因此使用一个打标签的方法实现，如果对象还有被引用，就是活的，打一个标签。怎么判断它还被引用？从一个GC Roots开始遍历所有的引用（递归遍历对象
  里的引用），若引用的对象还存活，就打标签，这里的Roots包括，VM栈中的引用，方法区的静态引用，JNI中的引用。
  jvm使用的方法是“停止-复制”：将堆分成两个区域，先暂停程序运行（不是后台运行的，所以慢啊），然后将标记活的对象复制到另一个堆，并修正引用；
  再对原来的堆进行全盘清理。这两个堆的大小比例不一样会影响回收效率，主要是分析对象的存活率问题。
  刚开始的jvm回收是“标记-清除”：将没有标记的死对象清理，但是会产生很对内存碎片，需要整理成连续空间，老的做法是移动活对象，再对活对象边界
  以外进行清理。后面就出现了复制的算法。
  第三种算法：jvm将堆内存划分更大的块，一个块可以存放一个对象，也可以多个对象，每个块有一个代数记录是否还存活，如果块被引用（及块里面的对象），
  则代数（generation count）增加。清理时大对象不会被复制。小型对象被复制整理到其他块。严格来讲，这只是为了提高“停止-复制”的效率，不是新算法。
  总体来说，jvm的gc是一个自适应的算法机制，如果对象稳定，回收率低就切换“标记-清除”算法，如果存在多内存碎片就切换“停止-复制”算法
  
8.静态初始化只有在Class对象首次加载的时候进行一次