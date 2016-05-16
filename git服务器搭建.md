### 搭建私人的git服务器，在push的同时自动同步到web站点目录
>有这样一种需求，在本地开发，并在服务器上搭建私人的git服务器。然后将本地的数据push到服务器上，希望可以自动同步到web站点目录，这样就可以直接查看到网页的效果。

#### 第一步 创建git用户

```
$ sudo adduser git
$ su git
$ cd
$ mkdir .ssh && chmod 700 .ssh
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

#### 第二步 为用户git的authorized_keys文件添加一些开发者的SSH公钥。

公钥获取： 开发用户到家目录，进入.ssh文件夹，找到id_dsa.pub命名的文件，内容就是SSH公钥。如果没有，运行ssh-keyen程序来创建。

将获取的公钥发送复制到服务器的authorized_keys文件中。  

#### 第三步 确定存放git仓库的位置，并创建仓库。

我们使用git家目录作为存放git的仓库，执行创建Test.git仓库的命令（git仓库建议用.git后缀）。

```
$ cd /home/git
$ git init --bare Test.git
```

可以在git家目录下看到Test.git仓库，为了写入数据，最好将仓库的所属和权限给到git用户。

```
$ chown -R git:git Test.git
$ chmod -R 775 Test.git
```

#### 第四步 本地用户John实现添加远程仓库，并推送最初版本到这个仓库中。（假设服务器地址gitsrever或某个IP）

```
# on John's computer
$ cd myproject
$ git init
$ git add .
$ git commit -m 'initial commit'
$ git remote add origin git@gitserver:/home/git/Test.git
$ git push origin master
```

也可以本地用户Son实现克隆此仓库，并推送文件。

```
$ git clone git@gitserver:/home/git/Test.git
$ cd project
$ vim README
$ git commit -m 'fix for the README file'
$ git push origin master
```

注意：将开发者的SSH公钥提前保存到服务器上，才会有读写权限。

#### 第五步 基本实现git服务器搭建后，可以发现此时的主机可以通过git用户身份直接登陆，处于安全考虑，对git用户加以限制。

通过编辑`/etc/passwd`文件完成，找到如下一行：

```
git:x:1001:1001:,,,:/home/git/bin/bash
```

改为：

```
git:x:1001:1001:,,,:/home/git/usr/bin/git-shell
```

如果将 /usr/bin/git-shell 设置为用户 git 的登录 shell（login shell），那么用户 git 便不能获得此服务器的普通 shell 访问权限。  

#### 第六步 git仓库里面的内容如果不做处理，将只是在仓库内保存着，但是如果想要将内容同样备份到服务器的某个目录下，假如想要将程序同步到web站点目录，可以通过设置钩子实现。

在服务器裸版本库目录下有一个hooks目录，进去后添加一个post-receive的可执行文件，并加入如下内容：

```
#!/bin/bash
git --work-tree=/var/www/html/Test checkout -f
```

注意：--work-tree对应web站点目录。同样为了写入数据最好将站点目录的所属和权限给到git用户。

然后就可以开始在本地版本库工作区里开发，然后使用git push指令推送到远程版本库，钩子post-receive会自动生效，将文件检出到--work-tree目录里，实现同步。
