---
layout: post
title: "App Extension Basics"
date: 2015-03-25 10:37:12 +0800
comments: true
categories: 
---

iOS 8 和 OS X v10.10 开始引入了扩展，这是个很赞的新功能，有了扩展，用户就可以在使用其他应用或者系统功能的上下文中使用我们程序的功能。苹果的文档中介绍扩展使用了这样的标题“App Extensions Increase Your Impact”，这实在是再适合不过了，因为有了扩展，我们不再局限于自己的应用，我们可以出现在任何扩展点允许的位置、合理的上下文中，为用户提供更丰富的有意义的功能。

# 扩展分类

应用扩展一共有7种，并且 iOS 和 OS X 两个系统提供的扩展类型是不一样的，以下列出了所有的类型：

* 今日视图；
* 分享；
* Action；
* 照片编辑；
* Finder 同步；
* Document Provider；
* 自定义键盘；
* Watch 应用；

其中，照片编辑、Document Provider、自定义键盘只在 iOS 上可用，Finder 同步只在 OS X 可用。Apple Watch 应用不用多说，只有 iOS 可用。


# 扩展的工作方式


## 扩展的生命周

应用扩展的生命周期与
应用扩展的生命周期不同于应用本身，如下图所示：

{% img images/app_extension_life_circle.png %}

_图片转自苹果开发文档_

通常情况下，扩展生命最明显的特征就是短，并且其生存环境通常是系统或者其他应用的上下文中。以今日视图的扩展 (Today View Extension) 来说，其启动和结束是由今日视图的操作来决定的，当新建一个今日视图的扩展项目后，我们可以看到 Xcode 自动为我们新建了一个 ViewController，如下图所示：

{% img images/today_view_extension_project_navigation.png %}

需要注意的是像今日视图的扩展对应的是一个ViewController，而不是像应用一样有一个AppDelegate，这个今日视图扩展的生命周期也就是这个ViewController的生命周期。从我目前的开发经验来看，开发大部分的今日视图应用，可以就像我们之前在应用中写一个ViewController没有太大区别，就好像有别的ViewController把我们所写的应用Present出来一样。当今日视图拉下来的时候，如果我们的应用启用了，则会依次调用viewDidLoad, viewWillAppear, viewDidApplea，而当今日视图收起的时候就会依次调用viewWillAppear, viewDidAppear, dealloc。
(目前我已经为我们的产品加上了一个今日视图的扩展，我们将在后续的文章中详细介绍今日视图扩展的开发。)

此外，扩展的生命周期通常比较短所以尽可能避免在其生命周期中做过于复杂的事情，如果一定需要的话，考虑使用后台任务来完成。

# 扩展与应用

与扩展相关联的，有两个 App，一个是扩展需要依赖于包含在其中发布的 Containing App，另一个是在运行时能够与扩展直接产生调用关联的 Host App (更确切来说 Host App 应该是一类 App，这些 App 包含了扩展对应的扩展点，从而能够启动对应的扩展)。

## Containing App

扩展不能够独立发布到 App Store，它需要和应用一起发布，这个应用被称为 Containing App，扩展通常就是 Containing App 功能的延伸，让用户可以在 App 之外的地方方便快捷地使用 App 的部分功能，比如在今日视图中查看当前正在进行的体育赛事的比分。

既然扩展一般是将应用的部分功能提取出来以供更方便的方式调用和展示，那么扩展的功能逻辑就会与应用本身存在重合的地方，建议将共同的逻辑打成一个 framework，然后让应用和扩展把这个 framework 加进去，这样可以复用代码，以后需要更新的时候也更方便。

## Host App

在运行的时候，Host App 与扩展的关联比 Containing App 要更紧密一些。首先，是 Host App 触发了扩展运行；其次，在扩展运行的过程中，它只能与 Host App 直接通信，而与 Containing App 之间的通信比较有限。

## 扩展与 App 的通信

扩展可以以不同的方式与 Host App、Containing App 通信，如下图所示：

{% img images/app_extension_comunication_methods.png %}

开发文档的截图顺带附上了我的一点标注。可以看到，扩展在运行期间可以和 Host App 直接通信，而扩展与 Containing App 的通信则比较受限制，主要可以通过两种方式，其一是我们熟悉的 openURL，这种方法可以跳转到 Containing App；其二是通过将需要共享的数据写在一个约定好的地方，让应用和扩展都去读写这一块的数据，达到数据共享的目标（这种方式需要建立一个 App Group，将应用和扩展都放在同一个组里）。

## 发布时的注意事项

扩展和应用都是以一个 target 的形式存在的，因此在发布的时候需要分别为他们配置 Provisioning Profile 和 Certificate，看到身边有好几个开发的朋友遇到过关于这两个 bundle ID 和签名证书的问题，在这里也顺便说一下，两个 target 的设置应该是这个样子的：

* 如果应用的 bundle ID 是 com.jiaruprimer.wyhabit，那么应用的 bundle ID 应该是 com.jiaruprimer.wyhabit.XXXXX；
* 两个 target 应该使用同一个开发帐号来签名；
* 如果只是调试的话，用＊也是可以的。

# 做什么样的扩展？

从用户的角度来看，扩展与应用有一个很重要的区别在于使用过程中的上下文——应用是独占式的，而扩展则是临时介入式的——这将对用户使用两者的习惯以及对功能的预期都有很大的影响，因此在决定是否做一个新的扩展，决定做什么样的扩展功能时，一定要认真地思考我们所提供的功能特性是否适合既定的上下文。