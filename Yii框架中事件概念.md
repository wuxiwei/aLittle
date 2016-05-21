使用事件，可以在特定的时点，触发执行预先设定的一段代码，事件既是代码解耦的一种方式，也是设计业务流程的一种模式。现代软件中，事件无处不在，比如，你发了个微博，触发了一个事件，导致关注你的人，看到了你新发出来的内容。对于事件而言，有这么几个要素：  
* 这是一个什么事件？一个软件系统里，有诸多事件，发布新微博是事件，删除微博也是一种事件。
* 谁触发了事件？你发的微博，就是你触发的事件。
* 谁负责监听这个事件？或者谁能知道这个事件发生了？服务器上处理用户注册的模块，肯定不会收到你发出新微博的事件。
* 事件怎么处理？对于发布新微博的事件，就是通知关注了你的其他用户。
* 事件相关数据是什么？对于发布新微博事件，包含的数据至少要有新微博的内容，时间等。

#### Yii中与事件相关的类
Yii中，事件是在`yii\base\Component`中引入的，注意，`yii\base\Object`不支持事件。所以，当你需要使用事件时，请从`yii\base\Component`进行继承。同时，Yii中还有一个与事件紧密相关的`yii\base\Event`，他封装了与事件相关的有关数据，并提供一些功能函数作为辅助:
```
ass Event extends Object
{
    public $name;               // 事件名
    public $sender;             // 事件发布者，通常是调用了 trigger() 的对象或类。
    public $handled = false;    // 是否终止事件的后续处理
    public $data;               // 事件相关数据
    private static $_events = [];
    public static function on($class, $name, $handler, $data = null, $append = true)
    {
            // ... ...
            // 用于绑定事件handler
    }

    public static function off($class, $name, $handler = null)
    {
            // ... ...
            // 用于取消事件handler绑定
    }

    public static function hasHandlers($class, $name)
    {
            // ... ...
            // 用于判断是否有相应的handler与事件对应
                        
    }

    public static function trigger($class, $name, $event = null)
    {
            // ... ...
            // 用于触发事件
    }
}
```
#### 事件handler
所谓事件handler就是事件处理程序，负责事件触发后怎么办的问题。从本质上来讲，一个事件handler就是一段PHP代码，即一个PHP函数。对于一个事件handler，可以是以下的形式提供：
* 一个PHP全局函数的函数名，不带参数和括号，光秃秃的就一个函数名。如`trim`，注意，不是`trim($str)`也不是`trim()`。
* 一个对象的方法，或一个类的静态方法。如`$person->sayHello()`可以用为事件handler，但要改写成以数组的形式，`[$person, 'sayHello']`，而如果是类的静态方法，那应该是`['namespace\to\Person', 'sayHello']`。
* 匿名函数。如`function ($event) { ...  }`

