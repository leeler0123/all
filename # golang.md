# golang 的安装（contOS 64位）


1.  在https://golang.org/dl/  找到适合自己源码地址链接，我的是contos 64位的
2.  cd /usr/local/    进入安装目录
3.  wget https://dl.google.com/go/go1.10.3.linux-amd64.tar.gz  下载
4.  tar -zxvf go1.10.3.linux-amd64.tar.gz  解压，目录会多出一个go/
5.  vim /etc/profile  编辑环境变量
6.  在底部增加go的执行目录，  代码  export PATH=$PATH:/usr/local/go/bin
7： source /etc/profile  使配置文件生效
8： echo $PATH   如果出现go/bin的目录代表配置成功
9   go version   如果出现版本号代表安装成功
