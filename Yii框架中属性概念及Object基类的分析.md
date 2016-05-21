#### 属性（Property）
属性用于表征类的状态，从访问的形式上看，属性与成员变量没有区别。你能一眼看出`$object->foo`中的`foo`是成员变量还是属性么？显然不行。但是，成员变量是就类的结构构成而言的概念，而属性是就类的功能逻辑而言的概念，两者紧密联系又相互区别。比如，我们说`People`类有一个成员变量`int $age`，表示年龄。那么这里年龄就是属性，`$age`就是成员变量。  
举个更学术化点的例子，与非门:
```
class NotAndGate extends Object{
    private $_key1;
    private $_key2;
    public function setKey1($value){
        $this->_key1 = $value;
    }
    public function setKey2($value){
        $this->_key2 = $value;
    }
    public function getOutput(){
        if (!$this->_key1 || !$this->_key2)
            return true;
        else if ($this->_key1 && $this->_key2)
            return false;
    }
}
```
与非门有两个输入，当两个输入都为真时，与非门的输出为假，否则，输出为真。上面的代码中，与非门类有两个成员变量，`$key1`和`$key2`。但是有3个属性，表示2个输入的`key1`和`key2`，以及表示输出的`output`。
成员变量和属性的区别与联系在于：  
* 成员变量是一个“内”概念，反映的是类的结构构成。属性是一个“外”概念，反映的是类的逻辑意义。
* 成员变量没有读写权限控制，而属性可以指定为只读或只写，或可读可写。（通常需要将成员变量设置为private属性，即私有）
* 成员变量不对读出作任何后处理，不对写入作任何预处理，而属性则可以。（简单理解属性为`getter/setter`操作）
* public成员变量可以视为一个可读可写、没有任何预处理或后处理的属性。而private成员变量由于外部不可见，与属性“外”的特性不相符，所以不能视为属性。
* 虽然大多数情况下，属性会由某个或某些成员变量来表示，但属性与成员变量没有必然的对应关系，比如与非门的`output`属性，就没有一个所谓的`$output`成员变量与之对应。

在Yii中，由`yii\base\Object`提供了对属性的支持，因此，如果要使你的类支持属性，必须继承自`yii\base\Object`。Yii中属性是通过PHP的魔法函数`__get()``__set()`来产生作用的。下面的代码是`yii\base\Object`类对于`__get()`和`__set()`的定义:
```
public function __get($name)              // 这里$name是属性名
{
    $getter = 'get' . $name;              // getter函数的函数名
    if (method_exists($this, $getter)) {
        return $this->$getter();          // 调用了getter函数
    } elseif (method_exists($this, 'set' . $name)) {
        throw new InvalidCallException('Getting write-only property: '
                    . get_class($this) . '::' . $name);
    } else {
        throw new UnknownPropertyException('Getting unknown property: '
                    . get_class($this) . '::' . $name);
    }
}

// $name是属性名，$value是拟写入的属性值
public function __set($name, $value)
{
    $setter = 'set' . $name;             // setter函数的函数名
    if (method_exists($this, $setter)) {
        $this->$setter($value);          // 调用setter函数
    } elseif (method_exists($this, 'get' . $name)) {
        throw new InvalidCallException('Setting read-only property: '
                        . get_class($this) . '::' . $name);
    } else {
        throw new UnknownPropertyException('Setting unknown property: '
                        . get_class($this) . '::' . $name);
    }
}

```
#### 实现属性的步骤
我们都知道，`__get()``__set()`方法主要是来对私有成员变量的获取和赋值，以及在读写对象的一个不存在的成员变量时自动调用。Yii正是利用这点，提供对属性的支持的。从上面的代码中，可以看出，如果访问一个对象的某个属性，Yii会调用名为get属性名()的函数。如，`SomeObject->Foo`，会自动调用`SomeObject->getFoo()`。如果修改某一属性，会调用相应的setter函数。如，`SomeObject->Foo = $someValue`，会自动调用`SomeObject->setFoo($someValue)`。  
因此，要实现属性，通常有三个步骤：  
* 继承自`yii\base\Object`。
* 声明一个用于保存该属性的私有成员变量。（易于维护，安全）
* 提供getter或setter函数，或两者都提供，用于访问、修改上面提到的私有成员变量。如果只提供了getter，那么该属性为只读属性，只提供了setter，则为只写。

