##Linux安装LAMP开发环境及配置文件管理
>Linux主要分为两大发行版，分别是RedHat和Debian，lamp环境的安装和配置也会有所不同，所以分别以CentOS 7.0和Ubuntu 14.04做为主机（L）

###安装lamp开发环境方法
1. 通过源码安装
2. 通过软件源安装
3. 通过集成lamp软件包直接安装

####通过源码安装
源码安装的好处在于，更加灵活，可自定义安装插件，位置等。
####通过软件源安装
#####1.安装并配置Apache
* CentOS 7.0 通过yum方式安装

`[root@localhost ~]# yum install httpd`

默认安装的Apache版本为httpd-2.4.6-31.el7.centos.x86_64，默认配置文件主目录位于/etc/httpd下，/etc/httpd/conf/httpd.conf是Apache的主配置文件，Apache模块位于/usr/lib64/httpd/modules目录下，Apache模块的配置文件位于/etc/httpd/conf.modules.d目录下，设置禁用或开启模块可以通过修改该目录下每个文件，Web根目录位于/var/www/html目录下，日志文件位于/var/log/httpd目录下。

重点关注/etc/httpd目录。这个目录下，有个conf.d目录，默认情况下/etc/httpd/conf.d目录下所有的“.conf”结尾的文件都会被读取。因此，很多情况下不需要修改主配置文件/etc/httpd/conf/httpd.conf，而是在/etc/httpd/conf.d目录下新建一个以“.conf”结尾的文件来完成各种配置。

将Apache设置为开机自启动模式

`[root@localhost wuxiwei]# systemctl enable httpd`

将关闭Apache服务

`[root@localhost wuxiwei]# systemctl stop httpd`

将开启Apache服务

`[root@localhost wuxiwei]# systemctl start httpd`

重新加载httpd

`[root@localhost wuxiwei]# systemctl reload httpd`

* Ubuntu 14.04 通过apt-get方式安装

***

实现独立域名访问，可通过配置Apache的虚拟主机访问来实现，在/etc/httpd/conf.d目录下新建一个文件wuxiwei.conf，内容如下。
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
