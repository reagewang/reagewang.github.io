---
layout: post
title: "Android中ExpandableListView常用属性"
tags: [Android, ExpandableListView]
comments: true
---

ExpandableListView的常用属性，以及一些样式的具体实现

# divider
---
这个属性用于设置父类之间的分割线样式，具体的设置见childDivider。

#childDivider
---
这个属性是用来设置同一个父项下，子类之间的分割线的样式。

文档的描述是：Drawable or color that is used as a divider for children. 也就是说可以是drawable类型或者是color类型的资源。

默认的分割线大概是高度为1px，颜色为google在material design中推荐的#1E000000。

所以如果只是设置颜色的话，仅仅只是改变分割线的颜色。

如果使用drawable资源，似乎不能使用图片来作为子项之间的分割线，只能使用自定义的drawable资源。而且在自定义的drawable资源中，对高度的设置是没有用的，对其它属性的设置会生效。

有一点要强调的是，当子项展开时，它的父项与子项之间的分隔条会变为子项之间的分割线样式，子项未展开时，父项之间的分割线按divider定义的样式，没有定义就按默认的样式。

下面举几个例子：

## 初始状态：
效果图：

![img](/images/20190419/1.png)

## 改变childDivider的属性：
### color类型
```xml
android:childDivider="@color/blue"
```
效果图：

![img](/images/20190419/2.png)

### drawable类型
```xml
android:childDivider="@drawable/child_divider"
```
child_divider.xml文件：
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid
        android:color="@color/red_700"/>
    <corners
        android:radius="3dp"/>
</shape>
```
这里为了显示出圆角的效果，把分割线的高度设置为5dp：
```xml
android:dividerHeight="5dp"
```
效果图：

![img](/images/20190419/3.png)

`注意：不管是divider还是childDivider，如果需要将分割线去掉的时候，都不要将这个属性设置为@null`，如下所示：
```xml
android:divider="@null"
android:childDivider="@null"
```
这样设置会出问题，虽然在Android Studio中显示这样写没有问题，但如果是设置childDivider为@null，运行时点击父项会报错：

![img](/images/20190419/4.png)

错误信息下面还有，太长了就不截图了，所以如果设置childDivider为@null，运行时点击父项会报空指针错误，如果设置divider为@null的时候，运行时不会报错，但是会出现父列表项显示不全的问题：

![img](/images/20190419/5.png)

如图所见，本来不应该出现滚动条，手机屏幕是足以将整个列表项显示完的。`所以，如果需要使分割线消失的话，不要将这两个属性设置为@null，把它们的颜色设置为与背景颜色相同的颜色即可`。

还有一点，divider和childDivider之间并不会相互覆盖，比如设置了divider但没有设置childDivider，不会使childDivider的样式与divider的样式相同，而是会使用默认的样式，反之亦然

# dividerHeight
---
用于设置分割线的高度

# groupIndicator
---
这个属性是用于控制父项前面的小箭头用的，如果想要用自己的图片替换掉那个箭头可以这样写：
```xml
android:groupIndicator="@drawable/picture"
```
在Android Studio中用drawable和mipmap都可以

如果想要去掉那个小箭头，可以写：
```xml
android:groupIndicator="@null"
```

# indicatorLeft
---
箭头或者自己设置的图片的右边框距离手机左边边缘的距离，类似于marginLeft

# indicatorStart
---
箭头或者自己设置的图片的左边框距离手机左边边缘的距离，类似于marginLeft

> `以上两个属性相结合可以使箭头或自己设置的图片变大变小或者翻转，因为控制了箭头或图片的左右边框`

# indicatorRight和indicatorEnd
---
暂时还用不上，因为箭头一般在列表的左边，使用方法应该和上面两个方法差不多

# childIndicator
---
用于设置子项前显示的图标，不设置的话默认是没有图标的

# childIndicatorStart、childIndicatorLeft、childIndicatorEnd、childIndicatorRight
---
用法见indicatorStart、indicatorLeft、indicatorEnd、indicatorRight

关于父项和子项前面的图标这个问题，觉得还是在控件中设置为不可见，然后在父项和子项的布局中再加上会比较好，因为父项和子项的布局是没有考虑前面的图标，按一个完整的布局来考虑的，这样就会出现遮盖住图标的情况。如果想要有图标的话，子项设置比较简单，父项因为下拉收起列表的时候图标需要有变化（上下翻转），所以需要给父项的点击设置一个监听器，除了拉下列表，还有图标要旋转180度。