---
layout: post
title: "使用代码来控制AutoLayout"
date: 2015-07-16 14:15:18 +0800
comments: true
categories: iOS
---

[Apple参考文档](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/AutoLayoutinCode/AutoLayoutinCode.html)

### 创建Constraint

你可以使用`NSLayoutConstraint`来表示constrains。

如果要创建constraints，一般要使用 `constraintsWithVisualFormat:options:metrics:views:`。

第一个参数是一个`visual format string`，这种`visual format language`已经尽可能的是自我解释的。一个view使用一个方括号来代表，view之间的连接用横杠来代表。`visual format language`的语法参见[Visual Format Language](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH3-SW1)

比如说，

{% img center /images/iOS/2015-07-16_1.png 273 %}

上图可以表示为:

	[button1]-12-[button2]

如果是标准的Aqua距离的话，可以不用标注数字
	
	[button1]-[button2]

<!--more-->

view的名字来自于参数`views`字典。key就是这里用到名字，而value是对应的view对象。

可以使用`NSDictionaryOfVariableBindings`来创建该字典，该函数使用view对象的名字来作为key值

	NSDictionary *viewsDictionary =
        NSDictionaryOfVariableBindings(self.button1, self.button2);

	NSArray *constraints =
        [NSLayoutConstraint constraintsWithVisualFormat:@"[button1]-[button2]"
                            options:0 metrics:nil views:viewsDictionary];

并不是每一种constraint都可以通过visual format language来表示的。比如有一种常用的constraint不能用该语言来表达，比如固定宽高比，这时候需要用这样的函数：

`constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constraint:`

比如上面contraint还可以这样创建

	[NSLayoutConstraint constraintWithItem:self.button1 
	                             attribute:NSLayoutAttributeRight
                                 relatedBy:NSLayoutRelationEqual 
                                    toItem:self.button2
                                 attribute:NSLayoutAttributeLeft 
                                multiplier:1.0 
                                  constant:-12.0];


### Installing Constraints

我们必须把constraint加到view中，这个view应该是所有涉及到的view的祖先view，通常是最近的那个祖先view。

关于这个我们可以使用`NSView`的`addConstraint:`

相关的函数还有：

* `removeConstraint:`
* `constraints`
* `fittingSize`


### 参考资料

* 最重要的一个开源库可以帮助我们不用写那么多无聊的代码

[Masonry](https://github.com/SnapKit/Masonry)

	[view1 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(superview).with.insets(padding);
	}];


* [从此爱上iOS Autolayout](http://segmentfault.com/a/1190000000646452)
* [Beginning Auto Layout Tutorial in iOS 7: Part 1](http://www.raywenderlich.com/50317/beginning-auto-layout-tutorial-in-ios-7-part-1)
* [Beginning Auto Layout Tutorial in iOS 7: Part 2](http://www.raywenderlich.com/50319/beginning-auto-layout-tutorial-in-ios-7-part-2)
