使用行为（behavior）可以在不修改现有类的情况下，对类的功能进行扩充。通过将行为绑定到一个类，可以使类具有行为本身所定义的属性和方法，就好像类本来就有这些属性和方法一样。而且不需要写一个新的类去继承或包含现有类。  
Yii中的行为，其实是`yii\base\Behavior`类的实例，只要将一个Behavior实例绑定到任意的`yii\base\Component`实例上，这个Component就可以拥有该Behavior所定义的属性和方法了。而如果将行为与事件关联起来，可以玩的花样就更多了。  
但有一点需要注意，Behavior只能与Component类绑定。如果你写了一个类，需要使用到行为，那么就果断地继承自`yii\base\Component`。同时，行为单独靠Behavior一方是实现不了的。为了支持Behavior，Yii对于`yii\base\Component`也进行了精心设计，这两者共同配合，才有了神奇的行为。
#### 使用行为
一个绑定了行为的类，表现起来是这样的:
```
// Step 1: 定义一个将绑定行为的类
class MyClass extends yii\base\Component
{
    // 空的
}

// Step 2: 定义一个行为类，他将绑定到MyClass上
class MyBehavior extends yii\base\Behavior
{
    // 行为的一个属性
    public $property1 = 'This is property in MyBehavior.';

    // 行为的一个方法
    public function method1()
    {
        return 'Method in MyBehavior is called.';
    }
}

$myClass = new MyClass();
$myBehavior = new MyBehavior();

// Step 3: 将行为绑定到类上
$myClass->attachBehavior('myBehavior', $myBehavior);

// Step 4: 访问行为中的属性和方法，就和访问类自身的属性和方法一样
echo $myClass->property1;
echo $myClass->method1();
```
上面的代码你不用全都看懂，虽然你可能已经用脚趾头猜到了这些代码的意思，但这里你只需要记住行为中的属性和方法可以被所绑定的类像访问自身的属性和方法一样直接访问就OK了。代码中，`$myClass`是没有`property1 method()`成员的。这俩是`$myBehavior`的成员。但是，通过`attachBehavior()`将行为绑定到对象之后，`$myCalss`就好像练成了吸星大法、化功大法，表现的财大气粗，将别人的属性和方法都变成了自己的。  
另外，从上面的代码中，你还要掌握使用行为的大致流程：  
* 从`yii\base\Component`派生自己的类，以便使用行为；
* 从`yii\base\Behavior`派生自己的行为类，里面定义行为涉及到的属性、方法；
* 将Component和Behavior绑定起来；
* 像使用Component自身的属性和方法一样，尽情使用行为中定义的属性和方法。

#### 行为的要素
我们提到了行为只是`yii\base\Behavior`类的实例。那么这个类究竟有什么秘密呢？其实说破了也没有什么的他只是一个简单的封装而已，非常的简单:
```
class Behavior extends Object
{
    // 指向行为本身所绑定的Component对象
    public $owner;

    // Behavior 基类本身没用，主要是子类使用，重载这个函数返回一个数组表
    // 示行为所关联的事件
    public function events()
    {
        return [];
    }

    // 绑定行为到 $owner
    public function attach($owner)
    {
        ... ...
    }

    // 解除绑定
    public function detach()
    {
        ... ...
    }
}
```
这就是Behavior的全部代码了，是不是很简单？Behavior类的要素的确很简单：  
* `$owner`成员变量，用于指向行为的依附对象；
* `events()`用于表示行为所有要响应的事件；
* `attach()`用于将行为与Component绑定起来；
* `deatch()`用于将行为从Component上解除。

