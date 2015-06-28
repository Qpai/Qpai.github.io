---
layout:     post
title:      "分析Facebook pop"
subtitle:   "让你知道，想写一个动画框架从何入手"
date:       2014-09-17 12:00:00
author:     "Qpai"
header-img: "img/cd-background-img.jpg"
---


## 分析Facebook pop

### pop的优势

首先，最重要的一点就是CALayer和presentLayer的属性是一致的，而CoreAnimation是不一致的。

其次，pop的动画是可中断式的，可中断是什么含义呢？假如CoreAnimation removeAnimation，那么运行动画的对象将回到动画开始的状态，而pop removeAnimation将停止在删除动画时，对象所在的状态，不会回到初始状态。

### CADisplayLink和NSTimer的区别

CADisplayLink是一个允许你的程序按照屏幕刷新去重绘的同步计时器对象。CADisplayLink提供了一个selector和target对象，屏幕一刷新就会调用selector，必须把CADisplayLink加入run loop。

屏幕的刷新频率是60HZ，所以CADisplayLink默认一秒刷新60次，当然你可以减少刷新次数，那么某些帧就会被跳过，不去调用selector。

pop就是使用的CADisplayLink。

NSTimer同样也是同步的，是runloop的定时源，runloop需要把NSTimer强引用，NSTimer才能启动。NSTimer的精度是50--100毫秒，但是如果错过了下一个时间点也是有可能的，所以并不精确。

比如设置NSTimer的周期是60s，如果runloop被阻塞了，当判断是不是该调用NSTimer的时候，发现已经61s了，因为可能会碰到阻塞或者其他什么情况，这是runloop就会忽略掉这个时间点。

CADisplayLink不属于定时源，所以runloop观察到屏幕刷新，就会调用selector，所以CADisplayLink相对精确。

### pop如何实现可中断动画

CADisplayerLink 在pop中的回调函数是在`POPAnimator`中的`- (void)render`，每次屏幕重绘的时候，就调用了`render`，在这个方法里面调用了

	- (void)_renderTime:(CFTimeInterval)time items:(std::list<POPAnimatorItemRef>)items

以修改所有多个需要执行动画的对象，在这个方法中，使用了核心动画事务类`CATransaction`，以同时修改多个对象的属性。

执行单个对象渲染的方法

	- (void)_renderTime:(CFTimeInterval)time item:(POPAnimatorItemRef)item

在这个方法里面，调用了

	static void applyAnimationTime(id obj, POPAnimationState *state, CFTimeInterval time)

`applyAnimationTime`里调用传入的writeblock和readblock进行修改和读取对象frame。

同时还执行了这段代码

	if (!state->advanceTime(time, obj)) {
    	return;
  	}

其中`advanceTime`方法，通过时间，刷新了当前时间应该设置的位置，或者说状态。

这里使用了`CACurrentMediaTime()`以获取相对精确的当前时间，是mach_absolute_time()获取微妙级别的时间，再转化成秒。这个时间每次系统更新都会被重置，如果mac，会是这次开机时间到当前时间的时间差，iPhone可能同理，开机时间和当前时间的时间差。

`advanceTime`内部调用了`advance`，这是一个可以被子类重写的方法，不同类型的动画，只需要重写这个方法，加入自己类型动画如何计算下一次刷新的状态逻辑，就可以搞定。



### POPAnimatableProperty

这是pop的动画属性，即，里面封装了所有可变化的kvc属性，每一个属性的name，writeblock,readblock，临界值都被封装成了一个结构`_POPStaticAnimatablePropertyState`

比如frame的变化，就如下所示：

	 {kPOPViewFrame,
    	^(UIView *obj, CGFloat values[]) {
      	values_from_rect(values, obj.frame);
    	},
    	^(UIView *obj, const CGFloat values[]) {
      	obj.frame = values_to_rect(values);
    	},
    	kPOPThresholdPoint
  	},

### pop的新增动画类型

#### PopSpringAnimation 弹簧效果

即对象执行这个动画的时候，会有一个晃动的效果，并且会逐渐衰减幅度

![fds](http://ww1.sinaimg.cn/mw1024/65cc0af7gw1egqpgva69rg208u0fpjtx.gif)

#### PopDecayAnimation 衰减动画

对象移动过程中，有一个缓慢到达，直到停止的效果，类似于UISrollView的拖动过程。

![](http://ww3.sinaimg.cn/mw1024/65cc0af7gw1egmzoapnqwg206i0bm7nn.gif)

### 自定义动画引擎

大家看了这篇文章，是不是有一种貌似很简单的感觉。

那么我们就梳理一下，pop这样的动画引擎都有那些必要部分


* 一个AnimationState状态类型，即我们目前支持什么属性动画，各种属性如何变化
* 一个全局动画单例，来接收屏幕刷新事件，并且保存需要执行动画的对象及其参数
* 各种动画类型，比如贝塞尔曲线类型，Spring类型，Decay类型，等，及其他们在时间变化的过程中，各参数进行变化的逻辑 


最后，如何组织运行动画？

CADisplayerLink接收屏幕刷新-》判断是否有需要执行动画的对象-》判断是什么动画，参数该根据时间怎么变？-》给对象设置新的状态

好吧，一个动画的一帧就结束了。

具体请参考[CHAnimation](https://github.com/cyndibaby905/CHAnimation)

### 参考

[Core Animation系列之CADisplayLink](http://blog.csdn.net/wzzvictory/article/details/22417181)

[Facebook pop](https://github.com/facebook/pop)

[RayWenderlich pop Demo](https://github.com/rnystrom/PopDemos)

[facebook pop 使用指南](http://geeklu.com/2014/05/facebook-pop-usage/)

[CHAnimation](https://github.com/cyndibaby905/CHAnimation)