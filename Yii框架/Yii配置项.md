说到配置项，读者朋友们第一反应是不是Yii的配置文件？这是一段配置文件的代码:
```
return [
    'id' => 'app-frontend',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'controllerNamespace' => 'frontend\controllers',

    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=yii2advanced',
            'username' => 'root',
            'password' => '',
            'charset' => 'utf8',
        ],
        ... ...
        'cache' => [
            'class' => 'yii\caching\MemCache',
            'servers' => [
                [
                    'host' => 'cache1.digpage.com',
                    'port' => 11211,
                    'weight' => 60,
                ],
                [
                    'host' => 'cache2.digpage.com',
                    'port' => 11211,
                    'weight' => 40,
                ],
            ],
        ],
    ],

    'params' => [...],
];
```
Yii中许多地方都要用到配置项，Yii应用自身和其他几乎一切类对象的创建、初始化、配置都要用到配置项。配置项是针对对象而言的，也就是说，配置项一定是用于配置某一个对象，用于初始化或配置对象的属性。关于属性的有关内容，请查看[属性（Property）]()。
#### 配置项的格式
一个配置文件包含了3个部分：  
* 基本信息配置。主要指如`id basePath`等这些应用的基本信息，主要是一些简单的字符串。
* components配置。配置文件的主体，也是我们接下来要讲的配置项。
* params配置。主要是提供一些全局参数。
我们一般讲的配置项是指component配置项及里面的子项。 简单来讲，一个配置项采用下面的格式:
```
[
    'class' => 'path\to\ClassName',
    'propertyName' => 'propertyValue',
    'on eventName' => $eventHandler,
    'as behaviorName' => $behaviorConfig,
]
```
作为配置项：
* 配置项以数组进行组织。
* `class` 数组元素表示将要创建的对象的完整类名。
* `propertyName` 数组元素表示指定为`propertyName`属性的初始值为`$propertyValue`。
* `on eventName` 数组元素表示将`$eventHandler`绑定到对象的`eventName`事件中。
* `as behaviorName` 数组元素表示用`$behaviorConfig`创建一个行为，并注入到对象中。这里的`$behaviroConfig`也是一个配置项；
* 配置项可以嵌套。
其中，`class`元素仅在特定的情况下可以没有。就是使用配置数组的时候，其类型已经是确定的。这往往是用于重新配置一个已经存在的对象，或者是在创建对象时，使用了`new`或`Yii::createObject()`指定了类型。除此以外的大多数情况`class`都是配置数组的必备元素:
```
// 使用 new 时指定了类型，配置数组中就不应再有 class 元素
$connection = new \yii\db\Connection([
    'dsn' => $dsn,
    'username' => $username,
    'password' => $password,
]);

// 使用 Yii::createObject()时，如果第一个参数指定了类型，也不应在配置数
// 组中设定 class
$db = Yii::createObject('yii\db\Connection', [
     'dsn' => 'mysql:host=127.0.0.1;dbname=demo',
     'username' => 'root',
     'password' => '',
     'charset' => 'utf8',
]);

// 对现有的对象重新配置时，也不应在配置数组中设定 class
Yii::configure($db, [
     'dsn' => 'mysql:host=127.0.0.1;dbname=demo',
     'username' => 'root',
     'password' => '',
     'charset' => 'utf8',
]);
```
#### 配置项产生作用的原理
从[环境和配置文件](#)部分的内容，我们了解到了一个Yii应用，特别是高级模版应用，是具有许多个配置文件的，这些配置文件在入口脚本`index.php`中被引入，然后按照一定的规则合并成一个配置数组`$config`并用于创建Application对象。
```
$application = new yii\web\Application($config);
```
在`yii\web\Application`中，会调用父类的构造函数`yii\base\Application::__construct($config)`，来创建Web Application。在这个构造函数中:
```
public function __construct($config = [])
{
    Yii::$app = $this;
    $this->setInstance($this);

    $this->state = self::STATE_BEGIN;

    // 预处理配置项
    $this->preInit($config);

    $this->registerErrorHandler($config);

    // 使用 yii\base\Component::__construct() 完成构建
    Component::__construct($config);
}
```
可以看到，其实分成两步，一是对`$config`进行预处理， 二是使用`yii\base\Component::__construct($config)`进行构建。
##### 配置项预处理
预处理配置项的`yii\base\Application::preInit()`方法，完整地看看这个方法和有关的属性:
```
// basePath属性，由Application的父类yii\base\Module定义，并提供getter和setter
private $_basePath;

// runtimePath属性和vendorPath属性，Application都为其定义了getter和setter。
private $_runtimePath;
private $_vendorPath;

// 还有一个timeZone属性，Application为其提供了getter和setter，但不提供存
// 储变量。
// 而是分别调用 PHP 的 date_default_timezone_get() 和
// date_default_timezone_set()

public function preInit(&$config)
{
    // 配置数组中必须指定应用id，这里仅判断，不赋值。
    if (!isset($config['id'])) {
        throw new InvalidConfigException(
            'The "id" configuration for the Application is required.');
    }

    // 设置basePath属性，这个属性在Application的父类 yii\base\Module 中定义。
    // 在完成设置后，删除配置数组中的 basePath 配置项
    if (isset($config['basePath'])) {
        $this->setBasePath($config['basePath']);
        unset($config['basePath']);
    } else {
        throw new InvalidConfigException(
        'The "basePath" configuration for the Application is required.');
    }

    // 设置vendorPath属性，并在设置后，删除$config中的相应配置项
    if (isset($config['vendorPath'])) {
        $this->setVendorPath($config['vendorPath']);
        unset($config['vendorPath']);
    } else {
        // set "@vendor"
        $this->getVendorPath();
    }

    // 设置runtimePath属性，并在设置后，删除$config中的相应配置项
    if (isset($config['runtimePath'])) {
        $this->setRuntimePath($config['runtimePath']);
        unset($config['runtimePath']);
    } else {
        // set "@runtime"
        $this->getRuntimePath();
    }

    // 设置timeZone属性，并在设置后，删除$config中的相应配置项
    if (isset($config['timeZone'])) {
        $this->setTimeZone($config['timeZone']);
        unset($config['timeZone']);
    } elseif (!ini_get('date.timezone')) {
        $this->setTimeZone('UTC');
    }

    // 将coreComponents() 所定义的核心组件配置，与开发者通过配置文件定义
    // 的组件配置进行合并。
    // 合并中，开发者配置优先，核心组件配置起补充作用。
    foreach ($this->coreComponents() as $id => $component) {

        // 配置文件中没有的，使用核心组件的配置
        if (!isset($config['components'][$id])) {
            $config['components'][$id] = $component;

        // 配置文件中有的，但并未指组件的class的，使用核心组件的class
        } elseif (is_array($config['components'][$id]) &&
            !isset($config['components'][$id]['class'])) {
            $config['components'][$id]['class'] = $component['class'];
        }
    }
}
```
从上面的代码可以看出，这个`preInit()`对配置数组`$config`作了以下处理：
* `id`属性是必不可少的。
* 从 `$config`中拿掉了basePath，runtimePath，vendorPath和timeZone4个属性的配置项。当然，也设置了相应的属性。
* 对`$config['components']`配置项进行两方面的补充。一是配置文件中没有的，而核心组件有的，把核心组件的配置信息补充进去。二是配置文件中虽然也有，但没有指定组件的class的，使用核心组件配置信息指定的class。

基于此，我们不难得出如下结论：  
* 有的配置项如`id`是不可少的，有的配置项如`basePath`等不用我们设置也是有默认值的。
* 对于核心组件，我们不配置也可以使用。
* 核心组件的ID是提前安排好的，没有充足的理由一般不要改变他，否则以后接手的人会骂你的。
* 核心组件可以不指明`class`，默认会使用预先安排的类型。

对于核心组件，不同的应用有不同的安排，这个我们可以看看，大致了解下，具体在于各应用的`coreComponents()`中定义:
```
// yii\base\Application 的核心组件
public function coreComponents()
{
    return [
        'log' => ['class' => 'yii\log\Dispatcher'],             // 日志组件
        'view' => ['class' => 'yii\web\View'],                  // 视图组件
        'formatter' => ['class' => 'yii\i18n\Formatter'],       // 格式组件
        'i18n' => ['class' => 'yii\i18n\I18N'],                 // 国际化组件
        'mailer' => ['class' => 'yii\swiftmailer\Mailer'],      // 邮件组件
        'urlManager' => ['class' => 'yii\web\UrlManager'],      // url管理组件
        'assetManager' => ['class' => 'yii\web\AssetManager'],  // 前端资源管理组件
        'security' => ['class' => 'yii\base\Security'],         // 安全组件
    ];
}

// yii\web\Application 的核心组件，在基类的基础上加入Web应用必需的组件
public function coreComponents()
{
    return array_merge(parent::coreComponents(), [
        'request' => ['class' => 'yii\web\Request'],            // HTTP请求组件
        'response' => ['class' => 'yii\web\Response'],          // HTTP响应组件
        'session' => ['class' => 'yii\web\Session'],            // session组件
        'user' => ['class' => 'yii\web\User'],                  // 用户管理组件
        'errorHandler' => ['class' => 'yii\web\ErrorHandler'],  // 错误处理组件
    ]);
}

// yii\console\Application 的核心组件，
public function coreComponents()
{
    return array_merge(parent::coreComponents(), [
        'request' => ['class' => 'yii\console\Request'],        // 命令行请求组件
        'response' => ['class' => 'yii\console\Response'],      // 命令行响应组件
        'errorHandler' => ['class' => 'yii\console\ErrorHandler'], // 错误处理组件
    ]);
}
```
这些我们大致有个印象就够了，不用刻意去记住，用着用着你就自然记住了。
##### 使用配置数组构造应用
在使用`preInit()`完成配置数组的预处理之后，Application构造函数又直接调用`yii\base\Component::__construct()`来构造Application对象。  
结果这个`yii\base\Component::__construct()`也是个推委扯皮的家伙，他根本就没自己定义。而是直接继承了父类的`yii\base\Object::__construct()`。因此，Application构造函数的最后一步， 实际上调用的是`yii\base\Object::__construct($config)`。  
只是这里有两类特殊的配置项需要注意，就是以`on *`打头的事件和以`as *`打头的行为。对于事件行为，可以阅读[事件（Event）]()和[行为（Behavior）]()部分的内容。Yii对于这两类配置项的处理，是在`yii\base\Component::__set()`中完成的，从Component开始，才支持事件和行为。具体处理的代码如下:
```
public function __set($name, $value)
{
    $setter = 'set' . $name;
    if (method_exists($this, $setter)) {
        $this->$setter($value);
        return;

    // 'on ' 打头的配置项在这里处理
    } elseif (strncmp($name, 'on ', 3) === 0) {

        // 对于 'on event' 配置项，将配置值作为事件 handler 绑定到 evnet 上去
        $this->on(trim(substr($name, 3)), $value);
        return;

    // 'as ' 打头的配置项在这里处理
    } elseif (strncmp($name, 'as ', 3) === 0) {

        // 对于 'as behavior' 配置项，将配置值作为创建Behavior的配置，创
        // 建后绑定为 behavior
        $name = trim(substr($name, 3));
        $this->attachBehavior($name, $value instanceof Behavior ? $value
            : Yii::createObject($value));
        return;
    } else {
        $this->ensureBehaviors();
        foreach ($this->_behaviors as $behavior) {
            if ($behavior->canSetProperty($name)) {
                $behavior->$name = $value;
                return;
            }
        }
    }
    if (method_exists($this, 'get' . $name)) {
        throw new InvalidCallException('Setting read-only property: ' .
            get_class($this) . '::' . $name);
    } else {
        throw new UnknownPropertyException('Setting unknown property: '
            . get_class($this) . '::' . $name);
    }
}
```
从上面的代码中可以看到，对于`on event`形式配置项，Yii视配置值为一个事件handler，绑定到`event`上。而对于`as behavior`形式的配置项，视配置值为一个Behavior，注入到当前实例中，并冠以`behavior`的名称。

引用：[配置项（Configuration） — 深入理解Yii2.0](http://www.digpage.com/configuration.html)
