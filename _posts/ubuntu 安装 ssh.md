ubuntu 安装 ssh

1. 首先在服务器上安装ssh的服务器端。
> sudo aptitude install openssh-server
2. 检查ssh是否启动
> ps -e |grep ssh
如果只有ssh-agent那ssh-server还没有启动，需要/etc/init.d/ssh start，如果看到sshd那说明ssh-server已经启动了。
> netstat -tlp
> tcp6 0 0 *:ssh *:* LISTEN -
看到上面这一行输出说明ssh-server已经在运行了。
4. 在客户端通过ssh登录服务器。

ssh-server配置文件位于/ etc/ssh/sshd_config，在这里可以定义SSH的服务端口，默认端口是22，你可以自己定义成其他端口号，如222。然后重启SSH服务
> /etc/init.d/ssh restart

提高登录速度
找到 GSSAPI options 这一节，将下面两行注释掉：
#GSSAPIAuthentication yes
#GSSAPIDelegateCredentials no
然后重新启动 ssh 服务即可