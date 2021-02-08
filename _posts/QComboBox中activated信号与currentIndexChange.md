# QComboBox中activated信号与currentIndexChanged信号的区别
在使用中可能会有类似的需求，改变ComboBox的当前选中值时发出消息信号，但是QComboBox提供了两小类信号，它们有什么区别呢？

activated(int)
This signal is sent when the user chooses an item in the combobox. The item’s index is passed. Note that this signal is sent even when the choice is not changed.

翻译：当用户选则ComboBox中的Item时，将会产生一个选中Item序号值的消息信号，不管选项是够改变都会产生该消息。
activated(QString)
This signal is sent when the user chooses an item in the combobox. The item’s text is passed. Note that this signal is sent even when the choice is not changed.

翻译：同理，只不过传送的是选中Item的ItemName。
currentIndexChanged (int)
This signal is sent whenever the currentIndex in the combobox changes either through user interaction or programmatically. The item’s index is passed or -1 if the combobox becomes empty or the currentIndex was reset.

翻译：当前ComboBox中的CurrentIndex改变时，不管是交互式还是通过程序改变选项，都会产生一个改变值得消息信号，如果ComboBox为空或重置当前ComboBox，产生的消息值返回-1。
currentIndexChanged (QString)
This signal is sent whenever the currentIndex in the combobox changes either through user interaction or programmatically. The item’s text is passed.

翻译：翻译：同理，只不过传送的是选中Item的ItemName
重点是currentIndexChanged不管是交互式（通过鼠标点击切换）还是通过程序改变选项，都会产生一个改变值的消息信号，以下是测试代码：

```c++
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QDebug>

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    ui->comboBox->addItem("111");
    ui->comboBox->addItem("222");
    ui->comboBox->addItem("333");
    ui->comboBox->addItem("444");
    connect(ui->comboBox, SIGNAL(activated(int)), this, SLOT(CheckActiveSignal()));
    connect(ui->comboBox, SIGNAL(currentIndexChanged(int)), this, SLOT(CheckIndexChangedSignal()));
    connect(ui->pushButton, SIGNAL(clicked()), this, SLOT(ComboboxSelectedPlus()));
}
MainWindow::~MainWindow()
{
    delete ui;
}
void MainWindow::ComboboxSelectedPlus()
{
	//此处绑定按钮的动作，点击按钮时，设置当前选中ItemNum增加1，
	//改变currentIndex又会触发CheckIndexChangedSignal函数，最前边有Connect消息。
    ui->comboBox->setCurrentIndex(ui->comboBox->currentIndex()+1);
}
void MainWindow::CheckActiveSignal()
{
	//通过鼠标切换选项会进入该函数
	//通过按钮改变currentIndex不会进入该函数
    int a = 0;
}
void MainWindow::CheckIndexChangedSignal()
{
	//通过鼠标切换选项会进入该函数
	//通过按钮改变currentIndex会进入该函数
    int a = 0;
}
//同时注意到有个小的细节，Connect的信号总是会先发currentIndexChanged，后发activated 
```