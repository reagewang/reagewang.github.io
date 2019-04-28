---
layout: post
title: "Ubuntu下JAVA环境变量"
tags: [Ubuntu, java]
comments: true
---

配置JAVA环境变量

# 方法1：修改/etc/profile 文件
---
所有用户的 shell都有权使用这些环境变量
1. 在shell终端执行命令
> vi /etc/profile

2. 在 profile文件末尾加入：
JAVA_HOME=/usr/lib/jvm/java-6-sun-1.6.0.15
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar: $JAVA_HOME/lib/tools.jar
export JAVA_HOME,PATH,CLASSPATH

3. 重启系统

# 方法2：修改.bashrc文件
---
如果你需要给某个用户权限使用这些环境变量，你只需要修改其个人用户主目录下的.bashrc就可以了,而不像第一种方法给所有用户权限。
1. 在shell终端执行命令
> vi /home/username/.bashrc

2. 在.bashrc文件末尾加入：
set JAVA_HOME=/usr/lib/jvm/java-6-sun-1.6.0.15
export JAVA_HOME
set PATH=$JAVA_HOME/bin:$PATH
export PATH
set CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export CLASSPATH

3. 重新登录

# 方法3：直接在shell下修改
---
用于在Shell下临时使用，换个Shell即无效
export JAVA_HOME=/opt/jdk1.5.0_02
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
`注意：Linux使用:(冒号)而不是;(分号)来分隔路径`

# 测试环境配置
---
1. 终端下输入 `java -version`，然后输出显示，显示出来的是当前系统JRE的最高版本
`注意：成功只是说明运行环境成功，一般只要安装了JRE就OK`         
2. 终端下输入 `javac`，如果出现了相应提示，说明编译环境已经配置成功
`注意：成功说明运行环境配置成功，接下来就可以进行Java的基本编程了`

# 环境变量配置文件
---
在Ubuntu中有如下几个文件可以设置环境变量,几个环境变量的优先级1>2>3
1. /etc/profile
在登录时,操作系统定制用户环境时使用的第一个文件,此 文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。
2. /etc/environment
在登录时操作系统使用的第二个文件,系统在 读取你自己的profile前,设置环境文件的环境变量。
3. ~/.bash_profile
在登录时用到的第三个文件是.profile文 件,每个用户都可使用该文件输入专用于自己使用的shell信息,当用 户登录时,该 文件仅仅执行一次!默认情况下,他设置一些环境变游戏量,执 行用户的.bashrc文件。/etc/bashrc:为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该 文件被读取.
4. ~/.bashrc
该文件包含专用于你的bash shell的bash信 息,当登录时以及每次打开新的shell时,该该文件被读取。

# 设置永久环境变量
---
1. 环境变量配置中，要先删除.bash_profile中的三行关于.bashrc的 定义，然后把环境变量配置在.bashrc中
2. 选择要使用的java环境：update-alternatives –config java
3. 要使得刚修改的环境变量生效：source .bashrc
4. 查看环境变量：env
可以放到/etc/bash/bashrc，这样就是系统级的
