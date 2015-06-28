---
layout:     post
title:      "cocoapods实践记"
subtitle:   "cocoapods进阶"
date:       2014-07-20 12:00:00
author:     "Qpai"
header-img: "img/cd-background-img.jpg"
---


# cocoapods实践记

## 背景

公司有一个主客户端包含了很多频道的业务，若干个频道的app客户端，经常需要频道客户端的代码和主客户端的代码进行相互copy to each other。

这样的问题在于，代码会被随意更改，不可复用，无法升级，工作量分散，

所以我主张使用`cocoapods`进行组件管理，当然有同学推荐我使用[git submodule](http://schacon.github.io/git/user-manual.html#submodules) 进行管理。

公司这时有了一个新组件:X，主客户端和频道子app客户端都需要用到，我想，这是一个彻底改变项目开发问题的机会。


## cocoapods介绍

其实不用我介绍，大多数iOS开发者都知道cocoapods。

我们为了避免重复造轮子，经常会使用第三方库，而iOS的项目开发，如果引入第三方库，需要进行很多项目的配置，以前我们只能人肉手动去修改项目文件，现在有了cocoapods，帮助我们以最快的速度将第三方库导入我们项目中，只需一个命令行`pod install`

## 真假美猴王


首先我开始对各个项目进行调研，马上问题出来了：

1. 公司的各个应用中存在相同的第三方库，但是版本不一样。
2. 很多第三方库被改动了
3. 各个项目中存在相同名称的类，但是功能不一样

比如Afnetworking，各个项目使用的是不同的版本，而且被改动了。有项目以前用的是老版本，但是后来看到了升级版本，手动加入了一部分升级功能，还不完全。

经过了一些讨论，我们进行了如下工作，来减少项目代码冲突

1. 对于相同的第三方库，各个项目进行版本调研，查找出项目的原始版本，和目前项目中使用的版本，有何不同，使用categroy等方式进行抽离，保证第三方库的完整和干净。2. 
2. 各个项目尽量使用相同版本的第三方库，各个项目同时进行升级
3. 同时，各个项目的代码文件，必须添加前缀，避免类名冲突。

## cocoapods私有库

为了将新项目X使用cocoapods进行集成，我们在内部的git库中建立了私有的specs库

按照[cocoapods private](http://guides.cocoapods.org/making/private-cocoapods.html)教程的介绍，我们进行建立。

其实github上面的[CocoaPods/Specs](https://github.com/CocoaPods/Specs/)是cocoapods的公共配置库，如果想建立自己的私有配置库，只需要在自己的git服务器上面建一个项目，比如叫DemoSpecs。内容呢，就遵循如下路径：

	├── Specs
    	└── [自定义库名]
        	└── [版本号]
            	└── [自定义库名].podspec
            
这样，私有库就建成了，每当我们需要加入一个私有新库的时候，只需要在Specs中增加一个文件夹，命名为`自定义库名`，在其中放入不同版本的配置文件即可。

当然这个时候，使用`pod search customPod`还是搜不到东西的，你需要把这个DemoSPecs克隆到本地去。

打开终端，输入以下命令

	pod repo add REPO_NAME SOURCE_URL

其中REPO_NAME 是你本地的specs库名，这个可以自定义。而SOURCE_URL是DemoSpecs的git地址。

这时候使用`pod search customPod`就能搜到你私有cocoapod的库了。

## 单体双胞胎

我们在开发新组件X的的过程中，使用了以前的老代码，而新组件的这部分老代码使用的是MRC，而新代码使用了ARC，这是复杂呀。那么如何在配置文件中，对这两部分代码进行区分呢？

podspecs文件中是可以配置`subspec`的，我将两部分代码进行了拆分，幸运的时候两部分代码一部分是数据处理，一部分是UI，那么我建立了两个文件夹分别存放这两部分代码，同时在podSpecs中配置了子`subspec`:

	subspec "data" do |sp|
  	sp.source_files = "Classes/data"
  	sp.requires_arc = false
	end

	subspec "UI" do |sp|
  	sp.source_files = "Classes/UI"
  	sp.requires_arc = true
	end
	
这样，集成的时候，就会对两部分代码进行分别配置。

## 静态库的处理

我们用到了静态库，在开发语音模块的过程中，使用了`libopencore-amrnb.a`的静态库，静态库如何集成进项目中呢？

[vendored_libraries](http://guides.cocoapods.org/syntax/podspec.html#vendored_libraries)给了我们答案：

spec.vendored_libraries = 'libProj4.a', 'libJavaScriptCore.a'

不过切记，这里写的路径，必须是git的路径，把路径写全了，只写一个.a文件名，可集成不进去。

## 为什么我老是集成失败？

我们在项目中使用了`yajl-json`，发现每次集成的时候都失败了。老是有文件没有集成进去，后来发现，cocoapods允许pod在install之后执行其他命令，而yajl执行了`cmake`命令，而我们没有安装`cmake`命令行工具，就导致了我们没有集成成功。



## 为什么使用cocoapods集成之后包变大了？

我想说其实这是一个伪命题，其实包没有大，只是Cocoapods帮你在项目里面多加了几个armv而已.至少我发现的是这样。

## 为什么让我每次都重新生成tag？？

我们在开发的过程中，直接引用的是git地址+tag方式，结果迭代过程中，每次修改，项目组其他成员集成的之前，都必须重新生成一次tag。我承认我犯二了。。其实cocoapods给了解决方案，那就是直接指向git地址，最终发布release版本后，再生成tag，这样就可以了。

## 最后

终于碰到的技术问题都解决了，`pod install`之后，新组件X终于集成进了各个项目里面。

感谢Cocoapods的团队，为我们提供了这么牛的工具!!