下面分别进行讲解。
##### 行为的依附对象
`yii\base\Behavior::$owner`指向的是Behavior实例本身所依附的对象。这是行为中引用所依附对象的唯一手段了。通过这个`$owner`，行为才能访问所依附的Component，才能将本身的方法作为事件handler绑定到Component上。  
`$owner`由`yii\base\Behavior::attach()`进行赋值。也就是在将行为绑定到某个Component时，`$owner`就已经名花有主了。一般情况下，不需要你自己手动去指定`$owner`的值， 在调用`yii\base\Component::attachBehavior()`将行为与对象绑定时，Component会自动地将`$this`作为参数，调用`yii\base\Behavior::attach()`。  
有一点需要格外注意，由于行为从本质来讲是一个PHP类，其方法就是类方法，就是成员函数。所以，在行为的方法中，`$this`引用的是行为本身，试图通过`$this`来访问行为所依附的Component是行不通的。正确的方法是通过`yii\base\Behavior::$owner`来访问Component。
##### 行为所要响应的事件
行为与事件结合后，可以在不对类作修改的情况下，补充类在事件触发后的各种不同反应。为此，只需要重载`yii\base\Behavior::events()`方法，表示这个行为将对类的何种事件进行何种反馈即可:
```
namespace app\Components;

use yii\db\ActiveRecord;
use yii\base\Behavior;

class MyBehavior extends Behavior
{
    // 重载events() 使得在事件触发时，调用行为中的一些方法
    public function events()
    {
        // 在EVENT_BEFORE_VALIDATE事件触发时，调用成员函数 beforeValidate
        return [
            ActiveRecord::EVENT_BEFORE_VALIDATE => 'beforeValidate',
        ];
    }

    // 注意beforeValidate 是行为的成员函数，而不是绑定的类的成员函数。
    // 还要注意，这个函数的签名，要满足事件handler的要求。
    public function beforeValidate($event)
    {
        // ...
    }
}
```
上面的代码中，`events()`返回一个数组，表示所要做出响应的事件，上例中的事件是`ActiveRecord::EVENT_BEFORE_VALIDATE`，以数组的键来表示，而数组的值则表示做好反应的事件handler，上例中是`beforeValidate()`，事件handler可以是以下形式：  
* 字符串，表示行为类的方法，如上面的例就是这种情况。这个是与事件handler不同的，事件handler中使用字符串时，是表示PHP全局函数，而这里表示行为类内部的方法。
* 一个对象或类的成员函数，以数组的形式，如`[$object, 'methodName']`。这个与事件handler是一致的。
* 一个匿名函数。

