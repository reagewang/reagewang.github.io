---
layout: post
title: "Android ListView滚动条配置完全解析"
tags: [ListView, scrollbar, Android]
comments: true
---

ListView的滚动条有哪些显示效果:
1. 滚动条自身的外观 
> 就是滚动条自身的颜色，形状等。
2. Track的外观 
> 默认的ListView是没有设置Track的。Track表示的是滚动条滑动时的”轨道”。
3. 滚动条的大小 
> ListView是垂直滚动条，它的大小就是滚动条的宽度。
4. 滚动条的显示位置 
> 滚动条出现在ListView左边，还是右边，以及是显示在内侧还是外侧。
5. 滚动条的Fade时间 
> 滚动条只有在滚动的时候才会显示，当停止滚动后，滚动条会在一段时间后渐渐消失。这里有两个时间点，一个是从停止滚动到开始消失的时间，一个是开始消失到完全消失的时间。

# 在XML中自定义ListView滚动条
---
自定义ListView滚动条可以直接在布局文件中对ListView进行配置。先看下ListView在XML中有哪些和滚动条相关的配置选项。 
* android:scrollbars 
* android:scrollbarThumbVertical 
* android:scrollbarTrackVertical 
* android:scrollbarSize 
* android:verticalScrollbarPosition 
* android:scrollbarStyle 
* android:fadeScrollbars 
* android:scrollbarDefaultDelayBeforeFade 
* android:scrollbarFadeDuration 
* android:scrollbarAlwaysDrawVerticalTrack 
* android:fastScrollEnabled 
* android:fastScrollStyle 
* android:fastScrollAlwaysVisible 
可以看到ListView中有非常多的和滚动条相关的配置选项。下面具体看下每个选项的含义及配置方法。
## android:scrollbars 
此选项表示是否显示滚动条，它的取值可以是vertical，horizontal或none。对ListView来说，它只能垂直滚动，将scrollbars设置成horizontal或者none效果都是一样的，也就是不会出现滚动条。所以如果不希望ListView显示滚动条，就将scrollbars设置成none。此外，如果scrollbars设置成none，那么其他的滚动条相关的配置都不会有效果。

## android:scrollbarThumbVertical 
此选项用来控制垂直滚动条的显示外观，这也是美化滚动条时最重要的一项配置。它可以设置为一个颜色值，或者是一个Drawable资源。对Drawable资源可以使用.9的png图片，也可以使用XML来配置。 例如下面这个xml就定义了一个Drawable资源，其内部是一个从绿色到红色的渐变色，四角有6dp的圆角，同时还有一个1dp的带有透明度的黑色边框。
```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <gradient
        android:angle="0"
        android:endColor="#FF0000"
        android:startColor="#00FF00" />
    <corners android:radius="6dp" />
    <stroke
        android:width="1dp"
        android:color="#A4000000" />
</shape>
```
将scrollbarThumbVertical设置为该Drawable资源后，显示效果如图所示。 

![img](/images/31/1.png)

## android:scrollbarTrackVertical 
此选项用来控制垂直滚动条背后滑动轨道的显示效果。和android:scrollbarThumbVertical配置一样，android:scrollbarTrackVertical可以设置为一个颜色值，或者是一个Drawable资源。 例如下面的xml，内部是一个#F1F0C1颜色的实心矩形，四角有6dp的圆角，同时还有一个1dp的带有透明度的黑色边框。
```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <solid
        android:color="#F1F0C1" />
    <corners android:radius="6dp" />
    <stroke
        android:width="1dp"
        android:color="#A2000000" />
</shape>
```
将scrollbarTrackVertical设置为该Drawable资源后，显示效果如图所示。 

![img](/images/31/2.png)

## android:scrollbarSize 
此选项表示滚动条的大小，对ListView来说就是滚动条的宽度。 例如设置android:scrollbarSize=”3dp”，表示滚动条宽度为3dp。显示效果如图所示。 

![img](/images/31/3.png)

