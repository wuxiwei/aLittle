## vim配置文件和插件管理 ##
本文通过总结零碎的资料总结而成，更多是去引导学习vim配置文件及插件使用，没有讲解任何vim命令。

 - 先是最基本的.vimrc配置文件，内容如下（备注清晰）：

```
"引入插件pathogen使用
call pathogen#infect()
colorscheme torte
"禁用VI兼容模式
set nocompatible
"Vim 的内部编码
set encoding=utf-8
"Vim 在与屏幕/键盘交互时使用的编码(取决于实际的终端的设定)
set termencoding=utf-8
"Vim 当前编辑的文件在存储时的编码
set fileencoding=utf-8
"Vim 打开文件时的尝试使用的编码
set fileencodings=ucs-bom,utf-8,gbk,default,latin1
"开启语法高亮功能
syntax enable
"允许用指定语法高亮配色替换默认方案
syntax on
"显示行号
set nu
" 括号匹配
set showmatch
"自动检测文件类型
filetype plugin indent on
"在缩进和遇到 Tab 键时使用空格替代
set expandtab
"根据文件类型设置缩进格式
au FileType html,python,vim,javascript setl shiftwidth=2
au FileType html,python,vim,javascript setl tabstop=2
au FileType java,php setl shiftwidth=4
au FileType java,php setl tabstop=4
" 启动vim时不要自动折叠代码
set foldlevel=100
"自动对齐
set ai
"依据上面的对起格式
set si
set smarttab
set wrap
set lbr
set tw=0
set foldmethod=syntax
" 总是显示状态栏
set laststatus=2
" 显示光标当前位置
set ruler
" 高亮显示当前行/列
set cursorline
" 高亮显示搜索结果
set hlsearch
"禁止拆行
set nowrap
"设置快捷键前缀，即<Leader>
let mapleader=";"
"设置快捷键将选中文本块复制到系统剪贴板
vnoremap <Leader>y "+y
"设置快捷键将系统剪贴板内容粘帖到vim
nmap <Leader>p "+p
"配置文件.vimrc更改后自动重新载入使设置生效
autocmd! bufwritepost .vimrc source ~/.vimrc
"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"
" => Plugin configuration
"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"”"
" nerdtree
"在 vim 启动的时候默认开启 NERDTree
autocmd VimEnter * NERDTree
"设置NERDTree打开关闭快捷键
map <F3> :NERDTreeToggle<CR>
" taglist  存在函数显示列表插件
"启动vim后自动打开taglist窗口
let Tlist_Auto_Open = 1
"不同时显示多个文件的tag，仅显示一个
let Tlist_Show_One_File = 1
"taglist为最后一个窗口时，退出vim
let Tlist_Exit_OnlyWindow = 1
"taglist窗口显示在右侧，缺省为左侧
let Tlist_Use_Right_Window =1
"设置taglist窗口大小
"let Tlist_WinHeight = 100
let Tlist_WinWidth = 30
"设置taglist打开关闭的快捷键F8
noremap <F10> :TlistToggle<CR>
"更新ctags标签文件快捷键设置
noremap <F9> :!ctags -R<CR>
"autocomplpop  自动补全插件
"只有在是PHP文件时，才启用PHP补全
au FileType php setlocal dict+=~/.vim/bundle/vim-autocomplpop/php_funclist.txt
"tag标签设置
"设置自动切换目录
set autochdir
"自动查找
set tags=tags;
"jsbeautify格式化js插件
nnoremap <F4> :call g:Jsbeautify()<CR>
```
这份配置文件只能是基础使用，开始是简单的字符编码设置，再是语法高亮（VIM自带javascript语法高亮，但是自带的那个位于syntax目录下的javascript.vim那个配置文件比较弱，有很多关键词没有高亮。我们可以到www.vim.org搜索最新的javascript.vim代替了原来的那个文件的)，其次是对Tab键的设置，并且可以设置根据不同文件区分Tab键宽度。最后是对快捷键的设置，我只设置了复制粘帖，注意这里的复制粘帖是指vim之外电脑本身的剪切板。  
以上就是.vimrc配置文件，一些基本的引导，可以自己做适合自己的快捷键或设置。

 - 接下来具体记录如何处理插件