如下的Post类，实现了可读可写的属性title:
```
class Post extends yii\base\Object    // 第一步：继承自 yii\base\Object
{
    private $_title;                 // 第二步：声明一个私有成员变量
    public function getTitle()       // 第三步：提供getter和setter
    {
           return $this->_title;
    }
    public function setTitle($value)
    {
           $this->_title = trim($value);
    }
}
```
由于`__get()`和`__set()`是在遍历所有成员变量，找不到匹配的成员变量（或为私有变量）时才被调用。因此，其效率天生地低于使用成员变量的形式。在一些表示数据结构、数据集合等简单情况下，且不需读写控制等，可以考虑使用成员变量作为属性，这样可以提高一点效率。  
另外一个提高效率的小技巧就是：使用`$pro = $object->getPro()`来代替`$pro = $object->pro`，用`$objcect->setPro($value)`来代替`$object->pro = $value`。这在功能上是完全一样的效果，但是避免了使用`__get()`和`__set()`，相当于绕过了遍历的过程。  
值得注意的是：  
* 由于自动调用`__get()``__set()`的时机仅仅发生在访问不存在的（或私有）成员变量时。因此，如果定义了成员变量`public $title`那么，就算定义了`getTitle()``setTitle()`，他们也不会被调用。因为`$post->title`时，会直接指向该`pulic $title`，`__get()``__set()`是不会被调用的。从根上就被切断了。
* 由于PHP对于类方法不区分大小写，即大小写不敏感，`$post->getTitle()`和`$post->gettitle()`是调用相同的函数。因此，`$post->title`和`$post->Title`是同一个属性。即属性名也是不区分大小写的。
* 由于`__get()`` __set()`都是public的，无论将`getTitle()``setTitle()`声明为public,private,protected，都没有意义，外部同样都是可以访问。所以，所有的属性都是public的。
* 由于`__get()``__set()`都不是static的，因此，没有办法使用static的属性。

#### Object的其他与属性相关的方法
除了`__get()``__set()`之外，`yii\base\Object`还提供了以下方法便于使用属性：
* `__isset()`用于测试属性值是否不为`null`，在`isset($object->property)`时被自动调用。注意该属性要有相应的getter。
* `__unset()`用于将属性值设为`null`，在`unset($object->property)`时被自动调用。注意该属性要有相应的setter。
* `hasProperty()`用于测试是否有某个属性。即，定义了getter或setter。如果`hasProperty()`的参数`$checkVars = true`（默认为true），那么只要具有同名的成员变量也认为具有该属性，如前面提到的`public $title`。
* `canGetProperty()`测试一个属性是否可读，参数`$checkVars`的意义同上。只要定义了getter，属性即可读。同时，如果`$checkVars`为`true`。那么只要类定义了成员变量，不管是public，private还是protected，都认为是可读。
* `canSetProperty()`测试一个属性是否可写，参数`$checkVars`的意义同上。只要定义了setter，属性即可写。同时，在`$checkVars`为`ture`。那么只要类定义了成员变量，不管是public，private还是protected，都认为是可写。

#### Object和Component
`yii\base\Component`继承自`yii\base\Object`，因此，他也具有属性等基本功能。  
但是，由于Componet还引入了事件、行为，因此，它并非简单继承了Object的属性实现方式，而是基于同样的机制，重载了`__get()``__set()`等函数。但从实现机制上来讲，是一样的。这个不影响理解。  
官方将Yii定位于一个基于组件的框架。可见组件这一概念是Yii的基础。如果你有兴趣阅读Yii的源代码或是API文档，你将会发现，Yii几乎所有的核心类都派生于（继承自）`yii\base\Component` 。  
在Yii1.1时，就已经有了component了，那时是CComponent。Yii2将Yii1.1中的CComponent拆分成两个类：`yii\base\Object`和`yii\base\Component`。  
其中，Object比较轻量级些，通过getter和setter定义了类的属性（property）。Component派生自Object，并支持事件（event）和行为（behavior）。因此，Component类具有三个重要的特性：  
* 属性（property）
* 事件（event）
* 行为（behavior）

相信你或多或少了解过，这三个特性是丰富和拓展类功能、改变类行为的重要切入点。因此，Component在Yii中的地位极高。  
在提供更多功能、更多便利的同时，Component由于增加了event和behavior这两个特性，在方便开发的同时，也牺牲了一定的效率。如果开发中不需要使用event和behavior这两个特性，比如表示一些数据的类。那么，可以不从Component继承，而从Object继承。典型的应用场景就是如果表示用户输入的一组数据，那么，使用Object。而如果需要对对象的行为和能响应处理的事件进行处理，毫无疑问应当采用Component。从效率来讲，Object更接近原生的PHP类，因此，在可能的情况下，应当优先使用Object。
#### Object的配置方法
Yii提供了一个统一的配置对象的方式。这一方式贯穿整个Yii。Application对象的配置就是这种配置方式的体现:
```
$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../../common/config/main.php'),
    require(__DIR__ . '/../../common/config/main-local.php'),
    require(__DIR__ . '/../config/main.php'),
    require(__DIR__ . '/../config/main-local.php')
);
$application = new yii\web\Application($config);
```
`$config`看着复杂，但本质上就是一个各种配置项的数组。Yii中就是统一使用数组的方式对对象进行配置，而实现这一切的关键就在`yii\base\Object`定义的构造函数中:
```
public function __construct($config = [])
{
    if (!empty($config)) {
        Yii::configure($this, $config);
            
    }
    $this->init();
}
```
所有`yii\base\Object`的构建流程是：  
* 构建函数以`$config`数组为参数被自动调用。
* 构建函数调用`Yii::configure()`对对象进行配置。
* 最后，构造函数调用对象的`init()`方法进行初始化。

