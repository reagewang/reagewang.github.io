# nfs 共享目录

## 虚拟机安装服务
> sudo apt-get install nfs-kernel-server

## 重启
> sudo service nfs-kernel-server restart

## 停止
> sudo service nfs-kernel-server stop

## 增加共享目录

修改NFS配置文件/etc/exports，增加以下内容：/home/wh/nfsdir *(rw,sync,no_subtree_check,no_root_squash)
* /home/wh/nfsdir：为Ubuntu上的共享目录
* *：为允许访问的网段，此值也可以是ip地址、主机名、还可以为*，这样所有人都能访问
* rw：读/写
* sync：数据同步
* no_root_squash：不降低root用户权限
* no_subtree_check：关闭子树检查

chmod -R 777 /home/wh/nfsdir/
chown -R nobody /home/wh/nfsdir/

## 远端挂载目录
> mount -t nfs -o nolock 192.168.102.58:/home/wh/nfsdir  /opt/ipnc/qt/nfsdir

> umount /opt/ipnc/qt/nfsdir

> mount -t nfs -o nolock 192.168.102.58:/home/wh/nfsdir  /mnt/nfsdir

> umount /mnt/nfsdir

> mount 192.168.102.58:/home/wh/winnfsdir X:  