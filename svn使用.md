##svn基本操作
1. 将文件checkout到本地目录  
`$ svn checkout path`（path是服务器上的目录）  
2. 往版本库中添加新的文件  
`$ svn add file`  
3. 查看文件状态  
`$ svn status`  
4. 将改动的文件提交到版本库  
`svn commit -m “LogMessage”`  
5. 更新某个版本  
`svn update`

##svn忽略文件办法
假设想忽略文件temp  
1. cd到temp所在的目录下：  
2. svn propedit svn:ignore .  
注意：请别漏掉最后的点（.表示当前目录），如果报错请看下面  
3. 打开的文件就是忽略列表文件了（默认是空的），每一行是一项，在该文件中输入temp，保存退出  
4. svn st查看状态，temp的?状态已经消除了  