关于此项配置需要注意的是，如果android:scrollbarThumbVertical配置的是一个.9.png的图片（自身有宽度的Drawable），那么android:scrollbarSize配置会被忽略，只有android:scrollbarThumbVertica配置的是颜色值或者xml时（自身宽度为0的Drawable），此项配置才会有效。下面看下Android SDK的源码，android.view.View类getVerticalScrollbarWidth()方法源码如下。
```java
public int getVerticalScrollbarWidth() {
    ScrollabilityCache cache = mScrollCache;
    if (cache != null) {
        ScrollBarDrawable scrollBar = cache.scrollBar;
        if (scrollBar != null) {
            int size = scrollBar.getSize(true);
            if (size <= 0) {
                size = cache.scrollBarSize;
            }
            return size;
        }
        return 0;
    }
    return 0;
}
```
mScrollCache.scrollBarSize就是这里配置的滚动条宽度。scrollBar是ScrollBarDrawable类的对象，其getSize()方法源码如下。
```java
public int getSize(boolean vertical) {
    if (vertical) {
        return mVerticalTrack != null ? mVerticalTrack.getIntrinsicWidth() : mVerticalThumb != null ? mVerticalThumb.getIntrinsicWidth() : 0;
    } else {
        return mHorizontalTrack != null ? mHorizontalTrack.getIntrinsicHeight() : mHorizontalThumb != null ? mHorizontalThumb.getIntrinsicHeight() : 0;
    }
}
```
mVerticalTrack，mVerticalThumb 就是android:scrollbarThumbVertical和android:scrollbarTrackVertical配置的Drawable对象，从这里可以看到，只有android:scrollbarThumbVertical配置的Drawable对象的getIntrinsicWidth()方法返回的是小于等于0的值，才会使用mScrollCache.scrollBarSize作为滚动条的宽度。关于Drawable对象的getIntrinsicWidth()方法何时返回大于0的值，何时返回小于等于0的值，这里不再探究，有兴趣的可以再继续翻源码。

## android:verticalScrollbarPosition 
此项配置用来设置滚动条的位置，它可以是right，left或者是defaultPosition。如果不设置，默认是defaultPosition。如果是defaultPosition，则滚动条的位置受到布局RTL配置的影响，如果布局是从右往左，则滚动条显示在left侧，如果布局是从左往右，则滚动条显示在right侧。 注意：滚动条没有上下位置的设置。对于可水平滚动的View（如HorizontalScrollView），滚动条始终在下方。不能设置到上方。

## android:scrollbarStyle 
此项配置也是用来设置滚动条的位置，不过不是左右位置，而是滚动条和ListView内容之间的相对位置，它的取值范围是insideoverlay，insideInset，outsideoverlay，outsideinset。 对一个View来说，它的内部可用区域是View自身大小再减去padding后的区域。
1. insideoverlay：表示滚动条右侧和ListView可用区域右侧对其，且覆盖在Item之上。
2. insideInset：表示滚动条右侧和ListView可用区域右侧对其，但不覆盖在Item之上，而是将Item挤到滚动条的左边。
3. outsideoverlay：表示滚动条右侧和ListView右侧对其，且覆盖在右侧padding之上。
4. outsideinset：表示滚动条右侧和ListView可用区域右侧对其，但不覆盖在padding之上，而是将padding挤到滚动条的左边。 

![img](/images/31/4.png)

## android:fadeScrollbars 
此项配置用来表示是否在ListView不滚动时隐藏滚动条，可以选择true或false，默认为true，也就是不滚动时隐藏。如果将其设置为false，那么只要ListView能够滚动，滚动条就会一直显示，不会隐藏。但如果ListView不足一页，不能滚动，则不会显示。此外，如果配置了android:scrollbarTrackVertical，也就是滚动条的Track，然后设置fadeScrollbars为不隐藏滚动条，那么不仅滚动条不会隐藏，滚动条的Track同样也不会隐藏。