但无论是何种方式提供，一个事件handler必须具有以下形式:
```
function ($event) {
    // $event 就是前面提到的 yii\base\Event
}
```
还有一点容易犯错的地方，就是对于类自己的成员函数，尽管在调用`on()`进行绑定时，看着这个handler是有效的，因此，有的小伙伴就写成这样了`$this->on(EVENT_A, 'publicMethod')`，但事实上，这是一个错误的写法。以字符串的形式提供handler，只能是PHP的全局函数。这是由于handler的调用是通过`call_user_func()`来实现的。因此，handler的形式，与`call_user_func()`的要求是一致的。这将在事件的触发中介绍。
#### 事件的绑定与解除
##### 事件的绑定
有了事件handler，还要告诉Yii，这个handler是负责处理哪种事件的。这个过程，就是事件的绑定，把事件和事件handler这两个蚂蚱绑在一根绳上，当事件跳起来的时候，就会扯动事件handler啦。  
`yii\base\Component::on()`就是用来绑定的，很容易就猜到，`yii\base\Component::off()`就是用来解除的。对于绑定，有以下形式:
```
$person = new Person;

// 使用PHP全局函数作为handler来进行绑定
$person->on(Person::EVENT_GREET, 'person_say_hello');

// 使用对象$obj的成员函数say_hello来进行绑定
$person->on(Person::EVENT_GREET, [$obj, 'say_hello']);

// 使用类Greet的静态成员函数say_hello进行绑定
$person->on(Person::EVENT_GREET, ['app\helper\Greet', 'say_hello']);

// 使用匿名函数
$person->on(Person::EVENT_GREET, function ($event) {
    echo 'Hello';
});
```
事件的绑定可以像上面这样在运行时以代码的形式进行绑定，也可以在配置中进行绑定。当然，这个配置生效的过程其实也是在运行时的。  
上面的例子只是简单的绑定了事件与事件handler，如果有额外的数据传递给handler，可以使用`yii\base\Component::on()`的第三个参数。这个参数将会写进`Event`的相关数据字段，即属性`data`。如:
```
$person->on(Person::EVENT_GREET, 'person_say_hello', 'Hello World!');

// 'Hello World!' 可以通过 $event访问。
function person_say_hello($event)
{
    echo $event->data;                // 将显示 Hello World!

}
```
yii\base\Component 维护了一个handler数组，用来保存绑定的handler:
```
// 这个就是handler数组
private _events = [];

// 绑定过程就是将handler写入_event[]
public function on($name, $handler, $data = null, $append = true)
{
    $this->ensureBehaviors();
    if ($append || empty($this->_events[$name])) {
           $this->_events[$name][] = [$handler, $data];
    } else {
            array_unshift($this->_events[$name], [$handler, $data]);
    }
}
```
##### 保存handler的数据结构
从上面代码我们可以了解两个方向的内容，一是`$_event[]`的数据结构，二是绑定handler的逻辑。  
从handler数组`$_evnet[]`的结构看，首先他是一个数组，保存了该Component的所有事件handler。该数组的下标为事件名，数组元素是形为一系列 [$handler, $data] 的数组。  
在事件的绑定逻辑上，按照以下顺序：  
* 参数`$append`是否为`true`。为`true`表示所要绑定的事件handler要放在`$_event[]`数组的最后面。这也是默认的绑定方式。
* 参数`$append`是否为`false`。表示handler要放在数组的最前面。这个时候，要多进行一次判定。
* 如果所有绑定的事件还没有已经绑定好的handler，也就是说，将要绑定的handler是第一个，那么无论`$append`是否是`true`，该handler必然是第一个元素，也是最后一个元素。
* 如果`$append`为`false`，且要绑定的事件已经有了handler，那么，就将新绑定的事件插入到数组的最前面。

handler在`$event[]`数组中的位置很重要，代表的是执行的先后顺序。这个在`多个事件handler的顺序`中会讲到。
##### 事件的解除
在解除时，就是使用`unset()`函数，处理`$_event[]`数组的相应元素。`yii\base\Component::off()`如下所示:
```
public function off($name, $handler = null)
{
    $this->ensureBehaviors();
    if (empty($this->_events[$name])) {
        return false;
    }
    // $handler === null 时解除所有的handler
    if ($handler === null) {
        unset($this->_events[$name]);
        return true;
    } else {
        $removed = false;
    // 遍历所有的 $handler
    foreach ($this->_events[$name] as $i => $event) {
        if ($event[0] === $handler) {
            unset($this->_events[$name][$i]);
            $removed = true;
        }
    }
    if ($removed) {
        $this->_events[$name] = array_values($this->_events[$name]);
    }
        return $removed;
    }
}
```
要留意以下几点：  
* 当`$handler`为`null`时，表示解除`$name`事件的所有handler。
* 在解除`$handler`时，将会解除所有的这个事件下的`$handler`。虽然一个handler多次绑定在同一事件上的情况不多见，但这并不是没有，也不是没有意义的事情。在特定的情况下，确实有一个handler多次绑定在同一事件上。因此在解除时，所有的`$handler`都会被解除。

