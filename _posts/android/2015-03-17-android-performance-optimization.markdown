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

  