在Yii中，所有类、接口、Traits都可以使用类的自动加载机制实现在调用前自动加载。Yii借助了PHP的类自动加载机制高效实现了类的定位、导入，这一机制兼容PSR-4的标准。在Yii中，类仅在调用时才会被加载，特别是核心类，其定位非常快，这也是Yii高效高性能的一个重要体现。
#### PHP类自动加载机制详解
php类自动加载主要由函数`spl_autoload_register()`实现，在了解这个函数之前先看一下另一个函数`__autoload()`，这是一个自动加载函数，在PHP5中，当我们实例化一个未定义的类时，就会触发此函数。看下面例子：
```
printit.class.php 
 
<?php 
class PRINTIT { 
    function doPrint() {
        echo 'hello world';
    }
}
?> 
 
index.php 
 
<?
function __autoload( $class ) {
    $file = $class . '.class.php';  
    if ( is_file($file) ) {  
        require_once($file);  
    }
} 
 
$obj = new PRINTIT();
$obj->doPrint();
?>
```
运行`index.php`后正常输出`hello world`。在`index.php`中，由于没有包含`printit.class.php`，在实例化`printit`时，自动调用`__autoload`函数，参数`$class`的值即为类名`printit`，此时`printit.class.php`就被引进来了。在面向对象中这种方法经常使用，可以避免书写过多的引用文件，同时也使整个系统更加灵活。  
再看`spl_autoload_register()`，这个函数与`__autoload`有与曲同工之妙，看个简单的例子：
```
<?
function loadprint( $class ) {
    $file = $class . '.class.php';  
    if (is_file($file)) {  
        require_once($file);  
    } 
} 
 
spl_autoload_register( 'loadprint' ); 
 
$obj = new PRINTIT();
$obj->doPrint();
?>
```
将`__autoload`换成`loadprint`函数。但是`loadprint`不会像`__autoload`自动触发，这时`spl_autoload_register()`就起作用了，它告诉PHP碰到没有定义的类就执行`loadprint()`。  
`spl_autoload_register()`调用静态方法： 
```
<? 
class test {
    public static function loadprint( $class ) {
        $file = $class . '.class.php';  
        if (is_file($file)) {  
            require_once($file);  
        } 
    }
} 
 
spl_autoload_register(  array('test','loadprint')  );
//另一种写法：spl_autoload_register(  "test::loadprint"  ); 
 
$obj = new PRINTIT();
$obj->doPrint();
?>
```
#### Yii自动加载机制的实现
Yii的类自动加载，依赖于PHP的`spl_autoload_register()`，注册一个自己的自动加载函数（autoloader），并插入到自动加载函数栈的最前面，确保Yii的autoloader会被最先调用。  
类自动加载的这个机制的引入要从入口文件`index.php`开始说起:
```
<?php
defined('YII_DEBUG') or define('YII_DEBUG', false);
defined('YII_ENV') or define('YII_ENV', 'prod');

// 这个是第三方的autoloader
require(__DIR__ . '/../../vendor/autoload.php');

// 这个是Yii的Autoloader，放在最后面，确保其插入的autoloader会放在栈最前面
require(__DIR__ . '/../../vendor/yiisoft/yii2/Yii.php');
// 后面不应再有autoloader了

require(__DIR__ . '/../../common/config/aliases.php');

$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../../common/config/main.php'),
    require(__DIR__ . '/../../common/config/main-local.php'),
    require(__DIR__ . '/../config/main.php'),
    require(__DIR__ . '/../config/main-local.php')
);

$application = new yii\web\Application($config);
$application->run();
```
这个文件主要看点在于第三方autoloader与Yii实现的autoloader的顺序。不管第三方的代码是如何使用`spl_autoload_register()`来注册自己的autoloader的，只要Yii的代码在最后面，就可以确保其可以将自己的autoloader插入到整个autoloder栈的最前面，从而在需要时最先被调用。  
接下来，看看Yii是如何调用`spl_autoload_register()`注册autoloader的，这要看`Yii.php`里发生了些什么:
```
<?php
require(__DIR__ . '/BaseYii.php');
class Yii extends \yii\BaseYii
{
}

// 重点看这个 spl_autoload_register
spl_autoload_register(['Yii', 'autoload'], true, true);

// 下面的语句读取了一个映射表
Yii::$classMap = include(__DIR__ . '/classes.php');

Yii::$container = new yii\di\Container;
```
这段代码，调用了`spl_autoload_register(['Yii', 'autoload', true, true])`，将`Yii::autoload()`作为autoloader插入到栈的最前面了。并将`classes.php`读取到`Yii::$classMap`中，保存了一个映射表。  
在上面的代码中，Yii类是里面没有任何代码，并未对`BaseYii::autoload()`进行重载，所以，这个`spl_autoload_register()`实际上将`BaseYii::autoload()`注册为autoloader。如果，你要实现自己的autoloader，可以在`Yii`类的代码中，对`autoload()`进行重载。  
在调用`spl_autoload_register()`进行autoloader注册之后，Yii将`calsses.php`这个文件作为一个映射表保存到`Yii::$classMap`当中。这个映射表，保存了一系列的类名与其所在PHP文件的映射关系，比如:
```
return [
  'yii\base\Action' => YII2_PATH . '/base/Action.php',
  'yii\base\ActionEvent' => YII2_PATH . '/base/ActionEvent.php',

  ... ...

  'yii\widgets\PjaxAsset' => YII2_PATH . '/widgets/PjaxAsset.php',
  'yii\widgets\Spaceless' => YII2_PATH . '/widgets/Spaceless.php',
];
```
这个映射表以类名为键，以实际类文件为值，Yii所有的核心类都已经写入到这个`classes.php`文件中，所以，核心类的加载是最便捷，最快的。现在，来看看这个关键先生`BaseYii::autoload()`
```
public static function autoload($className)
{
    if (isset(static::$classMap[$className])) {
        $classFile = static::$classMap[$className];
        if ($classFile[0] === '@') {
            $classFile = static::getAlias($classFile);
        }
    } elseif (strpos($className, '\\') !== false) {
        $classFile = static::getAlias('@' . str_replace('\\', '/',
            $className) . '.php', false);
        if ($classFile === false || !is_file($classFile)) {
            return;
        }
    } else {
        return;
    }

    include($classFile);

    if (YII_DEBUG && !class_exists($className, false) &&
        !interface_exists($className, false) && !trait_exists($className,
        false)) {
        throw new UnknownClassException(
        "Unable to find '$className' in file: $classFile. Namespace missing?");
    }
}
```
从这段代码来看Yii类自动加载机制的运作原理：  
* 检查`$classMap[$className]`看看是否在映射表中已经有拟加载类的位置信息；
* 如果有，再看看这个位置信息是不是一个路径别名，即是不是以`@` 打头，是的话，将路径别名解析成实际路径。如果映射表中的位置信息并非一个路径别名，那么将这个路径作为类文件的所在位置。类文件的完整路径保存在`$classFile`；
* 如果`$classMap[$className]`没有该类的信息，那么，看看这个类名中是否含有`\\`，如果没有，说明这是一个不符合规范要求的类名，autoloader直接返回。PHP会尝试使用其他已经注册的autoloader进行加载。如果有`\\`，认为这个类名符合规范，将其转换成路径形式。即所有的`\\`用`/`替换，并加上`.php`的后缀。
* 将替换后的类名，加上`@`前缀，作为一个路径别名，进行解析。从别名的解析过程我们知道，如果根别名不存在，将会抛出异常。所以，类的命名，必须以有效的根别名打头:
```
// 有效的类名，因为@yii是一个已经预定义好的别名
use yii\base\Application;

// 无效的类名，因为没有 @foo 或 @foo/bar 的根别名，要提前定义好
use foo\bar\SomeClass;
```
* 使用PHP的 include() 将类文件加载进来，实现类的加载。

从其运作原理看，最快找到类的方式是使用映射表。 其次，Yii中所有的类名，除了符合规范外，还需要提前注册有效的根别名。
#### 运用自动加载机制
在入口脚本中，除了Yii自己的autoloader，还有一个第三方的autoloader:
```
require(__DIR__ . '/../../vendor/autoload.php');
```
这个其实是Composer提供的autoloader。Yii使用Composer来作为包依赖管理器，因此，建议保留Composer的autoloader，尽管Yii的autoloader也能自动加载使用Composer安装的第三方库、扩展等，而且更为高效。但考虑到毕竟是人家安装的，人家还有一套自己专门的规则，从维护性、兼容性、扩展性来考虑，建议保留Composer的autoloader。  
如果还有其他的autoloader，一定要在Yii的autoloader注册之前完成注册，以保证Yii的autoloader总是最先被调用。  
如果你有自己的autoloader，也可以不安装Yii的autoloaer，只是这样未必能有Yii的高效，且还需要遵循一套类似的类命名和加载的规则。就个人的经验而言，Yii的autoloader完全够用，没必要自己重复造轮子。
