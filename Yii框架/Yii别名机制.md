可以将别名视为特殊的常量变量，他的作用在于避免将一些文件路径、URL以硬编码的方式写入代码中，或者多处出现一长串的文件路径、URL。
#### 预定义的别名
Yii中，别名以`@`开头，以区别于正常的文件路径和URL。Yii中预定义了许多常用的别名。别名的定义一般放在应用的最开始的阶段进行，比如引导阶段、初始化阶段等。这样可以保证后续代码可以使用这些定义好的别名。
##### 配置文件中的别名
别名一般放在`digpage.com\common\config\bootstrap.php`， 或者`digpage.com\frontend\config\bootstrap.php`等`bootstrap.php`文件中定义。比如:
```
<?php
Yii::setAlias('common', dirname(__DIR__));
Yii::setAlias('frontend', dirname(dirname(__DIR__)) . '/frontend');
Yii::setAlias('backend', dirname(dirname(__DIR__)) . '/backend');
Yii::setAlias('console', dirname(dirname(__DIR__)) . '/console');
```
在此定义的别名通过入口脚本引入Yii应用中，具体可以看看[入口文件index.php](#)部分的内容。上面的`bootstrap.php`文件定义了`@common，@frontend，@backend和@console`4个别名。开发者也可以自己在`bootstrap.php`中加入自己的别名定义，这是最常运用的定义别名的方式。
##### Yii预定义的别名
相比较于通过`bootstrap.php`的方式定义别名，有的别名就不那么直观了，可能没法一下子看到。而且，这些别名相对比较固定，与Yii框架和应用自身息息相关。这类别名直接写到Yii的代码中去了。对于这类Yii框架预定义的别名，一般不去修改他。改他的收益远小于造成的副作用，这种得不偿失的亏本买卖，高智商人群是不会去干的。  
这些预定义的别名，主要分布在`yii\BaseYii`和`yii\base\Application`等类中。  
在`yii\BaseYii`中:
```
// 定义了 @yii 别名
public static $aliases = ['@yii' => __DIR__];
```
`yii\BaseYii::$aliases`用于保存整个Yii应用的所有的别名。这里默认地把`yii\BaseYii.php`所在的目录作为`@yii`别名。  
另外，对于`yii\base\Application` 在其构造函数`__construct()`中，会调用以下代码:
```
public function preInit(&$config)
{
    ... ...

    // basePath必须在配置文件中给出，否则会抛出弃常
    if (isset($config['basePath'])) {
        // 这里会设置 @app
        $this->setBasePath($config['basePath']);
        unset($config['basePath']);
    } else {
        throw new InvalidConfigException(
        'The "basePath" configuration for the Application is required.');
    }

    // @vendor 如果配置文件中设置了 vendorPath 使用配置的值，否则使用默认的
    if (isset($config['vendorPath'])) {
        $this->setVendorPath($config['vendorPath']);
        unset($config['vendorPath']);
    } else {
        $this->getVendorPath();
    }

    // @runtime 如果配置文件中设置了 runtimePath ，就使用配置的值，否则使用默认的
    if (isset($config['runtimePath'])) {
        $this->setRuntimePath($config['runtimePath']);
        unset($config['runtimePath']);
    } else {
        $this->getRuntimePath();
    }

    ... ...
}
```
上面的代码中，预定义了5个别名： `app`，`@vendor`,`@bower`,`@npm`，`@runtime`。上面的代码中，`basePath`不是别名，但必须由开发者自己在配置文件中设定，表示应用的根目录。对于frontend而言，就是目录`path/to/digpage.com/frontend`。在定义`basePath`时，Yii顺便定义了`@app`，代码在`yii\base\Application::setBasePath()`中:
```
public function setBasePath($path)
{
    parent::setBasePath($path);
    Yii::setAlias('@app', $this->getBasePath());
}
```
可以看出，`@app`与`basePath`是一致的。  
在`yii\base\Application`的初始化过程中，与设置`basePath`类似，在配置`vendorPath runtimePath`时，Yii会调用`setVendorPath() setRuntimePath()`。如果未在配置文件中对这两个配置项作出设置，Yii会调用`getVendorPath()`和`getRuntimePath()`，这两个函数最终也会调用相应的set函数对这些别名进行定义。  
`@vendor`，`@bower`，`@npm`和`@runtime`这4个别名就由这两个set函数定义:
```
public function getVendorPath()
{
    // 在未设置vendorPath时，使用默认值
    if ($this->_vendorPath === null) {
        $this->setVendorPath($this->getBasePath() . DIRECTORY_SEPARATOR . 'vendor');
    }

    return $this->_vendorPath;
}

// 这里定义了3个别名
public function setVendorPath($path)
{
    $this->_vendorPath = Yii::getAlias($path);
    Yii::setAlias('@vendor', $this->_vendorPath);
    Yii::setAlias('@bower', $this->_vendorPath . DIRECTORY_SEPARATOR . 'bower');
    Yii::setAlias('@npm', $this->_vendorPath . DIRECTORY_SEPARATOR . 'npm');
}

public function getRuntimePath()
{
    // 在未设置runtimePath时，使用默认值
    if ($this->_runtimePath === null) {
        $this->setRuntimePath($this->getBasePath() . DIRECTORY_SEPARATOR . 'runtime');
    }

    return $this->_runtimePath;
}

// 这里定义了 @runtime 别名
public function setRuntimePath($path)
{
    $this->_runtimePath = Yii::getAlias($path);
    Yii::setAlias('@runtime', $this->_runtimePath);
}
```
对于上面的代码，默认情况下，会有：  
* `@app`，必须由开发者在配置文件中提供，一般为配置文件的`dirname(__DIR__)`。即`digpage.com/frontend`之类的目录。
* `@vendor`，一般定义为`@app/vendor`，高级模板中则定义为`@app/../vendorr`
* `@bower`，定义为`@vendor/bower`
* `@npm`，定义为`@vendor/npm`
* `@runtime`，定义为`@app/runtime`

但是，这里有一个比较特殊的，就是`@vendor`。对于使用Yii基础模版创建的应用而言，会使用上面提到的`@app/vendor`。但是，对于使用高级模版创建的应用，你会发现，vendor目录并不在`frontend`或`backend`目录下，而是跟他们是兄弟目录。这是因为对于整个工程而言，这个vendor的内容是`frontend`和`backend`等共用的。放在`frontend`或`backend`都不合适，干脆就地提拔吧。 这一点在[Yii应用的目录结构和入口脚本](#)已经讲过了。  
因此，实际上高级应用模版的`@vendor`应该是`@app/../vendor`，上面的代码显然不适用。对此，贴心的Yii也已经考虑到了。在使用高级模板创建应用时，`digpage.com/common/config/main.php`配置文件会重新设定 `endorPathr`
```
'vendorPath' => dirname(dirname(__DIR__)) . '/vendor'
```
这样就避免了使用代码中默认的值。  
对于Web应用，`yii\base\Web\Application`中又定义了`@webroot`和`@web`2个别名:
```
protected function bootstrap()
{
    $request = $this->getRequest();
    Yii::setAlias('@webroot', dirname($request->getScriptFile()));
    Yii::setAlias('@web', $request->getBaseUrl());

    parent::bootstrap();
}
```
这里`@webroot`就是入口脚本`index.php`所在的目录。而`@web`则是URL别名，表示当前应用的根URL地址 。
最后一个藏有别名的地方，在于Yii的扩展（extensions）。当使用Composer安装扩展后，会向`@vendor/yiisoft/extensions.php`写入信息，其中就包含相应的别名。只不过这些别名通常都是二级别名。然后，在`yii\base\Application::bootstrap()`中，将这些扩展的别名进行注册。  
首先看看一个典型的`extensions.php`
```
<?php

$vendorDir = dirname(__DIR__);

return array (
  'yiisoft/yii2-swiftmailer' =>
  array (
    'name' => 'yiisoft/yii2-swiftmailer',
    'version' => '9999999-dev',
    'alias' =>
    array (
      '@yii/swiftmailer' => $vendorDir . '/yiisoft/yii2-swiftmailer',
    ),
  ),

 ... ...

 'yiisoft/yii2-gii' =>
  array (
    'name' => 'yiisoft/yii2-gii',
    'version' => '9999999-dev',
    'alias' =>
    array (
      '@yii/gii' => $vendorDir . '/yiisoft/yii2-gii',
    ),
  ),
);
```
注意上面这段代码中的`alias`，这个键对应的就是一个别名及其所代表的实际路径。至于具体对这个`extensions.php`的内容进行处理并注册成别名的工作，是由`yii\base\Application::bootstrap()`完成:
```
protected function bootstrap()
{
    // 将 extensions.php 的内容读取进 $this->extensions 备用
    if ($this->extensions === null) {
        $file = Yii::getAlias('@vendor/yiisoft/extensions.php');
        $this->extensions = is_file($file) ? include($file) : [];
    }

    // 遍历 $this->extensions 并注册别名
    foreach ($this->extensions as $extension) {
        if (!empty($extension['alias'])) {
            foreach ($extension['alias'] as $name => $path) {
                Yii::setAlias($name, $path);
            }
        }
        ... ...
    }
}
```
经过上面这些代码，我们的各种插件也有了自己的别名，如上面的`@yii\swiftmailer`，`@yii\gii`等，常见的还有`@yii\bootstrap`等。
##### 所有预定义的别名
小结一下，默认预定义别名一共有12个，其中路径别名11个，URL别名只有`@web`1个：
* `@yii` 表示Yii框架所在的目录，也是`yii\BaseYii`类文件所在的位置；
* `@app` 表示正在运行的应用的根目录，一般是`digpage.com/frontend`；
* `@vendor` 表示Composer第三方库所在目录，一般是`@app/vendor`或`@app/../vendor`；
* `@bower` 表示Bower第三方库所在目录，一般是`@vendor/bower`；
* `@npm` 表示NPM第三方库所在目录，一般是`@vendor/npm`；
* `@runtime` 表示正在运行的应用的运行时用于存放运行时文件的目录，一般是`@app/runtime`；
* `@webroot` 表示正在运行的应用的入口文件`index.php`所在的目录，一般是`@app/web`；
* `@web` URL别名，表示当前应用的根URL，主要用于前端；
* `@common` 表示通用文件夹；
* `@frontend` 表示前台应用所在的文件夹；
* `@backend` 表示后台应用所在的文件夹；
* `@console` 表示命令行应用所在的文件夹；
* 其他使用Composer安装的Yii扩展注册的二级别名。

这样，在整个Yii应用中，只要使用上述别名，就可方便、且统一地表示特定的路径或URL。
#### 定义与解析别名
Yii使用`Yii::$aliases[]`来保存别名，定义别名就是将别名及其代表的实际路径或URL写入这个数组，而解析别名就是将别名的信息从数组读取出去并组合。
##### 别名的定义过程
除了像上面的代码那样定义一个别名之外，还有其他的用法:
```
// 使用一个路径定义一个路径别名
Yii::setAlias('@foo', 'path/to/foo');

// 使用一个URL定义一个URL别名
Yii::setAlias('@bar', 'http://www.example.com');

// 使用一个别名定义另一个别名
Yii::setAlias('@fooqux', '@foo/qux');

// 定义一个“二级”别名
Yii::setAlias('@foo/bar', 'path/to/foo/bar');
```
从上面的代码中可以了解到，`Yii::setAlias()`是定义别名的关键。实际上，该方法的代码在`BaseYii::setAlias()`中:
```
public static function setAlias($alias, $path)
{
    // 如果拟定义的别名并非以@打头，则在前面加上@
    if (strncmp($alias, '@', 1)) {
        $alias = '@' . $alias;
    }

    // 找到别名的第一段，即@ 到第一个 / 之间的内容，如@foo/bar/qux的@foo
    $pos = strpos($alias, '/');
    $root = $pos === false ? $alias : substr($alias, 0, $pos);

    if ($path !== null) {
        // 去除路径末尾的 \ / 。如果路径本身就是一个别名，直接解析出来
        $path = strncmp($path, '@', 1) ? rtrim($path, '\\/')
            : static::getAlias($path);

        // 检查是否有 $aliases[$root]，
        // 看看是否已经定义好了根别名。如果没有，则以$root为键，保存这个别名
        if (!isset(static::$aliases[$root])) {
            if ($pos === false) {
                static::$aliases[$root] = $path;
            } else {
                static::$aliases[$root] = [$alias => $path];
            }
        // 如果 $aliases[$root] 已经存在，则替换成新的路径，或增加新的路径
        } elseif (is_string(static::$aliases[$root])) {
            if ($pos === false) {
                static::$aliases[$root] = $path;
            } else {
                static::$aliases[$root] = [
                    $alias => $path,
                    $root => static::$aliases[$root],
                ];
            }
        } else {
            static::$aliases[$root][$alias] = $path;
            krsort(static::$aliases[$root]);
        }

    // 当传入的 $path 为 null 时，表示要删除这个别名。
    } elseif (isset(static::$aliases[$root])) {
        if (is_array(static::$aliases[$root])) {
            unset(static::$aliases[$root][$alias]);
        } elseif ($pos === false) {
            unset(static::$aliases[$root]);
        }
    }
}
```
对于别名的定义过程：
###### 别名规范化
如果要定义的别名`$alias`并非以`@`打头，自动为这个别名加上`@`前缀。总之，只要是别名，必然以`@`打头。下面的两个语句，都定义了相同的别名`@foo`
```
Yii::setAlias('foo', 'path/to/foo');
Yii::setAlias('@foo', 'path/to/foo');
```
获取根别名
`$alias`的根别名，就是`@`加上第一个`/`之间地内容，以`$root`表示。这里可以看出，别名是分层次的。下面3个语句的根别名都是`@foo`
```
Yii::setAlias('@foo', 'path/to/some/where');
Yii::setAlias('@foo/bar', 'path/to/some/where');
Yii::setAlias('@foo/bar/qux', 'path/to/some/where');
```
###### 新定义别名还是删除别名
如果传入的`$path`不是`null`，说明是正常的别名定义。对于正常的别名定义，就是往`BaseYii::$aliases[]`里写入信息。而如果`$path`为`null`，说明是要删除别名:
```
// 定义别名@foo
Yii::setAlias('@foo', 'path/to/some/where');

// 删除别名@foo
Yii::setAlias('@foo', null);
```
###### 解析`$path`
对于新定义别名，既然`$path`不为`null`，那么先进行解析：如果`$path`以`@`打头，说明这也是一个别名，则调用`Yii::getAlias()`，并将解析后的结果作为新的`$path`；如果`$path`不以`@`打头，说明是一个正常的path或URL，那么去除`$path` 末尾的`/`和`\\` 。
###### 别名的写入
对于全新的别名，也即其根别名是新的，`BaseYii::aliases[$root]`不存在。那么全新别名的写入分两种情况：如果全新别名本身就是根别名，那么直接`BaseYii::aliases[$alias] = $path`；而如果全新的别名并非是一个根别名，即形如`@foo/bar`带有二级、三级等路径的，`BaseYii::aliases[$root] = [$alias => $path]`。比如:
```
// BaseYii::aliases['@foo'] = ['@foo/bar' => 'path/to/foo/bar']
Yii::setAlias('@foo/bar', 'path/to/foo/bar');

// BaseYii::aliases['@qux'] = 'path/to/qux'
Yii::setAlias('@qux', 'path/to/qux');
```
而对于根别名已经存在的别名，在写入时，就要考虑覆盖、新增的问题了:
```
// 初始 BaseYii::aliases['@foo'] = 'path/to/foo'
Yii::setAlias('@foo', 'path/to/foo');

// 直接覆盖 BaseYii::aliases['@foo'] = 'path/to/foo2'
Yii::setAlias('@foo', 'path/to/foo2');

/**
* 新增
* BaseYii::aliases['@foo'] = [
*     '@foo/bar' => 'path/to/foo/bar',
*     '@foo' => 'path/to/foo2',
* ];
*/
Yii::setAlias('@foo/bar', 'path/to/foo/bar');

// 初始 BaseYii::aliases['@bar'] = ['@bar/qux' => 'path/to/bar/qux'];
Yii::setAlias('@bar/qux', 'path/to/bar/qux');

// 直接覆盖 BaseYii::aliases['@bar'] = ['@bar/qux' => 'path/to/bar/qux2'];
Yii::setAlias('@bar/qux', 'path/to/bar/qux2');

/**
* 新增
* BaseYii::aliases['@bar'] = [
*     '@bar/foo' => 'path/to/bar/foo',
*     '@bar/qux' => 'path/to/bar/qux2',
* ];
*/
Yii::setAlias('@bar/foo', 'path/to/bar/foo');
```
注意如果根别名对应的是一个数组，在新增、覆盖后，Yii会调用PHP的`krsort()`把数组按照键值重新逆向排序。这可以有效确保长的别名会放在短的类以别名前面， 比如，`@foo/bar/qux`和`@foo/bar`同样被放在根别名`@foo`之下，但长的那个，会被放在前面。
###### 别名的删除
传入的`$path`为`null`表示要删除别名。Yii使用PHP的`unset()`注销`BaseYii::$aliases[]`数组中的对应元素，达到删除别名的目的。注意删除别名后，不需要调用`krsort()`对数组进行处理。
##### 别名的解析过程
与定义过程使用`Yii::setAlias()`相对应，别名的解析过程使用`Yii::getAlias()`，实际代码在`BaseYii::getAlias()`中:
```
public static function getAlias($alias, $throwException = true)
{
    // 一切不以@打头的别名都是无效的
    if (strncmp($alias, '@', 1)) {
        return $alias;
    }

    // 先确定根别名 $root
    $pos = strpos($alias, '/');
    $root = $pos === false ? $alias : substr($alias, 0, $pos);

    // 从根别名开始找起，如果根别名没找到，一切免谈
    if (isset(static::$aliases[$root])) {
        if (is_string(static::$aliases[$root])) {
            return $pos === false ? static::$aliases[$root] :
                static::$aliases[$root] . substr($alias, $pos);
        } else {
            // 由于写入前使用了 krsort() 所以，较长的别名会被先遍历到。
            foreach (static::$aliases[$root] as $name => $path) {
                if (strpos($alias . '/', $name . '/') === 0) {
                    return $path . substr($alias, strlen($name));
                }
            }
        }
    }

    if ($throwException) {
        throw new InvalidParamException("Invalid path alias: $alias");
    } else {
        return false;
    }
}
```
别名的解析过程相对简单：  
* 先按根别名找到可能保存别名的分支。
* 遍历这个分支下的所有树叶。由于之前叶子（别名）是按键值逆排序的，所以优先匹配长别名。
* 将找到的最长匹配别名替换成其所对应的值，再接上`@alias`的后半截，成为新的别名。

别名的解析过程可以这么看:
```
// 无效的别名，别名必须以@打头，别名不能放在中间
// 但是语句不会出错，会认为这是一个路径，一字不变的路径： path/to/@foo/bar
Yii::getAlias('path/to/@foo/bar');

// 定义 @foo @foo/bar @foo/bar/qux 3个别名
Yii::setAlias('@foo', 'path/to/foo');
Yii::setAlias('@foo/bar', 'path/2/bar');
Yii::setAlias('@foo/bar/qux', 'path/to/qux');

// 找不到 @foobar根别名，抛出异常
Yii::getAlias('@foobar/index.php');

// 匹配@foo，相当于 path/to/foo/qux/index.php
Yii::getAlias('@foo/qux/index.php');

// 匹配@foo/bar/qux，相当于 path/to/qux/2/index.php
Yii::getAlias('@foo/bar/qux/2/index.php');

// 匹配@foo/bar，相当于 path/to/bar/2/2/index.php
Yii::getAlias('@foo/bar/2/index.php');
```
#### 小结
回顾上面的内容，我们有这么几个要点：  
* 别名需在使用前定义，因此通常来讲，定义别名应当在放在应用的初始化阶段。
* 别名必然以`@`打头。
* 别名的定义可以使用之前已经定义过的别名。
* 别名在储存时，至多只分成两级，第一级的键是根别名。第二级别名的键是完整的别名，而不是去除根别名后剩下的所谓的“二级”别名。
* Yii通过分层的树结构来保存别名最主要是为高效检索作准备。
* 很多地方可以直接使用别名，而不用调用 Yii::getAlias() 转换成真实的路径或URL。
* 别名解析时，优先匹配较长的别名。
* Yii预定义了许多常用的别名供编程时使用。
* 使用别名时，要将别名放在最前面，不能放在中间。
