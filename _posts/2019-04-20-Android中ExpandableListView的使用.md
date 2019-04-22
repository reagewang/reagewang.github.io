---
layout: post
title: "Android中ExpandableListView的使用"
tags: [Android, ExpandableListView]
comments: true
---

ExpandableListView是可扩展的下拉列表，它的可扩展性在于点击父item可以拉下或收起列表，适用于一些场景的使用，下面介绍的是在Activity中如何使用

# 下面介绍它的基本使用方法
---

![img](/images/20190420/1.gif)

# 最基本的使用
---
新建一个布局文件expandable_layout.xml，内容很简单，一个LinearLayout里面包含了一个ExpandableListView，别忘了给它加上id：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <ExpandableListView
        android:id="@+id/expandablelistview"
        android:layout_margin="5dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```
然后在activity文件中引入这个layout，获取这个ExpandableListView：
```java
...
private ExpandableListView listview;
...
setContentView(R.layout.expandable_layout);
listview = (ExpandableListView) findViewById(R.id.expandablelistview);
...
```
为了给ExpandableListView提供数据，需要先初始化数据，这里使用一个Map来存放数据，类型为<String, List<String>>：
```java
...
private Map<String, List<String>> dataset = new HashMap<>();
private String[] parentList = new String[]{"first", "second", "third"};
private List<String> childrenList1 = new ArrayList<>();
private List<String> childrenList2 = new ArrayList<>();
private List<String> childrenList3 = new ArrayList<>();
...
private void initialData() {
    childrenList1.add(parentList[0] + "-" + "first");
    childrenList1.add(parentList[0] + "-" + "second");
    childrenList1.add(parentList[0] + "-" + "third");
    childrenList2.add(parentList[1] + "-" + "first");
    childrenList2.add(parentList[1] + "-" + "second");
    childrenList2.add(parentList[1] + "-" + "third");
    childrenList3.add(parentList[2] + "-" + "first");
    childrenList3.add(parentList[2] + "-" + "second");
    childrenList3.add(parentList[2] + "-" + "third");
    dataset.put(parentList[0], childrenList1);
    dataset.put(parentList[1], childrenList2);
    dataset.put(parentList[2], childrenList3);
}
```
然后需要自己实现一个Adapte类，用于为ExpandableListView提供数据，该类继承了BaseExpandableListAdapter，下面这个是最简单的自定义的类：
```java
private class MyExpandableListViewAdapter extends BaseExpandableListAdapter {
 
    //  获得某个父项的某个子项
    @Override
    public Object getChild(int parentPos, int childPos) {
        return dataset.get(parentList[parentPos]).get(childPos);
    }
 
    //  获得父项的数量
    @Override
    public int getGroupCount() {
        return dataset.size();
    }
 
    //  获得某个父项的子项数目
    @Override
    public int getChildrenCount(int parentPos) {
        return dataset.get(parentList[parentPos]).size();
    }
 
    //  获得某个父项
    @Override
    public Object getGroup(int parentPos) {
        return dataset.get(parentList[parentPos]);
    }
 
    //  获得某个父项的id
    @Override
    public long getGroupId(int parentPos) {
        return parentPos;
    }
 
    //  获得某个父项的某个子项的id
    @Override
    public long getChildId(int parentPos, int childPos) {
        return childPos;
    }
 
    //  按函数的名字来理解应该是是否具有稳定的id，这个方法目前一直都是返回false，没有去改动过
    @Override
    public boolean hasStableIds() {
        return false;
    }
 
    //  获得父项显示的view
    @Override
    public View getGroupView(int parentPos, boolean b, View view, ViewGroup viewGroup) {
        return view;
    }
 
    //  获得子项显示的view
    @Override
    public View getChildView(int parentPos, int childPos, boolean b, View view, ViewGroup viewGroup) {
        return view;
    }
 
    //  子项是否可选中，如果需要设置子项的点击事件，需要返回true
    @Override
    public boolean isChildSelectable(int i, int i1) {
        return false;
    }
 }
 ```
 其中注释说明了每个方法的作用，这个adapter的所有数据来源都是刚刚初始化的dataset，因此当这个adapter需要返回父项的数目时，返回的就是dataset的大小，如果需要返回某个父项的某个子项时，通过父项的position，使用map的get方法即可获得。自定义的类中最重要的是下面这两个方法，下面先介绍getGroupView方法：
 ```java
 //  获得父项显示的view
@Override
public View getGroupView(int parentPos, boolean b, View view, ViewGroup viewGroup) {
    if (view == null) {
        LayoutInflater inflater = (LayoutInflater) ExpandableListViewTestActivity
                .this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        view = inflater.inflate(R.layout.parent_item, null);
    }
    view.setTag(R.layout.parent_item, parentPos);
    view.setTag(R.layout.child_item, -1);
    TextView text = (TextView) view.findViewById(R.id.parent_title);
    text.setText(parentList[parentPos]);
    return view;
}
```
这个方法用来指定父项显示的样式，内容和行为，其中使用了parent_item布局：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/parent_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:textColor="@color/black"
        android:textStyle="bold"
        android:text="这是父item"
        android:layout_margin="5dp"/>