##### 行为的绑定与解除
说到绑定与解除，这意味着这个事情有2方，行为和Component。单独一方是没有绑定或解除的说法的。对于绑定和解除，Behavior分别使用`attach()`和`detach()`来实现。
#### 定义一个行为
定义一个行为，就是准备好要注入到现有类中去的属性和方法，这些属性和方法要写到一个`yii\base\Behavior`类中。所以，定义一个行为，就是写一个Behavior的子类，子类中包含了所要注入的属性和方法:
```
namespace app\Components;

use yii\base\Behavior;

class MyBehavior extends Behavior
{
    public $prop1;

    private $_prop2;
    private $_prop3;
    private $_prop4;

    public function getProp2()
    {
        return $this->_prop2;
    }

    public function setProp3($value)
    {
        $this->_prop3 = $value;
    }

    public function foo()
    {
        // ...
    }

    protected function bar()
    {
        // ...
    }
}

```
上面的代码通过定义一个`app\Components\MyBehavior`类而定义一个行为。由于`MyBehavior`继承自`yii\base\Behavior`从而间接地继承自`yii\base\Object`。没错，这是我们的老朋友了。因此，这个类有一个public的成员变量`prop1`，一个只读属性`prop2`，一个只写属性`prop3`，一个public的方法`foo()`。另外，还有一个private的成员变量`$_prop4`，一个protected的方法`bar()`。  
当这MyBehavior与一个Component绑定后，绑定的Component也就拥有了`prop1 prop2`这两个属性和方法`foo()`，因为他们都是`public`的。而`private`的`$_prop4`和`protected`的`bar`就得不到了。至于原因么，后面讲行为注入的原理时解释。
##### 行为的绑定
行为的绑定通常是由Component来发起。有两种方式可以将一个Behavior绑定到一个`yii\base\Component`。一种是静态的方法，另一种是动态的。静态的方法在实践中用得比较多一些。因为一般情况下，在你的代码没跑起来之前，一个类应当具有何种行为，是确定的。动态绑定的方法主要是提供了更灵活的方式，但实际使用中并不多见。
###### 静态方法绑定行为
静态绑定行为，只需要重载`yii\base\Component::behaviors()`就可以了。这个方法用于描述类所具有的行为。如何描述呢？ 使用配置来描述，可以是Behavior类名，也可以是Behavior类的配置数组:
```
namespace app\models;

use yii\db\ActiveRecord;
use app\Components\MyBehavior;

class User extends ActiveRecord
{
    public function behaviors()
    {
        return [
            // 匿名的行为，仅直接给出行为的类名称
            MyBehavior::className(),

            // 名为myBehavior2的行为，也是仅给出行为的类名称
            'myBehavior2' => MyBehavior::className(),

            // 匿名行为，给出了MyBehavior类的配置数组
            [
                'class' => MyBehavior::className(),
                'prop1' => 'value1',
                'prop3' => 'value3',
            ],

            // 名为myBehavior4的行为，也是给出了MyBehavior类的配置数组
            'myBehavior4' => [
                'class' => MyBehavior::className(),
                'prop1' => 'value1',
                'prop3' => 'value3',
            ]
        ];
    }
}
```
还有一个静态的绑定办法，就是通过配置文件来绑定:
```
[
    'as myBehavior2' => MyBehavior::className(),

    'as myBehavior3' => [
        'class' => MyBehavior::className(),
        'prop1' => 'value1',
        'prop3' => 'value3',
    ],
]
```
###### 动态方法绑定行为
动态绑定行为，需要调用`yii\base\Compoent::attachBehaviors()`:
```
$Component->attachBehaviors([
    'myBehavior1' => new MyBehavior,  // 这是一个命名行为
    MyBehavior::className(),          // 这是一个匿名行为
]);
```
这个方法接受一个数组参数，参数的含义与上面静态绑定行为是一样一样的。  
在上面的这些例子中，以数组的键作为行为的命名，而对于没有提供键名的行为，就是匿名行为。  
对于命名的行为，可以调用`yii\base\Component::getBehavior()`来取得这个绑定好的行为:
```
$behavior = $Component->getBehavior('myBehavior2');
```
对于匿名的行为，则没有办法直接引用了。但是，可以获取所有的绑定好的行为:
```
$behaviors = $Component->getBehaviors();
```
##### 绑定的内部原理
只是重载一个`yii\base\Component::behaviors()`就可以这么神奇地使用行为了？ 这只是冰山的一角，实际上关系到绑定的过程，有关的方面有：
* `yii\base\Component::behaviors()`
* `yii\base\Component::ensureBehaviors()`
* `yii\base\Component::attachBehaviorInternal()`
* `yii\base\Behavior::attach()`

