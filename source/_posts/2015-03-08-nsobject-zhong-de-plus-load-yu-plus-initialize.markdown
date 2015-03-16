---
layout: post
title: "NSObject 中的 +load 与 +initialize"
date: 2015-03-08 20:22:53 +0800
comments: true
categories: 
---

在 NSObject 类的文档中，有两个初始化相关的方法，+load 和 +initialize，开发文档中说得很清楚，这两个方法会被 Objective-C Runtime 自动调用，然后其他的解释都不是很好懂，我看了好几遍也没有一个知根知底的感觉，所以今天来整理一下。

# 为啥把它俩放一起说？

* Objective-C 提供的两种自动类初始化方法；
* 都是 NSObject 的类方法；
* 都由 Objective-C Runtime 自动调用；
* 都和“只调用一次”有或多或少的关系；

# 开发文档这样说
## +initialize
* runtime 会在每一个类或者其任意子类刚刚好要被发送消息之前，先发 +initialize 消息给这个类；
* runtime 以线程安全的方式发送 +initialize 消息；
* 父类在子类之前收到该消息调用；
* 如果子类没有实现 +initialize 或者子类显式地调用 [super initialize] 时，父类的实现将出现被调用多次的情况；
* 因为 +initialize 是以线程安全的方式被调用的，并且不同类被调用的顺序是没有保障的，所以一定要注意在 +initialize 方法中尽可能少地做事情；

## 避免 initialize 实现被多次调用
开发文档中提到了，如果子类没有实现 +initialize 方法，或者子类显示调用了父类的 +initialize 将可能导致导致父类的 +initialize 实现被多次调用，如果我们不希望这种多次调用发生，可以如以下的方式来实现：

~~~
+ (void)initialize {
  if (self == [ClassName self]) {
    // ... do the initialization ...
  }
}
~~~

## +load
* +load 在类和分类动态加载或者静态链接的时候调用，并且前提是这个类或分类实现了 +load 方法；
* 父类在子类之前收到该消息调用；
* 类在其分类之前收到该消息调用；

好了，开发文档里面大致就是这些了，你明白这俩方法做什么用怎么用了吗？反正我是没明白。

# 所以有然后
通过查阅博客和 Demo，我发现要理解清楚这两个方法，尤其是 +load 方法，很重要的一点是清楚地理解这两个方法特殊的地方。

+initialize 在继承方面和普通方法是基本一致的，如果子类没有实现该方法，那么自动调用的时候将调用父类的实现，因此也就有了我们之前列出来的那种为了防止被子类反复调用的事情。

而 +load 则很特殊，只有在类实现了 +load 方法的时候，才会被调用，如果子类没有实现该方法，并不会去调用父类的实现。

## 初始化 Category
load 方法对扩展初始化很有用。《iOS 6 Programming Pushing The Limits》一书中提到：
>Objective-C probides a hook called +load that runs when the category is first attached. Like +initialize, youca use this to implement category-specified setup such as initializing static variables. You can't safely use +initialization in a category because the class may implement this already. If multiple categories implemented +initialize, the one that would run wouldn't be defined.

所以，初始化 Category 很适合写在 +load 方法中，而不是 +initialize。最大的问题就在与 +initialize 可能已经被类或者同一个类的其他 Category 所实现，这样就会产生覆盖的问题，最后哪一个 +initialize 被调用了是不确定的，确定的是，不是每个 +initialize 都会被调用。这种情况下使用 +load 就很合适。包括这个类和其所有的 category 在内，只要是实现了 +load 方法，都会被调用到。

## Lazy Invocation
我们知道 +initialize 是将将好在要收到第一个消息调用时由 runtime 自动调用的，这会比 +load 迟很多，也会在一个更加友好的环境中，因此在这里的限制会少很多。