--- layout: post category : 翻译 title : Afnetworking 2.0 Tutorial tags : [AFNetworking, ios, blog, developer] --- {% include JB/setup %}

##合法使用CocoaPods

[原文地址](http://blog.bignerdranch.com/4638-using-cocoapods-without-going-court/)




[CocoaPods](http://cocoapods.org/)把基于不同授权协议下的不同作者的源码引入到了你的项目中，这个过程简单的都有点过份。但是当心：感谢CocoaPods，你也以比过去快两倍的速度被卷入了版权麻烦中。

你想想，每个CocoaPod都基于软件授权协议下被分发，这个软件授权协议对你是有要求的。遵守它，你才能使用这个软件。但是如果你不想遵守授权协议的要求内容，你就失去了使用这个软件的权利。然后你必须删除这个pod并且做一些其他的事情。否则，你将会面临：

1. 应用被驳回。
2. 上法庭。



##我不是一个法律从业人员，你知道吗？

在我们开始之前，有些事情必须说清楚：这篇文章不是法律建议，如果你目前遇到了和授权相关的问题，那么请你去找法律顾问。如果你没有理由的担心，并且已经影响了你的生活质量，那么看一下心理医生吧还是。




##不只是COCOA的问题

就算你不是一个Cocoa开发，这也会对你有用，只需要把CocoaPods换成gems/modules/packages/或者其他什么，和我们一起讨论。这也是我们这些在Cocoa平台上正在以光的速度违反软件授权协议的人乐于看到的。

在你的世界里可能流行不同的软件授权协议。检出[tl;drLegal](https://tldrlegal.com/)为一个gist，然后就像吃一顿美味的意大利面条一样好好研究一下授权协议文本。很多授权协议的作者也会解答一些问题，这能帮助你理解和运用软件授权协议。




##没协议不使用

如果没有软件授权协议，默认的软件授权协议如下：作者独家使用，你不能使用。

苹果帮我们做了些事情，在一个新文件的样板文件中包含了一个独家权利声明。很多项目没有去修改每个独立源文件的声明文本，这使人不安。如果有疑问，就检查一下README，LICENSE 或者COPYING文件，去看看是否有一些其他的软件授权协议。如果你没有找到，从你的app中删除这段有害的代码。

当你复制一个组件的代码，你必须确保维护了它的软件授权协议。CocoaPods 维护了，但是如果你手动添加第三方的组件，你就得小心。




###ACADEMIC软件授权协议：注意 & 条款

一个软件授权协议最通常的要求是包括版权注意和软件授权协议条款。MTI软件协议就像下面这样:

    The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
    
    


你应该遵守每个pod设定的最小范围的条款，CocoaPods就是这么做的，它将所有pod的条款复制整合成一个大的授权协议文件，然后帮助你满足这些条款。该文件有两种形式：

1. 在你的app中以Markdown的文件存在。
2. 以Plist的形式被包含在设置中。



显示这些信息是为了让用户更容易看到他正在用的组件，它也给组件的作者以宣传。就是说，仅仅包含在你的app包中的注意和条款可能不够，你的app的每个副本都会包含他们。



很多所谓的学术许可证其实什么要求也没有。一般学术许可证包括这些：

1. [麻省理工许可证（MIT,又称X 11）](http://opensource.org/licenses/MIT)
2. [新版3条款的BSD license (Berkeley Software Distribution 伯克利软件套件 开源协议)](http://opensource.org/licenses/BSD-3-Clause)
3. [简版2条款的BSD licenes](http://opensource.org/licenses/BSD-2-Clause) 
4. [最简1条款的BSD licenes](http://opensource.org/licenses/ISC)



###APACHE 2.0

在冗长的注意和条款上面，Apache 许可证加上了：“你必须在你修改的文件显著位置标明：你修改了文件”

源码控制让你更容易发现你修改了开源协议文件，如果你修改了，插入一个评论的权利在Apache许可声明下面，像下面这样写上：“MODIFIED (年份) BY (修改人名字). Modifications licensed under (LICENSE).”给一个注意，问题就解决了。

还有一件事情需要注意：如果这库包含了注意文本文件，那么你需要把它包含到通用注意和条款所在的文件夹中。如果没有注意文件，你就可以忽律法律问题，算你走运。



###LGPL

LGPL 是GNU库（2.0版本）或者Lesser General Public License（2.1版本及以后版本）。很少有iOS软件支持LGPL。


    
我们唯一的选项是静态引入到其他代码，所以基于LGPL的组件必须为使用者开源。使用者也可以修改，并且发布他们自己的版本，但是也必须开源。

	I recommend you steer clear of using any LGPL-licensed software in a close-sourced app.

我建议你的闭源应用完全不要包括任何使用LGPL条款的软件。

###GPL

The GPL is the GNU General Public License.
GPL就是GNU General Public License。如果你的应用中包含任何限制在GPL条款下的代码，都不能被发布到appstore中。GPL禁止增加任何条款或者条件在GPL的条款之上。APPStore增加了额外的条款在GLP之上，所以非GPL的代码才能出现在Appstore中。

如果你不是要发布到appstore中，那么你可以拉取GPL库，记住，你的代码也要获得GPL的许可。

###条款会混合使用吗？

有些软件授权协议是和其他授权协议不兼容的。有时意味着你不能在你的应用中同时使用这些不同协议的代码。不过，当今大多数情况下，这不是个问题。除了GPL，你基本不需要担心这个问题。

你必须去读你使用的代码的授权协议，并且确保你真的看懂了。你认真读几次，你基本就能将不同协议之间的作用弄清楚。

###总结

* 如果你不遵守软件授权协议，那么你就是亵渎法律
* 使用MIT或者BSD条款下的代码，只需要将授权协议和版权文件，放到你的应用包里。
* 用Apache 2.0协议的代码是免费的，增加授权协议文件和版权文件在你的应用包中，放到MIT或者BSD同样的位置。必须复制完整的授权协议文本。
 1. 如果你改变了任何的Apache文件，那么你必须为你的改变增加一个突出显示的NOTICE。
 2. 如果已经有了NOTICE文件，那么就复制到你的应用包中，放到一个用户可以看到的地方，设置模块是比较理想的地方。
* 不用考虑GPL或者LGPL的代码，除非你想为它头疼或者被AppStore驳回。