4个方法中，Behavior只占其一，更多的代码，是在Component中完成的。  
`yii\base\Component::behaviors()`上面讲静态方法绑定行为时已经提到了，就是返回一个数组用于描述行为。那么`yii\base\Component::ensuerBehaviors()`呢？  
这个方法会在Component的诸多地方调用`__get() __set() __isset() __unset() __call() canGetProperty() hasMethod() hasEventHandlers() on() off()`等用到，看到这么多是不是头疼？一点都不复杂，一句话，只要涉及到类的属性、方法、事件这个函数都会被调用到。  
这么众星拱月，被诸多凡人所需要的`ensureBehaviors()`究竟是何许人也？ 就像名字所表明的，他的作用在于“ensure” 。其实只是确保`behaviors()`中所描述的行为已经进行了绑定而已:
```
public function ensureBehaviors()
{
    // 为null表示尚未绑定
    // 多说一句，为空数组表示没有绑定任何行为
    if ($this->_behaviors === null) {
        $this->_behaviors = [];

        // 遍历 $this->behaviors() 返回的数组，并绑定
        foreach ($this->behaviors() as $name => $behavior) {
            $this->attachBehaviorInternal($name, $behavior);
        }
    }
}
```
这个方法主要是对子类用的，`yii\base\Compoent`没有任何预先注入的行为，所以，这个调用没有用。但是对于子类，你可能重载了`yii\base\Compoent::behaviros()`来预先注入一些行为。那么，这个函数会将这些行为先注入进来。  
从上面的代码中，自然就看到了接下来要说的第三个东东，`yii\base\Component\attachBehaviorInternal()`:
```
private function attachBehaviorInternal($name, $behavior)
{
    // 不是 Behavior 实例，说是只是类名、配置数组，那么就创建出来吧
    if (!($behavior instanceof Behavior)) {
        $behavior = Yii::createObject($behavior);
    }

    // 匿名行为
    if (is_int($name)) {
        $behavior->attach($this);
        $this->_behaviors[] = $behavior;

    // 命名行为
    } else {

        // 已经有一个同名的行为，要先解除，再将新的行为绑定上去。
        if (isset($this->_behaviors[$name])) {
            $this->_behaviors[$name]->detach();
        }
        $behavior->attach($this);
        $this->_behaviors[$name] = $behavior;
    }
    return $behavior;
}
```
首先要注意到，这是一个private成员。其实在Yii中，所有后缀为`*Internal`的方法，都是私有的。这个方法干了这么几件事：
* 如果`$behavior`参数并非是一个`Behavior`实例，就以之为参数，用`Yii::createObject()`创建出来。
* 如果以匿名行为的形式绑定行为，那么直接将行为附加在这个类上。
* 如果是命名行为，先看看是否有同名的行为已经绑定在这个类上，如果有，用后来的行为取代之前的行为。

在`yii\base\Component::attachBehaviorInternal()`中，以`$this`为参数调用了`yii\base\Behavior::attach()`。从而，引出了跟绑定相关的最后一个家伙`yii\base\Behavior::attach()`， 这也是前面我们讲行为的要素时没讲完的。先看看代码:
```
public function attach($owner)
{
    $this->owner = $owner;
    foreach ($this->events() as $event => $handler) {
        $owner->on($event, is_string($handler) ? [$this, $handler] : $handler);
    }
}
```
上面的代码干了两件事：  
* 设置好行为的`$owner`，使得行为可以访问、操作所依附的对象
* 遍历行为中的`events()`返回的数组，将准备响应的事件，通过所依附类的`on()`绑定到类上

说了这么多，关于绑定，做个小结：  
* 绑定的动作从Component发起；
* 静态绑定通过重载`yii\base\Componet::behaviors()`实现；
* 动态绑定通过调用`yii\base\Component::attachBehaviors()`实现；
* 行为还可以通过为`Component`配置`as`配置项进行绑定；
* 行为有匿名行为和命名行为之分，区别在于绑定时是否给出命名。命名行为可以通过其命名进行标识，从而有针对性地进行解除等操作；
* 绑定过程中，后绑定的行为会取代已经绑定的同名行为；
* 绑定的意义有两点，一是为行为设置`$owner`。二是将行为中拟响应的事件的handler绑定到类中去。