(注：做同样的事肯定存在多种选择，插件也是，相同功能有很多不同插件可以实现，以下是我个人的选择）

 - 正常安装插件较为繁琐，需要到/usr/share/vim/vim**/文件下操作，插件比较不好管理，所以推荐使用插件管理器。pathogen和vundle都是用来管理vim插件的，但是其作用的方面不同。  
pathogen是为了解决每一个插件安装后文件分散到多个目录不好管理而存在的。  
vundle是为了解决自动搜索及下载插件而存在的。  
我个人使用的是apthogen插件，当插件过多时，条理的安置插件才是最好的，另外大部分使用的插件可以看到并不会短期内更新。介绍pathogen插件：首先，在用户目录的.vim目录下建立autoload目录和bundle目录，autoload目录中放pathogen.vim。可以在~/.vim/autoload/目录下，用如下命令下载  
`curl –Sso pathogen.vim https://raw.github.com/tpope/vim-pathogen/master/autoload/pathogen.vim`  
然后在配置文件的首行添加如下命令execute pathogen#infect()。回头看看上方.vimrc配置文件开头，有了pathogen后，下载的插件就直接把它们放到bundle目录下即可，而不需要管理相应的autoload、colors、plugin等目录。添加help文件：在vim下:helptags ~/.vim/doc/即可通过help命令查看插件文档。  
(注：哪怕插件只是一个.vim文件，也可以先新建一个文件夹，然后再建plugin文件夹，最后将该插件放入即可）  

有了这么好的插件管理器，安装插件就变得非常方便，因为vim毕竟是IT开发人员开发出来的，所以vim最终受益者还会是开发人员，所以接下来从配置IDE开发环境记录插件。  
配置开发环境分以下几点：

 - 文件浏览插件，一个好的文件浏览肯定少不了目录树，并且能够方便切换。我推荐使用NERDTree。我们先安装然后后续可以自己多看文档，正如我前面所说的有了好的插件管理器，就可以很方便完成安装使用。我们可以直接去到官网http://www.vim.org/scripts/script.php?script_id=1658 选择最新版本直接下载即可。我们将压缩包解压以后，通过cp命令行将文件夹直接复制到~/.vim/bundle/下即可，可以打开vim，并在并在命令行输入：NERDTree，并可以看到左边跳出当前文件下的目录树，为了便于方便可以设置每次打开vim自动跳出NERDTree所以我们可以在.vimrc文件中设置，同时可以设置打开关闭快捷键。是不是很方便！

 - 对齐文本插件，对于经常写代码来说，有Tabular会很方便，文本可以按等号，冒号等来对齐文本。我们还是先直接安装插件，到https://github.com/godlygeek/tabular 下载，这次是git上的一个项目，不用慌还是直接下载即可，得到安装包后解压到~/.vim/bundle/即可直接使用。  

接下来需要记录的几个插件都需要基于一个tags来实现。这里不多加解释，需要自己慢慢去啃。。。http://www.java123.net/v/583584.html ，这篇文章将tags介绍的很详细，简单的说tags是一个linux上很普遍的源码分析工具, 可以将代码中的函数变量等定义的位置记录在一个名称为tags的文件，类似于数据库记录功能，而接下来的插件就是需要用的这些标签，可以通过命令  
`sudo apt-get install ctags`  
进行安装（Ubuntu下），然后在工作目录下生成tags标签文件，最方便的命令是  
`ctags -R`  
直接根据目录下所有文件来获取标签，类似于函数名，变量名等，最后在.vimrc文件中添加路径  
`:set tags+=/home/user/tags`  
就可以让vim在每次启动的时候自动找到tags标签文档。有几个缺点：    
一、直接根据全部文件会浪费空间，tags文件过大。那么我们可以根据想要的某些文件类型来生成tags文件，像*.js,*.html等。  
二、每次我们换个工作目录都需要更换路径，似乎就会不方便，所以最后我们选择一种比较好的方法来避免，可以看上面.vimrc配置文件。  
三、最后就是对tags的更新问题，查了很多资料，都没有很好的解决，有通过编写shell脚步来自动更新，有些麻烦，所以最后选择通过添加快捷键来手动更新tags文件，按F9就可以直接实现，也算方便。更多关于tags的内容还是多看看资料吧。

 - 说了那么多，接下来继续记录插件，函数跳转插件，对于IDE而言也算最基本也是最方便的功能之一了吧，还是直接通过官网http://ctags.sourceforge.net/ 下载最新版本，解压缩到文件夹内，并将该文件夹直接复制到~/.vim/bundle/下即可，一些具体的跳转可以通过查看文档去深入学习，也可以通过快捷键来跳转。这里有没有疑问，它是如何实现跳转，哪来的标签？对没错就是我们前面介绍的tags标签文件，所以可以想象到tags文件生成的好坏直接影响到跳转的功能。

 - 自动补全插件，又一个强大的功能，相信对于每位开发者都不陌生，autocomplpop插件就实现了自动补全功能，同样我们先通过http://www.vim.org/scripts/script.php?script_id=1879 下载并安装，补全同样得益于tags文件，插件会实时检测是否有可以补全的函数或变量名等，这里额外加一点，对于一门语言来说，靠tags还是不够的，所以我们需要添加语言的参考文档，像自带函数，全局变量等，来辅助补全插件，举个php开发环境例子，所以我们找php的相关补全标签  
`wget http://chenpeng.info/apps/vim/funclist.txt`  
可以直接下载文件到当前目录，可以修改易于记忆的文件名，最后添加路径到.vimrc配置文件，可以看上面文件内容参考。

 - taglist插件，可以将tags标签展示在右边框内，帮助开发人员查看当前文件中的宏、全局变量、函数等标签，先通过http://vim.sourceforge.net/scripts/script.php?script_id=273  安装，然后在.vimrc文件中配置插件，参考上面文档内容，就可以自定义显示方式。通过选择标签也可以实现直接跳转功能，具体实现可以查看帮助文档。

 - 格式化代码，可以增强代码的可读性，举js代码的格式化例子，Jsbeautify插件可以通过http://www.vim.org/scripts/script.php?script_id=2727 下载使用，同时可以参考上面文档设置快捷键，方便使用。

 - syntastic插件，用来做静态语法检查的，支持多语言，比如 Ruby，Python 和 JavaScript 等。实际上这个插件是个接口，背后的语法检查是交给各个语言自己的检查器，Ruby 实际使用ruby -c命令，JavaScript 使用 jshint，jslint 等。

以上就是刚开始接触vim配置及插件的记录。在以后学习中可以慢慢的不断完善配置文档和插件的选择，最后希望能有真正属于自己并且适合自己的.vimrc。  
推荐一些不错的vim学习网站。  
1.http://easwy.com/blog/archives/tag/vim/
书也不错，讲解vim命令的操作  
2.http://ju.outofmemory.cn/entry/79671
跟我一起谢谢VIM  
3.http://www.cnblogs.com/mo-beifeng/archive/2011/09/07/2169994.html
配置一个高效的PHP开发环境Vim  