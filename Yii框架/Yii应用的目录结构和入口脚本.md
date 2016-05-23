以下是一个通过高级模版安装后典型的Yii应用的目录结构：
```
.
├── backend
├── common
├── console
├── environments
├── frontend
├── nbproject
├── tests
├── vendor
├── composer.json
├── composer.lock
├── init
├── init.bat
├── LICENSE.md
├── README.md
├── requirements.php
├── yii
└── yii.bat
```
对于高级应用而言，相当于有`backend frontend console`三个独立的Yii应用。 由于`console`类的应用比较特殊，我们稍后再讲。这里讲典型的Web应用的目录结构。
#### 公共目录
这里的公共目录可不止`common`目录，但这个目录从字面上来看，是所有公共目录里最“公共”的。`common`目录下的东西，对于本高级应用的任一独立的应用而言，都是可见、可用的。一般情况下，`common`具有以下结构：
```
.
├── config
├── mail
└── models
```
其中：
* `config`就是通用的配置，这些配置将作用于前后台和命令行。
* `mail`就是应用的前后台和命令行的与邮件相关的布局文件等。
* `models`就是前后台和命令行都可能用到的数据模型。 这也是 common 中最主要的部分。

除了`common`之外，还有一个很重要的公共目录，`vendor`。这个目录从字面的意思看，就是各种第三方的程序。这是Composer安装的其他程序的存放目录，包含Yii框架本身，也放在这个目录下面。 如果你向`composer.json`目录增加了新的需要安装的程序，那么下次调用Composer的时候，就会把新安装的目录也安装在这个`vendor`下面。  
好了，现在问题来了。对于`frontend backend console`等独立的应用而言，他们的内容放在各自的目录下面，他们的运作必然用到Yii框架等`vendor`中的程序。 他们是如何关联起来的？这个秘密，或者说整个Yii应用的目录结构的秘密，就包含在一个传说中的称为入口文件的地方。  
但是在了解入口文件index.php之前，有必要先看看诸如`frontend`等独立应用的目录结构。这比起整个Yii应用的目录结构面言，更为重要。因为你往往是在`frontend`等目录下写代码。但是，不大会在`path\to\digpage`目录下写代码。
#### 前台的目录结构
其实，前台和后台是一样的，只是我们逻辑上的一个划分。典型的，`frontend`具有如下的一个目录结构：
```
.
├── assets
├── config
├── controllers
├── models
├── runtime
├── views
├── web
└── widgets
```
按照顺序来讲：
* `assets`目录用于存放前端资源包PHP类。这里不需要了解什么是前端资源包，只要大致知道是用于管理CSS、js等前端资源就可以了。
* `config`用于存放本应用的配置文件，包含主配置文件`main.php`和全局参数配置文件`params.php`。
* `models views controllers`3个目录分别用于存放数据模型类、视图文件、控制器类。这个是我们编码的核心，也是我们工作最多的目录。
* `widgets`目录用于存放一些常用的小挂件的类文件。
* `tests`目录用于存放测试类。
* `web`目录从名字可以看出，这是一个对于Web服务器可以访问的目录。除了这一目录，其他所有的目录不应对Web用户暴露出来。这是安全的需要。
* `runtime`这个目录是要求权限为`chmod 777`，即允许Web服务器具有完全的权限，因为可能会涉及到写入临时文件等。但是一个目录并未对Web用户可见。也就是说，权限给了，但是并不是Web用户可以访问到的。