##### 解除行为
解除行为只需调用`yii\base\Component::detachBehavior()`就OK了:
```
$Component->detachBehavior('myBehavior2');
```
这样就可以解除已经绑定好的名为`myBehavior2`的行为了。但是，对于匿名行为，这个方法就无从下手了。不过我们可以一不做二不休，解除所有绑定好的行为:
```
$Component->detachBehaviors();
```
这上面两种方法，都会调用到`yii\base\Behavior::detach()`，其代码如下:
```
public function detach()
{
    // 这得是个名花有主的行为才有解除一说
    if ($this->owner) {

        // 遍历行为定义的事件，一一解除
        foreach ($this->events() as $event => $handler) {
            $this->owner->off($event, is_string($handler) ? [$this,
                $handler] : $handler);
        }
        $this->owner = null;
    }
}
```
与`yii\base\Behavior::attach()`相反，解除的过程就是干两件事： 一是将`$owner`设置为`null`，表示这个行为没有依附到任何类上。二是通过Component的`off()`将绑定到类上的事件hanlder解除下来。一句话，善始善终。
##### 行为响应的事件实例
上面的绑定和解除过程，我们看到Yii费了那么大劲，主要就是为了将行为中的事件handler绑定到类中去。在实际编程时，行为用得最多的，也是对于Compoent各种事件的响应。通过行为注入，可以在不修改现有类的代码的情况下，更改、扩展类对于事件的响应和支持。使用这个技巧，可以玩出很炫的花样。而要将行为与Component的事件关联起来，就要通过`yii\base\Behavior::events()`方法。  
上面Behavior基类的代码中，这个方法只是返回了一个空数组，说明不对所依附的Compoent的任何事件产生关联。 但是在实际使用时，往往通过重载这个方法来告诉Yii，这个行为将对Compoent的何种事件，使用哪个方法进行处理。  
比如，Yii自带的`yii\behaviors\AttributeBehavior`类，定义了在一个`ActiveRecord`对象的某些事件发生时，自动对某些字段进行修改的行为。他有一个很常用的子类`yii\behaviors\TimeStampBehavior`用于将指定的字段设置为一个当前的时间戳。常用于表示最后修改日期、上次登陆时间等场景。我们以这个行为为例，来分析行为响应事件的原理。  
在`yii\behaviors\AttributeBehavior::event()`中，代码如下:
```
public function events()
{
    return array_fill_keys(array_keys($this->attributes),
        'evaluateAttributes');
}
```
这段代码将返回一个数组，其键值为`$this->attributes`数组的键值，数组元素的值为成员函数`evaluateAttributes`。  
而在`yii\behaviors\TimeStampBehavior::init()`中，有以下的代码:
```
public function init()
{
    parent::init();

    if (empty($this->attributes)) {
        // 重点看这里
        $this->attributes = [
            BaseActiveRecord::EVENT_BEFORE_INSERT =>
                [$this->createdAtAttribute, $this->updatedAtAttribute],
            BaseActiveRecord::EVENT_BEFORE_UPDATE =>
                $this->updatedAtAttribute,
        ];
    }
}
```
上面的代码重点看的是对于`$this->attributes`的初始化部分。结合上面2个方法的代码，对于`yii\base\Behavior::events()`的返回数组，其格式应该是这样的:
```
return [
    BaseActiveRecord::EVENT_BEFORE_INSERT => 'evaluateAttributes',
    BaseActiveRecord::EVENT_BEFORE_UPDATE => 'evaluateAttributes',
];
```
数组的键值用于指定要响应的事件，这里是`BaseActiveRecord::EVENT_BEFORE_INSERT`和`BaseActiveRecord::EVENT_BEFORE_UPDATE`。数组的值是一个事件handler，如上面的`evaluateAttributes`。  
那么一旦TimeStampBehavior与某个ActiveRecord绑定，就会调用`yii\behaviors\TimeStampBehavior::attach()`，那么就会有:
```
// 这里 $owner 是某个 ActiveRecord
public function attach($owner)
{
    $this->owner = $owner;

    // 遍历上面提到的 events() 所定义的数组
    foreach ($this->events() as $event => $handler) {

        // 调用 ActiveRecord::on 来绑定事件
        // 这里 $handler 为字符串 `evaluateAttributes`
        // 因此，相当于调用 on(BaseActiveRecord::EVENT_BEFORE_INSERT,
        // [$this, 'evaluateAttributes'])
        $owner->on($event, is_string($handler) ? [$this, $handler] :
            $handler);
    }
}
```
因此，事件`BaseActiveRecord::EVENT_BEFORE_INSERT`和`BaseActiveRecord::EVENT_BEFORE_UPDATE`就绑定到了ActiveRecord上了。当新建记录或更新记录时，`TimeStampBehavior::evaluateAttributes`就会被触发。从而实现时间戳的功能。具体可以看看`yii\behaviors\AttributeBehavior::evaluateAttributes()`和`yii\behaviors\TimeStampBehavior::getValues()`的代码。这里因为只是具体功能实现。
#### 行为的属性和方法注入原理
上面我们了解到了行为的用意在于将自身的属性和方法注入给所依附的类。那么Yii中是如何将一个行为`yii\base\Behavior`的属性和方法，注入到一个`yii\base\Component`中的呢？对于属性而言，是通过`__get()`和`__set()`魔术方法来实现的。对于方法，是通过`__call()`方法。
##### 属性的注入
以读取为例，如果访问`$Component->property1`，Yii在幕后干了些什么呢？这个看看`yii\base\Component::__get()`
```
public function __get($name)
{
    $getter = 'get' . $name;
    if (method_exists($this, $getter)) {
        return $this->$getter();
    } else {
        // 注意这个 else 分支的内容，正是与 yii\base\Object::__get() 的
        // 不同之处
        $this->ensureBehaviors();
        foreach ($this->_behaviors as $behavior) {
            if ($behavior->canGetProperty($name)) {

                // 属性在行为中须为 public。否则不可能通过下面的形式访问呀。
                return $behavior->$name;
            }
        }
    }
    if (method_exists($this, 'set' . $name)) {
        throw new InvalidCallException('Getting write-only property: ' .
            get_class($this) . '::' . $name);
    } else {
        throw new UnknownPropertyException('Getting unknown property: ' .
            get_class($this) . '::' . $name);
    }
}
```
重点来看`yii\base\Compoent::__get()`与`yii\base\Object::__get()`的不同之处。就是在于对于未定义getter函数之后的处理，`yii\base\Object`是直接抛出异常，告诉你想要访问的属性不存在之类。但是`yii\base\Component`则是在不存在getter之后，还要看看是不是注入的行为的属性：  
* 首先，调用了`$this->ensureBehaviors()`。这个方法已经在前面讲过了，主要是确保行为已经绑定。
* 在确保行为已经绑定后，开始遍历`$this->_behaviors`。Yii将类所有绑定的行为都保存在`yii\base\Compoent::$_behaviors[]`数组中。
* 最后，通过行为的`canGetProperty()`判断这个属性，是否是所绑定行为的可读属性，如果是，就返回这个行为的这个属性`$behavior->name`。 完成属性的读取。对于setter，代码类似。

