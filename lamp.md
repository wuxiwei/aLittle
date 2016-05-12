##Linux安装LAMP开发环境及配置文件管理
>Linux主要分为两大发行版，分别是RedHat和Debian，lamp环境的安装和配置也会有所不同，所以分别以CentOS 7.0和Ubuntu 14.04做为主机（L）

###安装lamp开发环境方法
1. 通过源码安装
2. 通过软件源安装
3. 通过集成lamp软件包直接安装
####通过源码安装
源码安装的好处在于，更加灵活，可自定义安装插件，位置等。
####通过软件源安装
#####安装并配置Apache
* CentOS 7.0 通过yum方式安装
`[root@localhost ~]# yum install httpd`
默认安装的Apache版本为httpd-2.4.6-31.el7.centos.x86_64，默认配置文件主目录位于/etc/httpd下，/etc/httpd/conf/httpd.conf是Apache的主配置文件，Apache模块位于/usr/lib64/httpd/modules目录下，Apache模块的配置文件位于/etc/httpd/conf.modules.d目录下，Web根目录位于/var/www/html目录下，日志文件位于/var/log/httpd目录下。  
重点关注/etc/httpd目录。这个目录下，有个conf.d目录，默认情况下/etc/httpd/conf.d目录下所有的“.conf”结尾的文件都会被读取。因此，很多情况下不需要修改主配置文件/etc/httpd/conf/httpd.conf，而是在/etc/httpd/conf.d目录下新建一个以“.conf”结尾的文件来完成各种配置。



