---
layout: post
title: "为什么ListView的setSelection无效了"
tags: [Android, ListView]
comments: true
---

原因一：界面初始化完成之后listview失去了焦点。

原因二：因为listview的item高度不一致，或者添加了headerview，在setadapter之后调用setSelection无法准确定位。

万能解决方法：
```java
final ListView listView = new ListView(getActivity());
listView.post(new Runnable() {
    @Override
    public void run() {
        listView.requestFocusFromTouch();//获取焦点
         listView.setSelection(listView.getHeaderViewsCount()+10);//10是你需要定位的位置
    }
});
```
如果还不行:
```java
final ListView listView = new ListView(getActivity());
listView.postDelayed(new Runnable() {
    @Override
    public void run() {
        listView.requestFocusFromTouch();
        listView.setSelection(listView.getHeaderViewsCount()+10);
    }
},500);
```