## 使用git准备工作

#### 安装git命令
`$ apt-get install git-core`

#### 配置用户信息
`$ git config --global user.name "wxw"`  
`$ git config --global user.email wxw_it@163.com`

#### 检查已有的配置信息
`git config --list`

#### 使用ssh-key实现远程免密码提交（只针对git/ssh协议）
`$ cd ~/.ssh`  
`$ ssh-keygen -t rsa -C "wxw_it@163.com"`  
提示输入时，直接回车。然后就生成两个文件：id_rsa , id_rsa.pub。  
在GitHub上注册一个用户，然后进入SSH keys，把id_rsa.pub的内容复制进去保存即可。

#### 测试连接是否成功
`$ ssh -T git@github.com`  
`Hi wuxiwei! You've successfully authenticated, but GitHub does not provide shell access.`

### 本地仓库和远程仓库使用

#### 克隆git clone操作，以aLittle为例。
情况一：不用GitHub帐号，或则没有将私密id_rsa.pub保存到GitHub的SSH keys上  
`$ git clone https://github.com/wuxiwei/aLittle.git`  
情况一：将私密id_rsa.pub保存到GutHub的SSH keys上，否则提示没有权限  
`$ git clone git@github.com:wuxiwei/aLittle.git`  
目录aLittle即为本地仓库

***
#### 将本目录初始化为本地仓库
`$ git init`
#### 添加远程仓库
`$ git remote add origin git@github.com:wuxiwei/aLittle.git`  
origin为该远程仓库起的名称，可自定义。
#### 可以通过如下命令查看当前仓库连接的远程仓库
`$ git remote -v`
#### 将远程仓库的数据拉取到本地仓库
`$ git pull origin master`  
其中origin为远程仓库，master为本地默认主分支名称。如果本地仓库和远程仓库由冲突，必须先拉取远程代码。

## 代码提交远程仓库
`$ git push origin master`
其中origin为远程仓库，master为本地默认主分支名称。  
远程仓库是https协议下，每次都会提示输入GitHub帐号和密码。git/ssh协议下，将私密id_rsa.pub保存到GitHub上即可密码提交。
