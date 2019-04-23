---
layout: post
title: "改变ListView中item选中时文字的颜色"
tags: [Android, ListView]
comments: true
---

当listview的某个item选中时，默认有个选中的高亮显示，如果你要自定义选中时的高亮显示效果，可以在listview中设置属性

```xml
android:listSelector="@drawable/item_selector"
```

其中`item_selector`是在drawable目录下定义的一个xml文件，这种用于突出不同状态下显示效果的xml文件我们称之为`selector`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
  xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="false" android:drawable="@*android:color/transparent" />
    <item android:state_pressed="true" android:drawable="@drawable/grid_item_select_bg" />
    <item android:state_selected="true" android:drawable="@drawable/grid_item_select_bg_night" />
</selector>
```

上面这个selector定义了三种状态下的显示效果。但是如果我们想在listview的某个item选中时改变该item的某个textview的文字颜色，上面的办法就行不通了。那该如何做呢？

其实如果我们真正了解android:listSelector的含义的话，很容易实现上面的需求。我发现如果不在listview中设置listSelector，也就是将android:listSelector="@drawable/item_selector"去掉，而把item 的background属性设为item_selector，会得到同样的选中高亮效果。由此可见listview可以将自己的状态（state_press、state_select、state_focus等）向内传递，当然item本身也可以将这些状态继续传递给子view。受此启发，我们可以将需要高亮显示文字颜色的TextView的textColor属性也设置成selector的形式

修改前item:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical"                
    >
    <TextView
        android:id="@+id/txt"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/hello"
        android:layout_margin="5dp"         
        />
</LinearLayout>
```

修改后item:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical"
    android:background="@drawable/item_selector" <!-- item背景色变换 -->
    >
    <TextView
        android:id="@+id/txt"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/hello"
        android:layout_margin="5dp"
       android:textColor="@drawable/item_text_selector" <!-- item文字颜色变换 -->
        />
</LinearLayout>
```

其中，item_text_selector.xml的源码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_focused="true" android:color="#333333" /> <!-- focused -->
    <item android:state_pressed="true" android:color="#333333" /> <!-- pressed -->
    <item android:state_selected="true" android:color="#333333" /> <!-- pressed -->
    <item android:color="#f4f4f4" /> <!-- default -->
</selector>
```

如果想更加可靠不妨给TextView 增加个属性

```xml
android:duplicateParentState="true"
```

表示会跟随ParentView的状态来变化，其实没加也不会有问题，因为默认状态本来就是能传递的，只是在某些极端的情况下可以设置这个属性做一层保险。
