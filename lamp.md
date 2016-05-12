##Linux安装LAMP开发环境及配置文件管理
>Linux主要分为两大系发行版，分别是RedHat和Debian，lamp环境的安装和配置也会有所不同，所以分别以CentOS 7.0和Ubuntu 14.04做为主机（L）  
>Linux下安装软件，最常见有源码安装方式、RPM/deb安装方式、yum/apt-get安装方式等，在这里使用yum/apt-get安装LAMP开发环境

####CentOS 7.0 下安装LAMP开发环境及配置文件管理

#####安装并配置Apache

`[root@localhost ~]# yum install httpd`

默认安装的Apache版本为httpd-2.4.6-31.el7.centos.x86_64，默认配置文件主目录位于/etc/httpd下，/etc/httpd/conf/httpd.conf是Apache的主配置文件，Apache模块位于/usr/lib64/httpd/modules目录下，Apache模块的配置文件位于/etc/httpd/conf.modules.d目录下，设置禁用或开启模块可以通过修改该目录下每个文件，Web根目录位于/var/www/html目录下，日志文件位于/var/log/httpd目录下。

重点关注/etc/httpd目录。这个目录下，有个conf.d目录，默认情况下/etc/httpd/conf.d目录下所有的“.conf”结尾的文件都会被读取。因此，很多情况下不需要修改主配置文件/etc/httpd/conf/httpd.conf，而是在/etc/httpd/conf.d目录下新建一个以“.conf”结尾的文件来完成各种配置。

将Apache设置为开机自启动模式

`[root@localhost wuxiwei]# systemctl enable httpd`

关闭Apache服务

`[root@localhost wuxiwei]# systemctl stop httpd`

开启Apache服务

`[root@localhost wuxiwei]# systemctl start httpd` 

重新加载httpd

`[root@localhost wuxiwei]# systemctl reload httpd`

#####安装并配置MariaDB（MYSQL）

`[root@localhost wuxiei]# yum install mariadb-server mariadb`

MariaDB完全兼容MYSQL，包括API和命令行。CentOS 从7.x开始默认使用MariaDB。

通过内置的安全配置脚本可实现对数据库的安全保护

`[root@localhost wuxiwei]# /usr/bin/mysql_secure_installation`

将MariaDB设置为开机启动

`[root@localhost wuxiwei]# systemctl enable mariadb`

开启MariaDB服务

`[root@localhost wuxiwei]# systemctl start mariadb`

关闭MariaDB服务

`[root@localhost wuxiwei]# systemctl stop mariadb`

#####安装并配置PHP

`[root@localhost wuxiei]# yum install php php-cli php-pear php-pdo php-mysqlnd php-gd php-mbstring php-mcrypt php-xml`

CentOS 7.1版本中，默认安装PHP为PHP5.4版本，其中php-mysqlnd是PHP源码提供的MYSQL驱动数据库。

很多时候会对PHP环境要求校新的版本，例如PHP5.6环境，记录一种通过yum工具安装最新PHP版本的方法。首先，需要在系统上安装一个扩展yum源，即epel源。可从http://fedoraproject.org/wiki/EPEL 网站下载并安装

`[root@localhost wuxiwei]# wget http://mirrors.neusoft.edu.cn/epel/7/x86_64/e/epel-release-7-5.noarch.rpm`
`[root@localhost wuxiwei]# rpm -ivh epel-release-7-5.noarch.rpm`

####Ubuntu 14.04 下安装LAMP开发环境及配置文件管理

#####安装并配置Apache

`[root@localhost wuxiwei]# apt-get install apache2`

重启Apache服务

`[root@localhost wuxiwei]# service apache2 restart`

#####安装并配置PHP5

`[root@localhost wuxiwei]# apt-get install php5`

查看Apache是否已经正确配置PHP5

`[root@localhost wuxiwei]# cat /etc/apache2/mods_enables/libphp5.so`

安装PHP5常用扩展

`[root@localhost wuxiwei]# apt-get install php5-gd curl libcurl3 libcurl3-dev php5-curl`

#####安装并配置MYSQL

`[root@localhost wuxiwei]# apt-get install mysql-server`

查看PHP5和MYSQL是否可以正常数据交互

`[root@localhost wuxiwei]# cat /etc/php5.d/conf.d/mysql.ini`

手动安装PHP5对于MYSQL扩展

`[root@localhost wuxiwei]# apt-get install php5-mysql`

重启MYSQL服务

`[root@localhost wuxiwei]# service mysql restart`

#####配置文件管理

Apache配置文件位于/etc/apache2目录下，Apache加载配置首先加载/etc/apache2/apache2.conf文件，通过Include将其他配置文件载入，核心配置文件包括：mods-*** Apache模块；sites-*** 虚拟主机，其中关键词available表示可以使用的；enable表示已启用的，两者通过ln -s命令建立软连接。

PHP5配置文件位于/etc/php5目录下，核心配置文件php.ini。

MYSQL配置文件位于/etc/mysql目录下，核心配置文件my.cnf，默认数据库存储位于/var/lin/mysql目录下。

***

####Apache虚拟主机配置
* CentOS 7.0 在/etc/httpd/conf.d目录下新建wuxiwei.conf文件，并重启apache。
* Ubuntu 14.04 在/etc/apache2/sites-available目录下新建wuxiwei.conf文件，同时在/etc/apache2/sites-enabled目录下创建软链接到wuxiwei.conf文件，并重启apache。
* wuxiwei.conf文件内容基本如下。
```
<VirtualHost *:80>
#管理员邮箱
ServerAdmin admin@wuxiwei.cn
#访问的主机名
ServerName wuxiwei.cn
#ServerName别名，通过别名也可以访问这个虚拟主机
ServerAlias www.wuxiwei.cn
#主机目录
DocumentRoot /var/www/html/wuxiwei

<Directory "/var/www/html/wuxiwei/">
#指定该目录启用FollowSymLinks特性，None：表示不起用任何的服务器特性，Indexes：如果输入的网址对应服务器上的一个目录，而此目录中又没有Directorylndex指令（例如：Directorylndex index.php index.html），那么服务器就会返回由mod_autoindex模块生成的一个格式化后的目录列表，并列出该目录下所有的文件。
    Options FollowSymLinks
    #允许所有都可以访问
    AllowOverride All
    Require all granted
</Directory>

#错误日志
Errorlog /var/log/httpd/wuxiwei_error.log
#访问日志
CustomLog /vat/log/httpd/wuxiwei_access.log combined
</VirtualHost>
```
