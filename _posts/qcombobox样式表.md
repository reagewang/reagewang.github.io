```
QComboBox
{
	border-radius:3px;
	background-color:rgba(60,60,60,200);
	font: 75 9pt "Microsoft YaHei";
    color:rgb(255,255,255);
    border:0px ;
	padding-top: 2px;
	padding-left: 2px;
}
QComboBox:disabled
{
	background-color:rgba(50,50,50,200);
	font: 75 9pt "Microsoft YaHei";
    color:rgb(160,160,160);
}
QComboBox:hover
{
	background-color:rgba(45,45,45,200);
	border:1px solid rgb(31,156,220) ;
}
/*点击combox的样式*/
QComboBox:on
{
	border-radius:3px;
	background-color:rgba(35,35,35);
	font: 75 9pt "Microsoft YaHei";
    color:rgb(255,255,255);
    border:1px solid rgb(31,156,220) ;
}
/*下拉框的样式*/
QComboBox QAbstractItemView 
{
    outline: 0px solid gray;  /*取消选中虚线*/
    border: 1px solid rgb(31,156,220);  
    font: 75 9pt "Microsoft YaHei";
    color: rgb(255,255,255);
    background-color: rgb(45,45,45);   
    selection-background-color: rgb(90,90,90);   
}
 /*选中每一项高度*/
QComboBox QAbstractItemView::item
 { 
	height: 25px;  
 }
/*选中每一项的字体颜色和背景颜色*/
QComboBox QAbstractItemView::item:selected 
{
    color: rgb(31,163,246);
	background-color: rgb(90,90,90); 
}
 /*下拉箭头的边框*/
QComboBox::drop-down 
{
	border:none;
}
 /*下拉箭头样式*/
QComboBox::down-arrow 
{
    right:10px;/*控制箭头左右偏移*/
    width: 9px;  
    height: 9px;   
    image: url(:/Resource/spinbox/combox_up.png);
}
 /*下拉箭头点击样式*/
QComboBox::down-arrow:on
{
    width: 9px;  
    height: 9px;   
    image: url(:/Resource/spinbox/combox_down.png);
}
```