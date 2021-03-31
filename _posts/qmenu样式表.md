```
/*Qmenu Style Sheets*/
QMenu {
    background-color: white; /* sets background of the menu 设置整个菜单区域的背景色，我用的是白色：white*/
    border: 1px solid white;/*整个菜单区域的边框粗细、样式、颜色*/
}

QMenu::item {
    /* sets background of menu item. set this to something non-transparent
        if you want menu color and menu item color to be different */
    background-color: transparent;
    padding:8px 32px;/*设置菜单项文字上下和左右的内边距，效果就是菜单中的条目左右上下有了间隔*/
    margin:0px 8px;/*设置菜单项的外边距*/
    border-bottom:1px solid #DBDBDB;/*为菜单项之间添加横线间隔*/
}

QMenu::item:selected { /* when user selects item using mouse or keyboard */
    background-color: #2dabf9;/*这一句是设置菜单项鼠标经过选中的样式*/
}
```
```
//去掉投影
m_pMenu->setWindowFlag(Qt::NoDropShadowWindowHint);

 QString menuQss = "QMenu{color:rgb(255, 255, 255);background:rgba(255,255,255,16);border:none;font:16px \"SimHei\";}";
 menuQss.append("QMenu::item{color:rgb(255, 255, 255);background:rgba(0, 0, 0,0);height:38px;padding-left:10px;} ");
 menuQss.append("QMenu::item:selected{color:rgb(255, 255, 255);background:rgb(37, 183, 152);}");
 menuQss.append("QMenu::icon{Top:1px;right:8px;bottom:0px;left:8px;}"); 
 m_pMenu->setStyleSheet(menuQss);
```