</LinearLayout>
```
布局很简单，只是一个LinearLayout包含了一个TextView。getGroupView方法先判断view是否为空，如果view不为空，说明已经加载过一次parent_item布局，因此不需要重复加载以提高效率。如果view为空，那么使用如下方法加载parent_item布局：
```java
if (view == null) {
    LayoutInflater inflater = (LayoutInflater) ExpandableListViewTestActivity.this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    view = inflater.inflate(R.layout.parent_item, null);
}
```
然后定义父项的内容和行为，这里只需要定义父项的内容，先通过id获得parent_item布局中的TextView，然后通过方法提供的parentPos参数获取到存放在dataset中的内容，调用TextView的setText方法即可设置父项要显示的内容：
```java
TextView text = (TextView) view.findViewById(R.id.parent_title);
text.setText(parentList[parentPos]);
```
然后将view返回即可。接下来是getChildView方法：
```java
//  获得子项显示的view
@Override
public View getChildView(int parentPos, int childPos, boolean b, View view, ViewGroup viewGroup) {
    if (view == null) {
        LayoutInflater inflater = (LayoutInflater) ExpandableListViewTestActivity
                .this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        view = inflater.inflate(R.layout.child_item, null);
    }
    view.setTag(R.layout.parent_item, parentPos);
    view.setTag(R.layout.child_item, childPos);
    TextView text = (TextView) view.findViewById(R.id.child_title);
    text.setText(dataset.get(parentList[parentPos]).get(childPos));
    text.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Toast.makeText(ExpandableListViewTestActivity.this, "点到了内置的textview", Toast.LENGTH_SHORT).show();
        }
    });
    return view;
}
```
这个方法用于定义子项的布局，内容和行为，同样需要一个child_item来指定子项的布局：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/child_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:textColor="@color/black"
        android:text="这是子item"
        android:layout_margin="5dp"/>
</LinearLayout>
```
关于view的处理和父项一样，这里在方法中还给子项显示内容的textview添加了一个点击的监听器。adapter自定义完之后，通过声明一个Adapter，实例化后调用setAdapter方法即可：
```java
private MyExpandableListViewAdapter adapter;
adapter = new MyExpandableListViewAdapter();
listview.setAdapter(adapter);
```
至此，最基本的ExpandableListView的使用就可以满足啦!

# 稍微复杂一点的使用方法
---
## 为子项添加点击的监听事件

![img](/images/20190420/2.gif)

只需要调用ExpandableListView的setOnChildClickListener方法即可：
```java
listview.setOnChildClickListener(new ExpandableListView.OnChildClickListener() {
    @Override
    public boolean onChildClick(ExpandableListView expandableListView, View view,
                                int parentPos, int childPos, long l) {
        Toast.makeText(ExpandableListViewTestActivity.this,
                dataset.get(parentList[parentPos]).get(childPos), Toast.LENGTH_SHORT).show();
        return true;
    }
});
```
这里在每个子项被点击了之后会显示是哪个子项被点击了。

`特别注意`
1. 在使用这个方法的时候需要将自定义的adapter中的isChildSelectable方法的返回值设置为true，否则子项的点击不生效，但子项布局中设置的控件的监听器依然可以生效。
```java
// 子项是否可选中，如果需要设置子项的点击事件，需要返回true
@Override
public boolean isChildSelectable(int i, int i1) {
    return true;
}
```
2. 如果在子项中对某个控件设置了监听器，这个控件要注意不能铺满整个子项，所以在设置高度和宽度时要特别注意，否则设置了子项的监听器也是没有用的