数组配置对象的秘密在`Yii::configure()`中，但说破了其实也没有什么神奇的:
```
public static function configure($object, $properties)
{
    foreach ($properties as $name => $value) {
         $object->$name = $value;
    }
    return $object;
}
```
配置的过程就是遍历`$config`配置数组，将数组的键作为属性名，以对应的数组元素的值对对象的属性赋值。因此，实现Yii这一统一的配置方式的要点有：  
* 继承自`yii\base\Object`。
* 为对象属性提供setter方法，以正确处理配置过程。
* 如果需要重载构造函数，请将`$config`作为该构造函数的最后一个参数，并将该参数传递给父构造函数。
* 重载的构造函数的最后，一定记得调用父构造函数。
* 如果重载了`yii\base\Object::init()`函数，注意一定要在重载函数的开头调用父类的`init()`。

只要实现了以上要点，就可以使得你编写的类可以按照Yii约定俗成的方式进行配置。这在编写代码的过程中，带来许多便利。但是如果配置数组的某个配置项，也是一个数组，这怎么办？如果某个对象的属性，也是一个对象，而非一个简单的数值或字符串时，又怎么办？  
这两个问题，其实是同质的。如果一个对象的属性，是另一个对象，就像Application里会引入诸多的Component一样，这是很常见的。如后面会看到的`$app->request`中的`request`属性就是一个对象。那么，在配置`$app`时，必然要配置到这个`reqeust`对象。既然`request`也是一个对象，那么他的配置要是按照Yii的规矩来，也就是用一个数组来配置它。因此，上面提到的这两个问题，其实是同质的。  
那么，怎么实现呢？秘密在于setter函数。由于`$app`在进行配置时，最终会调用`Yii::configure()`函数。该函数又不区分配置项是简单的数值还是数组，就直接使用`$object->$name = $value`完成属性的赋值。那么，对于对象属性，其配置值`$value`是一个数组，为了使其正确配置。你需要在其setter函数上做出正确的处理方式。Yii应用`yii\web\Application`就是依靠定义专门的setter函数，实现自动处理配置项的。比如，我们在Yii的配置文件中，可以看到一个配置项`components`，一般情况下，他的内容是这样的:
```
'components' => [
        'request' => [
                // !!! insert a secret key in the following (if it is empty) -
                // this is required by cookie validation
                'cookieValidationKey' => 'v7mBbyetv4ls7t8UIqQ2IBO60jY_wf_U',
        ],
        'user' => [
                'identityClass' => 'common\models\User',
                'enableAutoLogin' => true,
        ],
        'log' => [
                'traceLevel' => YII_DEBUG ? 3 : 0,
                'targets' => [
                [
                       'class' => 'yii\log\FileTarget',
                      'levels' => ['error', 'warning'],
                ],
                ],
        ],
        'errorHandler' => [
                'errorAction' => 'site/error',
        ],
],
```
这是一个典型嵌套配置数组。那么Yii是如何把他们配置好的呢？ Yii定义了一个名为`setComponents`的setter函数。当然，Yii并未将该函数放在`yii\web\Application`里，而是放在父类`yii\di\ServiceLocator`里面。（具体可以查看服务定位器（Service Locator））
```
public function setComponents($components)
{
    foreach ($components as $id => $component) {
        $this->set($id, $component);
    }
}
```
#### Object和属性总结
从`yii\base\Object::__construct()`来看，对于所有Object，包括Component的属性，都经历这么4个阶段：  
1. 预初始化阶段。这是最开始的阶段，就是在构造函数`__construct()`的开头可以设置property的默认值。
2. 对象配置阶段。也就是前面提到构造函数调用`Yii::configure($this, $config)`阶段。这一阶段可以覆盖前一阶段设置的property的默认值，并补充没有默认值的参数，也就是必备参数。`$config`通常由外部代码传入或者通过配置文件传入。
3. 后初始化阶段。也就是构造函数调用`init()`成员函数。通过在`init()`写入代码，可以对配置阶段设置的值进行检查，并规范类的property。
4. 类方法调用阶段。前面三个阶段是不可分的，由类的构造函数一口气调用的。也就是说一个类一但实例化，那么就至少经历了前三个阶段。 此时，该对象的状态是确定且可靠的，不存在不确定的property。所有的属性要么是默认值，要么是传入的配置值，如果传入的配置有误或者冲突，那么也经过了检查和规范。