## android:scrollbarDefaultDelayBeforeFade 
此项配置表示滚动条从显示到隐藏的间隔时间，单位为毫秒，如果不设置，默认为300毫秒。例如，设置android:scrollbarDefaultDelayBeforeFade=”1200”，表示停止滚动后1.2秒后开始隐藏滚动条。需要注意的是，ListView初次加载完成时，会自动显示出滚动条。这时如果没有去做滑动操作，滚动条也会自动隐藏，不过和滑动后隐藏不同的是，这里需要经过4倍间隔时间后才会开始隐藏。可以在Android SDK源码android.view.View类的initialAwakenScrollBars()方法中找到这块代码。

## android:scrollbarFadeDuration 
此项配置表示滚动条从滚动条开始隐藏到完全隐藏的间隔时间，单位为毫秒，如果不设置，默认为250毫秒。例如，设置android:scrollbarDefaultDelayBeforeFade=”2000”，表示开始隐藏滚动条后2秒后完全隐藏，中间是一个滚动条逐渐透明的过程。

## android:scrollbarAlwaysDrawVerticalTrack 
此项配置表示是否总是显示垂直滚动条的Track，可以选择true或false，默认为false。通常，如果设置了滚动条的Track，那么Track会随着滚动条一起显示和隐藏。但如果设置了android:scrollbarAlwaysDrawVerticalTrack为true，则滚动条的Track将一直显示，不会隐藏。当然，如果没有配置android:scrollbarTrackVertical，即使设置了android:scrollbarAlwaysDrawVerticalTrack为true，也不会有Track显示。此外，android:fadeScrollbars配置为false，则无论android:scrollbarAlwaysDrawVerticalTrack配置为true还是false，Track都不会隐藏。

## android:fastScrollEnabled 
此项配置表示是否启用快速滚动条，可以选择true或false，默认为false，也就是不使用快速滚动条。如果启用了快速滚动条，当ListView页数小于4页时，仍然会显示普通滚动条，当ListView页数超过4页时，才会显示快速滚动条。前面的10项配置除了android:verticalScrollbarPosition之外，都只对普通滚动条有效果，对快速滚动条没有影响。但android:verticalScrollbarPosition配置会影响快速滚动条，如果设置android:verticalScrollbarPosition为left，则快速滚动条也会显示在ListView的左侧。此外，当快速滚动条显示时，普通滚动条会自动隐藏，即使普通滚动条设置为不隐藏。当快速滚动条隐藏时，普通滚动条会自动显示，除非设置为不显示（android:scrollbars=”none”）（从Android 5.0开始，快速滚动条隐藏时，普通滚动条不再显示）。

## android:fastScrollStyle 
此项配置用来设置快速滚动条的样式，其可配置的值，及含义和android:scrollbarStyle一样。不过此项配置是从Android5.0才开始有的，所以只有Android5 .0以上的系统中，此项配置才会有效。

## android:fastScrollAlwaysVisible 
此项配置用来设置是否始终显示快速滚动条，可以选择true或false，默认为false，如果将其设置为true，则快速滚动条将始终显示，不会隐藏，且不受4页的约束，也就是说ListView即使不足4页，快速滚动条也会显示，甚至即使ListView不足一页，快速滚动条都会显示。此外，android:fadeScrollbars设置将不会生效，也就是即使android:fadeScrollbars设置为false，也不会显示普通滚动条。

# fastScroll的其他配置 
---
当启用了快速滚动条后，在布局文件中只提供了快速滚动条样式和是否始终可见的设置。并没有像普通滚动条一样的可以用自定义的Drawable来替代默认的图标的配置选项，也没有提供对Track的配置。要想修改快速滚动条的图标和Track，可以在ListView所在的Activity的theme中来配置（如果Activity没有配置Theme，则在Application的theme中配置）。theme中支持的配置项有
* android:fastScrollThumbDrawable
* android:fastScrollOverlayPosition
* android:fastScrollTextColor
* android:fastScrollTrackDrawable
* android:fastScrollPreviewBackgroundLeft
* android:fastScrollPreviewBackgroundRight
这些配置项都是从API 11，也就是Android 3.0开始支持的。其中fastScrollThumbDrawable和fastScrollTrackDrawable就是用来配置快速滚动条的图标及Track图标的。