`backend`目录与`frontend`的结构、内容是一样一样的。所谓的前台和后台，只是人为从逻辑上对Web应用的功能划分，目的在于分解应用的规模和复杂程度，便于维护和使用。从代码角度来讲，Yii压根就不认得哪个是前台，哪个是后台。  
前面提到的，传说中的入口文件`index.php`就位于`web`目录下面。
#### 入口文件index.php
首先来看看 index.php 文件的内容:
```
<?php
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'dev');

require(__DIR__ . '/../../vendor/autoload.php');
require(__DIR__ . '/../../vendor/yiisoft/yii2/Yii.php');
require(__DIR__ . '/../../common/config/bootstrap.php');
require(__DIR__ . '/../config/bootstrap.php');

$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../../common/config/main.php'),
    require(__DIR__ . '/../../common/config/main-local.php'),
    require(__DIR__ . '/../config/main.php'),
    require(__DIR__ . '/../config/main-local.php')
);

$application = new yii\web\Application($config);
$application->run();
```
##### 设置调试模式和代码环境
前两行是两个`define`语句:
```
defined(‘YII_DEBUG’) or define(‘YII_DEBUG’, true); defined(‘YII_ENV’) or define(‘YII_ENV’, ‘dev’);
```
定义当前的运行模式和环境。如果定义了`YII_DEBUG`，那么表示当前为调试状态，应用在运行过程中，会有一些调试信息的输出。在抛出异常时，也会有一个详细的调用栈的显示。默认情况下，`YII_DEBUG`为`false`。但在开发过程中，最好按上面写的那样，定义为`true`这样便于查找和分析错误。  
如果定义了`YII_ENV`，那么就是指定了当前应用的运行环境。上面的代码显示应用将运行于`dev`环境。默认情况下，`YII_ENV`为`prod`表示产品环境。  
这些环境只是一个名称，具体的意义和环境内容要看环境的定义。`dev prod`是Yii安装后默认的两个环境，分别表示开发环境和最终的产品环境。此外还有一个`test`环境，表示测试环境。  
环境与模式的作用不同。环境在代码中主要是影响配置文件。`YII_ENV`的`dev prod test`三种环境，会分别使`YII_ENV_DEV YII_ENV_PROD YII_ENV_TEST`的值为`true`。这样，在应用的配置中，特别是在相同的一个配置文件中，可以对不同环境作出不同的配置。  
比如，你希望在测试环境下，使用另一个数据库。在开发环境下，启用调试工作条，等等。那么，可以这么做:
```
$config = [...];

if (!YII_ENV_TEST) {
    // 以下配置项仅在测试环境中起作用
    $config['bootstrap'][] = 'debug';
    $config['modules']['debug'] = 'yii\debug\Module';
    $config['modules']['gii'] = 'yii\gii\Module';
}
```
其实，这个`YII_ENV`的定义就是一个与`init`脚本环境切换的相互补充。如果各环境比较明晰，用`init`来切换各种环境的配置是完全够用的。不必在脚本中再有如`YII_ENV_TEST`之类的判断语句，这会使本来已经显得很长的配置文件看上去更臃肿。
##### 引入必要的文件
紧接着两个`define`语句之后，就是4个`require`语句:
```
require(__DIR__ . '/../../vendor/autoload.php');
require(__DIR__ . '/../../vendor/yiisoft/yii2/Yii.php');
require(__DIR__ . '/../../common/config/bootstrap.php');
require(__DIR__ . '/../config/bootstrap.php');
```
这4个语句的前3个中都使用到了相对于当前目录的叔伯目录中的php文件，第4个语句使用了相对于当前目录的兄弟目录。  
我们知道，`__DIR__`表示当前文件`index.php`所在的目录。`/../../`表示的是当前目录的爷爷目录。若`index.php`的当前目录是`/path/to/digpapge.com/frontend/web`，那么爸爸目录就是`frontend`，爷爷目录就是`digpage.com`了。  
第一个require引入了`/path/to/digpage.com/vendor`下面的`autoload.php`。这个是composer的类自动加载机制注册文件。引入这个文件后，可以使用composer的类自动加载功能。  
第二个引入了`vendor`下面的`yiisoft/yii2/Yii.php`，这是Yii的工具类文件。引入了这个类文件后，才能使用Yii的提供的各种工具，才有`Yii::createObject() Yii::$app`之类的东东可以使用。  
第三个引入了`/path/to/digpage.com/common`下面的`config/bootstrap.php`。这个文件主要用于执行一些Yii应用引导的代码，比如定义一系列的路径别名:
```
<?php
Yii::setAlias('common', dirname(__DIR__));
Yii::setAlias('frontend', dirname(dirname(__DIR__)) . '/frontend');
Yii::setAlias('backend', dirname(dirname(__DIR__)) . '/backend');
Yii::setAlias('console', dirname(dirname(__DIR__)) . '/console');
Yii::setAlias('vendor', dirname(dirname(__DIR__)) . '/vendor');
```
这是默认安装后定义好的`common frontend backend console vendor`5个路径别名，如果你要新增一个用于表示插件的目录`plugin`可以自己在这个文件里面加一行:
```
Yii::setAlias('plugin', dirname(dirname(__DIR__)) . '/plugins');
```
第四个require引入了`path/to/digpage.com/frontend`下面的`config/bootstrap.php`。作用与上面类似，只是其中的代码仅适用于当前应用（frontend）。而第三个require中的，是适应于所有应用（common）。  
再接下来，是一个函数`yii\helpers\ArrayHelper::merge()`。这个函数的作用在于合并参数所指定的各个数组。其中，后面的数组会把前面数组中相同下标的元素覆盖掉。这个语句的作用，就是读取、合并应用的各配置文件并保存在`$config`变量中。这里我们看到一共是读取了4个配置文件: 
```
require('path/to/digpage.com/common/config/main.php'),
require('path/to/digpage.com/common/config/main-local.php'),
require('path/to/digpage.com/frontend/config/main.php'),
require('path/to/digpage.com/frontend/config/main-local.php')
```
依次是通用目录common下的2个配置文件，和当前应用frontend下的2个配置文件。在优先顺序上，当前的配置覆盖通用的配置。同时，带有`-local`的配置文件在后，所以，本地配置文件覆盖团队配置文件。  
最后，以`$config`为参数，实例化了一个`Application`对象，并调用他的`run()`函数。这时，Yii应用就跑起来了。
#### 命令行应用入口脚本
命令行应用的入口脚本是`path/to/digpage.com/yii`文件。这个文件被`init`脚本设为可执行的。他的内容如下:
```
#!/usr/bin/env php
<?php

defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'dev');

// fcgi doesn't have STDIN and STDOUT defined by default
defined('STDIN') or define('STDIN', fopen('php://stdin', 'r'));
defined('STDOUT') or define('STDOUT', fopen('php://stdout', 'w'));

require(__DIR__ . '/vendor/autoload.php');
require(__DIR__ . '/vendor/yiisoft/yii2/Yii.php');
require(__DIR__ . '/common/config/bootstrap.php');
require(__DIR__ . '/console/config/bootstrap.php');

$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/common/config/main.php'),
    require(__DIR__ . '/common/config/main-local.php'),
    require(__DIR__ . '/console/config/main.php'),
    require(__DIR__ . '/console/config/main-local.php')
);

$application = new yii\console\Application($config);
$exitCode = $application->run();
exit($exitCode);
```
对比于Web应用的`index.php`入口脚本，`yii`并没有太多的新东西，其中核心的东西根本就没变。我们先来看看这个`yii`是什么？  
首先，它没有扩展名，我们不好知道其具体类型。但是从文件内容的第一行`#!/usr/bin/env php`来看，这是一个bash脚本。第一行在告诉bash，也在告诉我们，这是一个使用PHP运行的脚本。  
但第二行的`<?php`又清楚的向我们表明，这货其实也是个`PHP`文件，只是没有加上PHP后缀而已 。  
接下来，`define('STDIN')`和`define('STDOUT')`则为fcgi定义了标准输入和标准输出。  
在各require语句中，由于`yii`的位置与`index.php`不同，是位于应用根目录下，所以目录结构上更简单些。  
最后，在Yii应用跑起来后，还要获取其返回值，并以该返回值退出脚本，通知操作系统退出时的状态。对于Windows系统而言，命令行的入口脚本仍然是`yii`，但是命令行下无法直接运行。所以，细心的Yii为我们准备了一个`yii.bat`。这个文件会以`php`yii 形式调用PHP来运行入口脚本 。 

引用：[Yii应用的目录结构和入口脚本 — 深入理解Yii2.0](http://www.digpage.com/app_struct.html)
