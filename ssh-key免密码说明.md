#### 使用ssh-key可以免密码登陆远程服务器或连接github
##### github连接过程
`$cd ~/.ssh`  
`$ssh-keygen -t rsa -C "wxw_it@163.com"`  
之后会让你输入密码，直接回车。然后就生成两个文件：id_rsa , id_rsa.pub。在GitHub上注册一个用户，然后进入SSH keys，把id_rsa.pub的内容复制进去保存即可。  
测试连接是否成功  
`$ssh -T git@github.com`  
`Hi wuxiwei! You've successfully authenticated, but GitHub does not provide shell access.`
##### 登陆远程服务器
`$ssh-keygen -t rsa`  
之后会让你输入密码，直接回车。  
ssh-copy-id方法简单快捷  
`$ssh-copy-id -i ~/.ssh/id_rsa.pub awy3ib44jg@121.42.27.147`  
然后重新启动启动终端  
`ssh awy3ib44jd@121.42.27.147`  
输入密码完成  
如果连接服务器如果报错：Agent admitted failure to sgin using the key.  
`$ssh-add`
##### 关于git的ssh-key解决本地多个ssh-key的问题
本地配置两个ssh-key，一个连接sever，一个连接github，如何解决冲突。  
为github配置新的key ，取名为github  
`$ ssh-keygen -t rsa -C "xxx@gmail.com" -f ~/.ssh/github`  
查看.ssh目录生成的文件  
`$ ls ~/.ssh`  
`github github.pub id_rsa id_rsa.pub known_hosts`  
其中默认的id_rsa是公司server用的，github.pub是刚刚生成连接github用的，主要在于区分两个不同文件名。