## 为子项添加长按的监听器
在ExpandableListView中并没有提供设置子项长按监听器的方法，多方查找之后找到了一个算是比较靠谱的方法，目前用起来暂时还没有什么问题。

ExpandableListView中关于长按有一个setOnItemLongClickListener方法，但是这个方法有个问题，没办法区分被长按的item是父项还是子项，所以需要在自定义adapter的getGroupView方法和getChildView方法中加一点东西来区分是父项还是子项：

完整的getGroupView方法和getChildView方法：
```java
//  获得父项显示的view
@Override
public View getGroupView(int parentPos, boolean b, View view, ViewGroup viewGroup) {
    if (view == null) {
        LayoutInflater inflater = (LayoutInflater) ExpandableListViewTestActivity
                .this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        view = inflater.inflate(R.layout.parent_item, null);
    }
    view.setTag(R.layout.parent_item, parentPos);
    view.setTag(R.layout.child_item, -1);
    TextView text = (TextView) view.findViewById(R.id.parent_title);
    text.setText(parentList[parentPos]);
    return view;
}

//  获得子项显示的view
@Override
public View getChildView(int parentPos, int childPos, boolean b, View view, ViewGroup viewGroup) {
    if (view == null) {
        LayoutInflater inflater = (LayoutInflater) ExpandableListViewTestActivity
                .this.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        view = inflater.inflate(R.layout.child_item, null);
    }
    view.setTag(R.layout.parent_item, parentPos);
    view.setTag(R.layout.child_item, childPos);
    TextView text = (TextView) view.findViewById(R.id.child_title);
    text.setText(dataset.get(parentList[parentPos]).get(childPos));
    text.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Toast.makeText(ExpandableListViewTestActivity.this, "点到了内置的textview",
                    Toast.LENGTH_SHORT).show();
        }
    });
    return view;
}
```
这里用到了view的setTag方法，一共设置了两个Tag，标签虽然在设置的时候提示说只要int类型即可，但一开始使用0和1来做tag的时候，显示没有报错，但编译运行就报错了，要求是资源文件的id才行，因此换成了R.layout.parent_item和R.layout.child_item。

如果是父项，就设置R.layout.parent_item为第几个父项，设置R.layout.child_item为-1。

如果是子项，就设置R.layout.parent_item属于第几个父项，设置R.layout.child_item为该父项的第几个子项，这样就可以区分被长按的是父项还是子项了。

然后设置ExpandableListView长按item的监听器：
```java
listview.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
    @Override
    public boolean onItemLongClick(AdapterView<?> adapterView, View view, int i, long l) {
        String content = "";
        if ((int) view.getTag(R.layout.child_item) == -1) {
            content = "父类第" + view.getTag(R.layout.parent_item) + "项" + "被长按了";
        } else {
            content = "父类第" + view.getTag(R.layout.parent_item) + "项" + "中的"
                    + "子类第" + view.getTag(R.layout.child_item) + "项" + "被长按了";
        }
        Toast.makeText(ExpandableListViewTestActivity.this, content, Toast.LENGTH_SHORT).show();
        return true;
    }
 });
 ```
 这样就可以区分父项和子项了。

 ## 更新数据
列表项更新数据一般使用的都是adapter的notifyDataSetChanged()方法，ExpandableListView也不例外，下面展示一下怎么更新数据。

首先要在原先的expandable_layout.xml文件中添加一个按钮，用来更新数据：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <ExpandableListView
        android:id="@+id/expandablelistview"
        android:layout_margin="5dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <Button
        android:id="@+id/updateData"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        android:layout_gravity="center"
        android:text="刷新数据"/>
</LinearLayout>
```
然后在Activity文件中引入这个Button：
```java
private Button button;
button = (Button) findViewById(R.id.updateData);
```
为button设置点击的监听器：
```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        updateData();
        Toast.makeText(ExpandableListViewTestActivity.this, "数据已更新", Toast.LENGTH_SHORT).show();
    }
});
```
其中updateData()方法就是用于更新ExpandableListView的方法，看一下具体的实现：
```java
/**
 * 更新数据
 */
