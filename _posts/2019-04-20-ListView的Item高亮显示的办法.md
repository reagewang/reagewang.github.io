---
layout: post
title: "ListView的Item高亮显示的办法"
tags: [Android, ListView]
comments: true
---

在我们使用ListView的时候，经常会遇到某一项（Item）需要高亮显示的情况，但有时候业务逻辑需要让item根据条件自动变亮，

自定义的适配器MyCustomAdapter 用来继承BaseAdapter,注意最后的setSelectItem方法是关键
```java
public class MyCustomAdapter extends BaseAdapter {
    ...
    @Override
	public View getView(int position, View convertView, ViewGroup parent) {
        ...
        if (position == selectItem) {   
            convertView.setBackgroundColor(Color.CYAN);   
        }    
        else {   
            convertView.setBackgroundColor(Color.TRANSPARENT);   
        }
				
		return convertView;
    }

    private int  selectItem = -1;
    public  void setSelectItem(int selectItem) {
        this.selectItem = selectItem;   
    }       
}
```
调用
```java
customAdapter.setSelectItem(CURRENT_POSITION);						
customAdapter.notifyDataSetInvalidated();
```

我们可以看到setSelectItem是我们第二步自定义适配器里面的方法，用于获得当前的选中的Item项，然后接着调用notifyDataSetInvalidated();就行了，有人可能会发现此处不是用的notifyDataSetChanged()，的确这里我们需要的是对控件改变进行通知，而不是对其中的内容发生改变通知，详细可以了解notifyDataSetInvalidated()与notifyDataSetChanged()的相同不同点。