##### 方法的注入
与属性的注入通过`__get() __set()`魔术方法类似， Yii通过`__call()`魔术方法实现对行为中方法的注入:
```
public function __call($name, $params)
{
    $this->ensureBehaviors();
    foreach ($this->_behaviors as $object) {
        if ($object->hasMethod($name)) {
            return call_user_func_array([$object, $name], $params);
        }
    }
    throw new UnknownMethodException('Calling unknown method: ' .
        get_class($this) . "::$name()");
}
```
从上面的代码中可以看出，Yii还是先是调用了`$this->ensureBehaviors()`确保行为已经绑定。  
然后，也是遍历`yii\base\Component::$_behaviros[]`数组。通过`hasMethod()`方法判断方法是否存在。 如果所绑定的行为中要调用的方法存在，则使用PHP的`call_user_func_array()`调用之。至于`hasMethod()`方法看后面再讲。
##### 注入属性与方法的访问控制
在前面我们针对行为中public和private、protected的成员在所绑定的类中是否可访问举出了具体例子。这里我们从代码层面解析原因。  
在上面的内容，我们知道，一个属性可不可访问，主要看行为的`canGetProperty()`和`canSetProperty()`。而一个方法可不可调用，主要看行为的`hasMethod()`。 由于`yii\base\Behavior`继承自我们的老朋友`yii\base\Object`，所以上面提到的三个判断方法，事实上代码都在`Object`中。我们一个一个来看:
```
public function canGetProperty($name, $checkVars = true)
{
    return method_exists($this, 'get' . $name) || $checkVars &&
        property_exists($this, $name);
}

public function canSetProperty($name, $checkVars = true)
{
    return method_exists($this, 'set' . $name) || $checkVars &&
        property_exists($this, $name);
}

public function hasMethod($name)
{
    return method_exists($this, $name);
}
```
这三个方法真的谈不上复杂。对此，我们可以得出以下结论：
* 当向Component绑定的行为读取（写入）一个属性时，如果行为为该属性定义了一个getter (setter)，则可以访问。或者，如果行为确实具有该成员变量即可通过上面的判断，此时，该成员变量可为public,private,protected。但最终只有`public`的成员变量才能正确访问。原因在上面讲注入的原理时已经交待了。
* 当调用Component绑定的行为的一个方法时，如果行为已经定义了该方法，即可通过上面的判断。此时，这个方法可以为public,private,protected。但最终只有`public`的方法才能正确调用。如果你理解了上一款的原因，那么这里也就理解了。