private void updateData() {
    childrenList1.clear();
    childrenList1.add(parentList[0] + "-new-" + "first");
    childrenList1.add(parentList[0] + "-new-" + "second");
    childrenList1.add(parentList[0] + "-new-" + "third");
    childrenList2.clear();
    childrenList2.add(parentList[1] + "-new-" + "first");
    childrenList2.add(parentList[1] + "-new-" + "second");
    childrenList2.add(parentList[1] + "-new-" + "third");
    childrenList3.clear();
    childrenList3.add(parentList[2] + "-new-" + "first");
    childrenList3.add(parentList[2] + "-new-" + "second");
    childrenList3.add(parentList[2] + "-new-" + "third");
    adapter.notifyDataSetChanged();
}
```
还记得上面初始化数据的时候使用的childrenList1、childrenList2、childrenList3吗，这三个列表是我们用来放子项列表，然后已经添加到dataset里面去了。这个时候无需再用dataset添加一次，只需要将列表清空，添加上更新的内容即可。也可以根据自己的需求选择性地删除一些东西来获得新的子项列表。

`需要注意的是，子项列表在内存中的地址是不可以改变的，不能使用形如childrenList1 = new ArrayList<>();这样的方法来获的新列表，这样会使childrenList1在内存中的地址发生改变，导致调用adapter的notifyDataSetChanged()方法时不生效。所以如果想要清空这个列表项时，使用childrenList1.clear()方法，这样才能保证顺利更新列表。`

# 更新
---
首先是接口部分

![img](/images/20190420/3.png)

除了可以设置子类被点击的监听器外，还可以设置父类被点击的监听器，以及一个列表展开和收起的监听器

接下来看一下具体的方法

收起某一个列表，参数为父类第几项，如果是要收起第一个列表，那么groupPos = 0. 如果这个列表已经收起了，返回值为false，表示收起失败，因为列表已经收起了。如果这个列表还没有收起，那么收起这个列表，返回值为true。

![img](/images/20190420/4.png)

展开列表的用法和收起列表的用法一样，如果列表已经展开，返回false，如果列表还没有展开，返回true。

![img](/images/20190420/5.png)

关于列表的展开还有一个方法

![img](/images/20190420/6.png)

比上一个方法多了一个参数，如果把这个参数设置为true，列表展开的时候会有动画效果，该方法需在API大于等于14的时候才可以用于判断列表是否展开的方法

![img](/images/20190420/7.png)

列表已展开，返回true；列表未展开，返回false

# 这里讲一下为父类点击事件，列表收缩事件，列表展开事件设置监听器的方法
---
为父类的点击事件设置监听器
```java
listview.setOnGroupClickListener(new ExpandableListView.OnGroupClickListener() {
    @Override
    public boolean onGroupClick(ExpandableListView expandableListView, View view, int i, long l) {
        if (listview.isGroupExpanded(i)) {
            listview.collapseGroup(i);
        } else {
            listview.expandGroup(i, true);
        }
        return true;
    }
});
```
上面的代码实现的效果是当父类被点击时，判断列表是否展开，如果没有展开的话就展开列表，如果列表已经展开，那么收起列表

为列表收缩事件和展开事件设置监听器
```java
listview.setOnGroupCollapseListener(new ExpandableListView.OnGroupCollapseListener() {
    @Override
    public void onGroupCollapse(int i) {
        Toast.makeText(ExpandableListViewTestActivity.this, "第" + i + "个列表收缩了", Toast.LENGTH_SHORT).show();
    }
});
listview.setOnGroupExpandListener(new ExpandableListView.OnGroupExpandListener() {
    @Override
    public void onGroupExpand(int i) {
        Toast.makeText(ExpandableListViewTestActivity.this, "第" + i + "个列表伸展了", Toast.LENGTH_SHORT).show();
    }
});
```
如果需要进入的时候列表就展开，然后不再收起，可以这样设置：在setAdapter之后遍历每一个列表使它们展开
```java
for (int i = 0; i < parentList.length; i++) {
     if (!listview.isGroupExpanded(i)) {
        listview.expandGroup(i);
    }
}
```
然后设置父类的监听器直接返回true即可，不可以设置父类的监听器为null，那样起不到屏蔽原先系统设置的监听器的效果
```java
listview.setOnGroupClickListener(new ExpandableListView.OnGroupClickListener() {
    @Override
    public boolean onGroupClick(ExpandableListView expandableListView, View view, int i, long l) {
        return true;
    }
});
```