#### 事件的触发
事件的处理程序handler有了，事件与事件handler关联好了，那么只要事件触发了，handler就会按照设计的路子走。事件的触发，需要调用`yii\base\Component::trigger()`
```
public function trigger($name, Event $event = null)
{
    $this->ensureBehaviors();
    if (!empty($this->_events[$name])) {
        if ($event === null) {
            $event = new Event;
        }
        if ($event->sender === null) {
            $event->sender = $this;
        }
        $event->handled = false;
        $event->name = $name;
        // 遍历handler数组，并依次调用
        foreach ($this->_events[$name] as $handler) {
            $event->data = $handler[1];

            // 使用PHP的call_user_func调用handler
            call_user_func($handler[0], $event);

            // 如果在某一handler中，将$evnet->handled 设为true，
            // 就不再调用后续的handler
            if ($event->handled) {
                    return;
            }
        }
    }
    Event::trigger($this, $name, $event);   // 触发类一级的事件
}
```
以`yii\base\Application`为例，他定义了两个事件，`EVENT_BEFORE_REQUEST``EVENT_AFTER_REQUEST`分别在处理请求的前后触发:
```
abstract class Application extends Module
{
    // 定义了两个事件
    const EVENT_BEFORE_REQUEST = 'beforeRequest';
    const EVENT_AFTER_REQUEST = 'afterRequest';
    public function run()
    {
    try {
       $this->state = self::STATE_BEFORE_REQUEST;
       // 先触发EVENT_BEFORE_REQUEST
       $this->trigger(self::EVENT_BEFORE_REQUEST);
       $this->state = self::STATE_HANDLING_REQUEST;

       // 处理Request
       $response = $this->handleRequest($this->getRequest());

       $this->state = self::STATE_AFTER_REQUEST;

       // 处理完毕后触发EVENT_AFTER_REQUEST
       $this->trigger(self::EVENT_AFTER_REQUEST);

       $this->state = self::STATE_SENDING_RESPONSE;
       $response->send();

       $this->state = self::STATE_END;

       return $response->exitStatus;

               
       }catch (ExitException $e) {

       $this->end($e->statusCode, isset($response) ? $response : null);
       return $e->statusCode;
       }
     }
}
```
对于事件的定义，提倡使用const常量的形式，可以避免写错。`trigger('Hello')`和`trigger('hello')`可是不同的事件哦。原因在于handler数组下标，就是事件名。而PHP里数组下标是区分大小写的。所以，用类常量的方式，可以避免这种头疼的问题。  
在触发事件时，可以把与事件相关的数据传递给所有的handler。比如，发布新微博事件:
```
// 定义事件的关联数据
class MsgEvent extend yii\base\Event
{
    public $dateTime;   // 微博发出的时间
    public $author;     // 微博的作者
    public $content;    // 微博的内容
}

// 在发布新的微博时，准备好要传递给handler的数据
$event = new MsgEvent;
$event->title = $title;
$event->author = $auhtor;

// 触发事件
$msg->trigger(Msg::EVENT_NEW_MESSAGE, $event);
```
注意这里数据的传入，与使用`on()`绑定handler时传入数据方法的不同。在`on()`中，使用一个简单变量，传入，并在handler中通过`$event->data`进行访问。这个是在绑定时确定的数据。而有的数据是没办法在绑定时确定的，如发出微博的时间。这个时候，就需要在触发事件时提供其他的数据了。也就是上面这段代码使用的方法了。这两种方法，一种用于提供绑定时的相关数据，一种用于提供事件触发时的数据，各有所长，互相补充。
#### 多个事件handler的顺序
Yii中是支持这种一对多的绑定的。那么，在一个事件触发时，哪个handler会被先执行呢？各handler之间总有一个先后问题吧。这个可能不同的编程语言、不同的框架有不同的实现方式。有的语言是以堆栈的形式来保存handler，可能会以后绑定上去的事件先执行的方式运作。这种方式的好处是编码的人权限大些，可以对事件进行更改、拦截、中止，移花接木、偷天换日、无中生有，各种欺骗后面的handler。而Yii是使用数组来保存handler的，并按顺序执行这些handler。这意味着一般框架上预设的handler会先执行。但是不要以为Yii的事件handler就没办法偷天换日了，要使后加上的事件handler先运行，只需在调用`yii\base\Component::on()`进行绑定时，将第四个参数设为`$append`设为`false`那么这个handler就会被放在数组的最前面了，它就会被最先执行，它也就有可能欺骗后面的handler了。  
为了加强安全生产，国家安监局对某个煤矿进行监管，一旦发生矿难，他们会收到报警，这就是一个事件和一个handler:
```
$coal->on(Coal::EVENT_DISASTER, [$government, 'onDisaster']);
class Government extend yii\base\Component
{
    ... ...
    public function onDisaster($event)
    {
        echo 'DISASTER! from ' . $event->sender;
    }

}
```
由于煤矿自身也要进行管理，所以，政府允许煤矿可以编写自己的handler对矿难进行处理。但是，这个小煤窑的老板，你有张良计，我有过墙梯，对于发生矿难这一事件编写了一个handler专门用于瞒报:
```
// 第四个参数设为false，使得该handler在整个handler数组中处于第一个
$coal->on(Coal::EVENT_DISASTER, [$baddy, 'onDisaster'], null, false);

calss Baddy extend yii\base\Component
{
    ... ...
    public function onDisaster($event)
    {
        // 将事件标记为已经处理完毕，阻止后续事件handler介入。
        $event->handled = true;
    }
}
```
坏人不可怕，会编程的坏人才可怕。我们要阻止他，所以要把绑定好的handler解除。这个解除是绑定的逆向过程，在实质上，就是把对应的handler从handler数组中删除。使用`yii\base\Component::off()`就能删除:
```
// 删除所有EVENT_DISASTER事件的handler
$coal->off(Coal::EVENT_DISASTER);

// 删除一个PHP全局函数的handler
$coal->off(Coal::EVENT_DISASTER, 'global_onDisaster');

// 删除一个对象的成员函数的handler
$coal->off(Coal::EVENT_DISASTER, [$baddy, 'onDisaster']);

// 删除一个类的静态成员函数的handler
$coal->off(Coal::EVENT_DISASTER, ['path\to\Baddy', 'static_onDisaster']);

// 删除一个匿名函数的handler
$coal->off(Coal::EVENT_DISASTER, $anonymousFunction);
```
其中，第三种方法就可以把小煤窑老板的handler解除下来。  
细心的读者朋友可能留意到，在删除匿名函数handler时，需要使用一个变量。请读者朋友留意，就算你调用`yii\base\Component::on()``yii\base\Component::off()`时，写了两个一模一样的匿名函数，你也没办法把你前面的匿名handler解除。从本质上来讲，两个匿名函数就是两个不同的存在，为了能够正确解除，需要先把匿名handler保存成一个变量，如上面的`$anonymousFunction`，然后再依次绑定、解除。但是，使用了变量后，就失去了匿名函数的一大心理上的优势，你本不用去关心他的，建议是在这种情况下，就不要使用匿名函数了。因此，在作为handler时，要慎重使用匿名函数。只有在确定不需要解除时，才可以使用。
#### 事件的级别
前面的事件，都是针对类的实例而言的，也就是事件的触发、处理全部都在实例范围内。这种级别的事件用情专一，不与类的其他实例发生关系，也不与其他类、其他实例发生关系。除了实例级别的事件外，还有类级别的事件。对于Yii，由于Application是一个单例，所有的代码都可以访问这个单例。因此，有一个特殊级别的事件，全局事件。但是，本质上，他只是一个实例级别的事件。
##### 类级别事件
先讲讲类级别的事件。类级别事件用于响应所有类实例的事件。比如，工头需要了解所有工人的下班时间，那么，对于数百个工人，即数百个Worker实例，工头难道要一个一个去绑定自己的handler么？这也太低级了吧？其实，他只需要绑定一个handler到Worker类，这样每个工人下班时，他都能知道了。与实例级别的事件不同，类级别事件的绑定需要使用`yii\base\Event::on()`
```
Event::on(
    Worker::className(),                     // 第一个参数表示事件发生的类
    Worker::EVENT_OFF_DUTY,                  // 第二个参数表示是什么事件
    function ($event) {                      // 对事件的处理
        echo $event->sender . ' 下班了';
    }
);
```
这样，每个工人下班时，会触发自己的事件处理函数，比如去打卡。之后，会触发类级别事件。类级别事件的触发仍然是在`yii\base\Component::trigger()`中:
```
Event::trigger($this, $name, $event);                // 触发类一级的事件
```
这个语句就触发了类级别的事件。类级别事件，总是在实例事件后触发。既然触发时机靠后，那么如果有一天你要早退又不想老板知道，你就可以向小煤窑老板那样，通过`$event->handled = true`，来终止事件处理。  
从`yii\base\Event::trigger()`的参数列表来看，比`yii\base\Component::trigger()`多了一个参数`$class`表示这是哪个类的事件。因此，在保存`$_event[]`数组上，`yii\base\Event`也比`yii\base\Component`要多一个维度:
```
// Component中的$_event[] 数组
$_event[$eventName][] = [$handler, $data];

// Event中的$_event[] 数组
$_event[$eventName][$calssName][] = [$handler, $data];
```
那么，反过来的话，低级别的handler可以在高级别事件发生时发生作用么？这当然也是不行的。由于类级别事件不与任意的实例相关联，所以，类级别事件触发时，类的实例可能都还没有呢，怎么可能进行处理呢？  
类级别事件的触发，应使用`yii\base\Event::trigger()`。这个函数不会触发实例级别的事件。值得注意的是，`$event->sender`在实例级别事件中，`$event->sender`指向触发事件的实例，而在类级别事件中，指向的是类名。在`yii\base\Event::trigger()`代码中，有:
```
if (is_object($class)) {        // $class 是trigger()的第一个参数，表示类名
   if ($event->sender === null) {
       $event->sender = $class;
   }
   $class = get_class($class); // 传入的是一个实例，则以类名替换之
   } else {
       $class = ltrim($class, '\\');
   }')
}
```
这段代码会对`$evnet->sender`进行设置，如果传入的时候，已经指定了他的值，那么这个值会保留，否则，就会替换成类名。  
对于类级别事件，有一个要格外注意的地方，就是他不光会触发自身这个类的事件，这个类的所有祖先类的同一事件也会被触发。但是，自身类事件与所有祖先类的事件，视为同一级别:
```
// 最外面的循环遍历所有祖先类
do {
if (!empty(self::$_events[$name][$class])) {
        foreach (self::$_events[$name][$class] as $handler) {
            $event->data = $handler[1];
            call_user_func($handler[0], $event);

            // 所有的事件都是同一级别，可以随时终止
            if ($event->handled) {
               return;
            }
        }
}
} while (($class = get_parent_class($class)) !== false);
```
上面的嵌套循环的深度，或者叫时间复杂度，受两个方面影响，一是类继承结构的深度，二是`$_event[$name][$class][]`数组的元素个数，即已经绑定的handler的数量。从实践经验看，一般软件工程继承深度超过十层的就很少见，而事件绑定上，同一事件的绑定handler超过十几个也比较少见。因此，上面的嵌套循环运算数量级大约在100～1000之间，这是可以接受的。  
但是，从机制上来讲，由于类级别事件会被类自身、类的实例、后代类、后代类实例所触发，所以，对于越底层的类而言，其类事件的影响范围就越大。因此，在使用类事件上要注意，尽可能往后代类安排，以控制好影响范围，尽可能不在基础类上安排类事件。
#### 全局事件
接下来再讲讲全局级别事件。上面提到过，所谓的全局事件，本质上只是一个实例事件罢了。他只是利用了Application实例在整个应用的生命周期中全局可访问的特性，来实现这个全局事件的。当然，你也可以将他绑定在任意全局可访问的的Component上。  
全局事件一个最大优势在于：在任意需要的时候，都可以触发全局事件，也可以在任意必要的时候绑定，或解除一个事件:
```
Yii::$app->on('bar', function ($event) {
    echo get_class($event->sender);        // 显示当前触发事件的对象的类名称
});

Yii::$app->trigger('bar', new Event(['sender' => $this]));
```
上面的`Yii::$app->on()`可以在任何地方调用，就可以完成事件的绑定。而`Yii::$app->trigger()`只要在绑定之后的任何时候调用就OK了。

