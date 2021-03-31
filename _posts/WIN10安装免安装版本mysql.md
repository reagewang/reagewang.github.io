# 免安装版mysql-8.0.23-winx64.zip
解压缩到c:\mysql-8.0.23-winx64目录或自定义目录，不要使用中文目录或带空格目录

在目录下创建一个data文件夹，用来存放数据库文件

创建一个my.ini配置文件

```ini
[mysqld]
# 设置端口3306（mysql的默认端口为3306）
port=3306
# 设置mysql的安装目录
basedir=c:\mysql-8.0.23-winx64
# 设置mysql数据库的数据的存放目录
datadir=c:\mysql-8.0.23-winx64\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。
max_connect_errors=10
# 设置mysql服务器使用的字符集，默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

以管理员的身份开启CMD命令窗口,进入c:\mysql-8.0.23-winx64\bin目录

初始化mysql,这是一个关键的操作步骤。初始化过程中会产生一个临时的密码，需要记住该密码
> mysqld --initialize --user=mysql --console

在CMD的窗口中输入下面的命令,将mysql安装为windows的服务
> mysqld -install

启动服务
> net start mysql

登入mysql,这里会出现一个密码输入的界面，这里的密码就是上面分配的临时密码
> mysql -u root -p

在mysql的提示符下输入下面的语句修改密码
> ALTER USER root@localhost IDENTIFIED BY '123456'

如果需要随时调用mysql的命令,可以配置mysql的环境变量。右键点击【此电脑】-【属性】-【高级系统设置】-【环境变量】,新建一个MYSQL_PATH变量指向c:\mysql-8.0.23-winx64,接下来在path中添加%MYSQL_PATH%\bin

# MySQL登录时出现Access denied for user ‘root‘@‘localhost‘ (using password: YES)无法打开的解决方法
出现Access denied的原因有如下可能：
1. MySQL的服务器停止了
2. 用户的端口号或者IP导致拒绝访问
3. MySQL的配置文件错误（my.ini等文件）
4. root用户的密码错误

针对root用户密码错误解决方法,用 –init-file 参数在服务启动时加载并运行修改密码的命令文件,该命令一旦执行,服务启动后密码即已经清除或者重置,启动服务后即可以空密码或指定密码登入

先关掉服务器
> net stop mysql

在MySQL的目录下创建一个文本文件mysqlc.txt,内含一条密码修改命令
> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456'

命令行方式启动服务器,指定启动时执行上述的密码修改命令文件
> mysqld --init-file=c:\mysql-8.0.23-winx64\mysqlc.txt --console

重启服务器
> net start mysql

登入mysql
> mysql -u root -p

# Host is not allowed to connect to this MySQL server解决方法
先说说这个错误，其实就是我们的MySQL不允许远程登录，所以远程登录失败了，解决方法如下：

在装有MySQL的机器上登录

mysql -u root -p

执行use mysql;

执行update user set host = '%' where user = 'root';这一句执行完可能会报错，不用管它。

执行FLUSH PRIVILEGES;

经过上面4步，就可以解决这个问题了。

注: 第四步是刷新MySQL的权限相关表，一定不要忘了，我第一次的时候没有执行第四步，结果一直不成功，最后才找到这个原因。