#### 行为与继承和特性（Traits）的区别
从实现的效果看，你是不是会认为Yii真是多此一举？PHP中要达到这样的效果，可以使用继承呀，可以使用PHP新引入的特性（Traits）呀。但是，行为具有继承和特性所没有的优点，从实际使用的角度讲，继承和特性更靠底层点。靠底层，就意味着开发效率低，运行效率高。行为的引入，是以可以接受的运行效率牺牲为成本，谋取开发效率大提升的一笔买卖。
##### 行为与继承
首先来讲，拿行为与继承比较，从逻辑上是不对的，这两者是在完全不同的层面上的事物，是不对等的。之所以进行比较，是因为在实现的效果上，两者有的类似的地方。看起来，行为和继承都可以使一个类具有另一个类的属性和方法，从而达到扩充类的功能的目的。  
相比较于使用继承的方式来扩充类功能，使用行为的方式，一是不必对现有类进行修改，二是PHP不支持多继承，但是Yii可以绑定多个行为，从而达到类似多继承的效果。  
反过来，行为是绝对无法替代继承的。亚洲人，美洲人都是地球人，你可以将亚洲人和美洲人当成地球人来对待。但是，你绝对不能把一只在某些方面表现得像人的猴子，真的当成人来对待。  
##### 行为与特性
特性是PHP5.4之后引入的一个新feature。从实现效果看，行为与特性都达到把自身的public变量、属性、方法注入到当前类中去的目的。在使用上，他们也各有所长，但总的原则可以按下面的提示进行把握。  
倾向于使用行为的情况：  
* 行为从本质上讲，也是PHP的类，因此一个行为可以继承自另一个行为，从而实现代码的复用。而特性只是PHP的一种语法，效果上类似于把特性的代码导入到了类中从而实现代码的注入，特性是不支持继承的。
* 行为可以动态地绑定、解除，而不必要对类进行修改。但是特性必须在类在使用`use`语句，要解除特性时，则要删除这个语句。换句话说，需要对类进行修改。
* 行为还以在在配置阶段进行绑定，特性就不行了。
* 行为可以用于对事件进行反馈，而特性不行。
* 当出现命名冲突时，行为会自行排除冲突，自动使用先绑定的行为。而特性在发生冲突时，需要人为干预，修改发生冲突的变量名、属性名、方法名。

倾向于使用特性的情况：  
* 特性比行为在效率上要高一点，因为行为其实是类的实例，需要时间和空间进行分配。
* 特性是PHP的语法，因此，IDE的支持要好一些。目前还没有IDE能支持行为。


