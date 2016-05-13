#初次使用git连接github（ubuntu）
##安装git命令
`$ apt-get install git-core`
##使用ssh-key实现远程免密码登陆
`$cd ~/.ssh`

`$ssh-keygen -t rsa -C "wxw_it@163.com"`

之后直接回车，不用填写东西，之后会让你输入密码（建议不输密码，才能实现免密码登陆）。然后就生成两个文件：id_rsa , id_rsa.pub。在GitHub上注册一个用户，然后进入SSH keys，把id_rsa.pub的内容复制进去保存即可。
##测试连接是否成功
`$ssh -T git@github.com`  
`Hi wuxiwei! You've successfully authenticated, but GitHub does not provide shell access.`
##配置用户信息
`$ git config --global user.name "wxw"`  
`$ git config --global user.email wxw_it@163.com`
##检查已有的配置信息
`git config --list`
##使用git clone或测试连接是否成功时出现如下问题
`ssh: connect to host github.com port 22: Connection refused`  
`fatal: Could not read from remote repository.`  
`Please make sure you have the correct access rights and the repository exists.`
##解决办法
`$ vim .ssh/config`  
`Host github.com`  
`User wxw_it@163.com`  
`Hostname ssh.github.com`  
`PreferredAuthentications publickey`  
`IdentityFile ~/.ssh/id_rsa`  
`Port 443`
##测试连接是否成功
`$ssh -T git@github.com`
`Hi wuxiwei! You've successfully authenticated, but GitHub does not provide shell access.`
##如果出现如下警告
`Warning: Permanently added '[ssh.github.com]:443,[192.30.252.151]:443' (RSA) to the list of known hosts.`
##解决办法
`$vim /etc/hosts`
##添加一行
192.30.252.151   github.com
##接下来就可以克隆操作
##以gerrit-trigger-plugin为例，下面的链接都是从相应页面上直接拷贝的。
###法一：不用github的账号，打开这个库在github上的主页，运行下面命令即可
####read only
`$git clone https://github.com/jenkinsci/gerrit-trigger-plugin.git`
####下面的三种方法都要先在github上注册账户，然后生成相应的ssh key，并把public key添加到个人账户里面，详见github帮助
####read+write
`$git clone git@github.com:flyingbird1221/gerrit-trigger-plugin.git`
####read+write
`$git clone https://flyingbird1221@github.com/flyingbird1221/gerrit-trigger-plugin.git`
###会提示输入密码，注意此处的密码不是你在github上账户的密码，而是当前登录系统用户的密码。
####read only
`$git clone git://github.com/flyingbird1221/gerrit-trigger-plugin.git`
##本地仓库直接添加到github上
`cd newdir`  
`git init`
####添加远程仓库
`git remote add origin git@github.com:wuxiwei/aLittle.git`
####更新数据
`git pull origin master`
####上传数据
`git push origin master`
