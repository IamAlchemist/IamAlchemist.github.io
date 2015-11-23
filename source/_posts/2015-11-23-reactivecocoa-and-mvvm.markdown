---
layout: post
title: "ReactiveCocoa和MVVM简介"
date: 2015-11-23 14:33:38 +0800
comments: true
categories: iOS, Reactive, MVVM
---

MVC模式有一个非常让人头疼的问题就是controller往往过于庞大。`MVVM`是解决的方法之一。

###MVVM的三个结论

* `MVVM`可以兼容当下使用的`MVC`
* `MVVM`可以增加应用的可测试新
* `MVVM`配合绑定机制效果最好

###什么是MVVM

 <!-- more -->

与其专注于说明 MVVM 的来历，不如让我们看一个典型的 iOS 是如何构建的，并从那里了解 MVVM：

{%img center /images/iOS/2015-11-23-mvvm1.png 408 %}

我们看到的是一个典型的 MVC 设置。Model 呈现数据，View 呈现用户界面，而 View Controller 调节它两者之间的交互。Cool！

稍微考虑一下，__虽然 View 和 View Controller 是技术上不同的组件，但它们几乎总是手牵手在一起，成对的__。你什么时候看到一个 View 能够与不同 View Controller 配对？或者反过来？所以，为什么不正规化它们的连接呢？

{%img center /images/iOS/2015-11-23-mvvm2.png 324 %}

这更准确地描述了你可能已经编写的 MVC 代码。但它并没有做太多事情来解决 iOS 应用中日益增长的重量级视图控制器的问题。在典型的 MVC 应用里，许多逻辑被放在 View Controller 里。它们中的一些确实属于 View Controller，但更多的是所谓的“表示逻辑（presentation logic）”，以 MVVM 属术语来说，就是那些将 Model 数据转换为 View 可以呈现的东西的事情，例如将一个 NSDate 转换为一个格式化过的 NSString。

我们的图解里缺少某些东西，那些使我们可以把所有表示逻辑放进去的东西。我们将其称为 “View Model” —— 它位于 View/Controller 与 Model 之间：

{%img center /images/iOS/2015-11-23-mvvm3.png 484 %}

看起好多了！这个图解准确地描述了什么是 MVVM：一个 MVC 的增强版，我们正式连接了视图和控制器，并将表示逻辑从 Controller 移出放到一个新的对象里，即 View Model。MVVM 听起来很复杂，但它本质上就是一个精心优化的 MVC 架构，而 MVC 你早已熟悉。

下图显示了整个变化的过程

{%img center /images/iOS/2015-11-23-mvmcv.gif 784 %}

现在视图控制器仅关注于用 view-model 的数据配置和管理各种各样的视图, 并在先关用户输入时让 view-model 获知并需要向上游修改数据. 视图控制器不需要了解关于网络服务调用, Core Data, 模型对象等. 

view-model 会在视图控制器上_以一个属性的方式存在_. 视图控制器知道 view-model 和它的公有属性, 但是 view-model 对视图控制器一无所知. 你该对这个设计感觉好多了因为我们的关注点在这儿进行更好地分离.
为了更好的理解如何把这些组件组装在一起，以及每个组件对应职责，我们可以看看新的应用架构的模块层级图.

{%img center /images/iOS/2015-11-23-mvvm-layers.png 791 %}


### ViewModel 和 ViewController 在一起，但独立

我们来看个简单的 view-model 头文件，这样可以对我们的新组件 ViewModel 有一个直观印象。我们举一个简单的例子, 假设我们在制作一个推特的客户端，通过在输入框里，输入他们的姓名并点击 “Go”，它可以用来查看任何推特用户的最新回复。
 
我们的例子的界面将会是这样:

* 有一个让用户输入他们姓名的 UITextField , 和一个写着 “Go” 的 UIButton
* 有显示被查看的当前用户头像和姓名的 UIImageView 和 UILabel 各一个
* 下面放着一个显示最新回复推文的 UITableView
* 允许无限滚动

{%img center /images/iOS/2015-11-23-tweeboatplus.svg 320 %}

#### ViewModel的例子

{% gist 8193de9995d7f9bd88cd MYTwitterLookupViewModel.h %}

注意到这些 readonly 属性了么?这个 view-model 暴漏了视图控制器所必需的最小量信息, 视图控制器实际上并不在乎 view-model 是如何获得这些信息的. 现在我们两者都不在乎. 仅仅假定你习惯于标准的网络服务请求, 校验, 数据操作和存储.

sjdifjisdjfisjfijf














###引用

[MVVM 介绍](http://objccn.io/issue-13-1/)
[ReactiveCocoa and MVVM, an Introduction](http://www.sprynthesis.com/2014/12/06/reactivecocoa-mvvm-introduction/)


