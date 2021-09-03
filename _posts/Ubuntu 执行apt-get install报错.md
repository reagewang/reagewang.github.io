Ubuntu 执行apt-get install报错

E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?

解决方案
方案一
sudo killall apt apt-get
如果提示没有apt进程
apt: no process found
apt-get: no process found

方案二：
依次执行
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock*
sudo dpkg --configure -a
sudo apt update
