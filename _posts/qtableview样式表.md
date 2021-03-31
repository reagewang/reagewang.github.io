```css
QTableView QTableCornerButton::section 
{   /*左上角的button样式*/ 
    color: white;     
    background-color: skyblue;     /*背景颜色与表头一致*/ 
    border: 1px solid white;     
    border-radius:0px;     
    border-color: rgb(64, 64, 64);  
}   
QTableView 
{     
    color: white;                                       /*表格内文字颜色*/    
    gridline-color: white;                              /*表格内框颜色*/     
    background-color: rgb(27, 27, 27);               /*表格内背景色*/     
    alternate-background-color: rgb(64, 64, 64);     
    selection-color: white;                             /*选中区域的文字颜色*/     
    selection-background-color: rgb(77, 77, 77);        /*选中区域的背景色*/     
    border: 1px white;     /*边框像素和颜色*/   
    border-radius: 0px;      /*边框圆角半径*/   
    padding: 1px 1px; 
}  
QHeaderView 
{       /*表头样式*/ 
    color: white;       
    font: 16pt "Cantarell";
    background-color: skyblue;       
    border:0px solid rgb(191,191,191);     
    border-left-color: rgba(255, 255, 255, 0);     
    border-top-color: rgba(255, 255, 255, 0);     
    border-radius:0px;     
    min-height:30px; 
}  
QHeaderView::section 
{     /*表头分段样式*/ 
    color: white;     
    background-color: skyblue;     
    border: 1px solid white;     
    border-radius:0px; 
}
```