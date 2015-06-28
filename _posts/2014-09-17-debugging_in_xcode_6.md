---
layout:     post
title:      "Debugging in Xcode 6"
subtitle:   "Xcode 6 的一些新功能和调试技巧"
date:       2014-09-17 12:00:00
author:     "Qpai"
header-img: "img/cd-background-img.jpg"
---


## Debugging in Xcode 6

## 内容包括

* 更有效的调试GCD
* 浏览你的用户界面
* 集成快速预览到你的类


## Backtraces 回溯

GCD让人崩溃的回溯记录

![fd](img/GCDBefore.png "")

看到了没有，加入使用GCD异步启动一个任务，等到了断点的时候，只有GCD所在的方法能显示出来

![fd](img/GCDNow.png)

可是现在，完全看到了整个堆栈记录，还知道来自哪个线程

![fd](img/iconDebug.png)

这些图标都是什么意思？灰色又是代表什么？

live的堆栈是彩色的

记录堆栈帧的图标是灰色的,因为它是一个视觉提示让你知道它已经是历史。

* Historical
* No console interaction 不能在控制台中交互，因为已经不在内存中了
* No frame variables 

![fd](img/debugControllerBtn.png)

这里有一个按钮，叫Process View Option Selector，点击它，我们可以看到三个选项：

* 按照线程查看进程
* 按照队列查看进程
* 查看UI层级

点击第二个选项

![fd](img/blockQueue.png)


我写了一个嵌套的block异步执行任务，上面的绿色icon下面的是正在执行的block，下面灰色的是已经执行完的block。

假如队列中有N个串行任务，断点断到的任务是执行中的任务，而xcode 6也会显示等待中的任务。

![fd](img/sequenceBlock.png)

所以Xcode 6 中会显示等待执行的block

![fd](img/serialDebug.png)

## Debug 用户界面

还记得我们之前Process View Option Selector按钮下面有一个查看UI层级的选项吗？点一下看看会发生什么？

![fd](img/UIDebug.png)

看到了我们Demo的当前界面对吗？每个视图还加上了边。

而Xcode 6的左边还有层次关系，还能搜索呢，对吧。

![fd](img/UIDebugHierarchy.png)

而左下角的两个小按钮，左边的那个点中之后，显示主要视图，右面那个，可以把所有隐藏的视图显示出来。

![fd](img/3DUIDebug.png)

看啊，是不是很帅，可以3D看层次，还可以选择某个层次的视图，是不是很方便。之前有`http://revealapp.com/`可以做到这件事，不过收费了，而且还需要嵌入SDK，现在有了xcode 6，码农们有福了。

![fd](img/UIDebugToolButton.png)

我们来看看这些按钮都是做什么的。

第一个可拖动的按钮，向左滑，层级之间的间距扩大
第四个是复原位置
第五个是放大缩小视图
第六个是从右向左或者从左向右依次隐藏视图
第二个第三个点了没反应

## 快速预览

Xcode 最近一年的更新增加了很多功能，以前预览功能，就是那个小眼睛，能看的东西很少，比如UIImage，现在变得多了很多：

![fd](img/quickView.png)

而且，我们可以实现自定义View的预览。

只需要在对象内部实现`- (id)debugQuickLookObject`即可



##参考

[Debugging in Xcode 6](http://devstreaming.apple.com/videos/wwdc/2014/413xxr7gdc60u2p/413/413_debugging_in_xcode_6.pdf?dl=1)


[debugQuickLookObject示例](https://github.com/toyship/KTest)