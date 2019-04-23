---
layout: post
title: "如何禁止ListView的item项获得焦点,而让item的子控件获得焦点"
tags: [Android, ListView]
comments: true
---

在机顶盒开发中会遇到这样的需求，listview上的item项中有一张图片（item里的ImageView子控件），当按下机顶盒遥控器的方向键时

让listview的某一个item项里面的图片获得焦点（如下图）

![img](/images/22/1.jpg)

而不是让item自身获得焦点（如下图）

![img](/images/22/2.jpg)

默认的情况下，是listview的item自身获得了焦点（如上图右），也就是说listview的item获得焦点后，没有传递给子控件或者子控件默认不能获得焦点。

这时候我们可能会想到在布局文件里设置listview的`descendantFocusability`属性（焦点传递性）：`android:descendantFocusability="afterDescendants"`，然而无论设置其值为`afterDescendants`还是`beforeDescendants`或`blocksDescendants`都没有达到想要的效果。

其实与item相关的设置在配置文件里面虽然没有，但是代码里还是有的，如下：
```java
listView.setItemsCanFocus(true); //设置item项的子控件能够获得焦点（默认为false，即默认item项的子空间是不能获得焦点的）
```
通过这一行代码即可实现以上需求。

如果没有出现左图的获得焦点高亮效果，可能有以下原因：
1. ImageView默认不能获得焦点，应该设置属性为：android:focusable="true" （如果是ImageButton或Button等则不需要设置，他们默认是可以获得焦点的）。
2. 没有为该ImageView设置自定义drawable图片的的selector（该ImageView其实已经获得焦点了，只是没有看出来而已）。

