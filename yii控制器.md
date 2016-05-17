#### 控制器
控制器是 MVC 模式中的一部分，是继承yii\base\Controller类的对象，负责处理请求和生成响应。 具体来说，控制器从应用主体接管控制后会分析请求数据并传送到模型，传送模型结果到视图，最后生成输出响应信息。
##### 控制器ID（Yii内部调用控制器时的唯一标识）

通常情况下，控制器用来处理请求有关的资源类型，因此控制器ID通常为和资源有关的名词。 例如使用`article`作为处理文章的控制器ID。  

控制器ID应仅包含英文小写字母、数字、下划线、中横杠和正斜杠， 例如`article`和`post-comment`是真是的控制器ID，`article?`,`PostComment`,`admin\post`不是控制器ID。  

控制器Id可包含子目录前缀，例如`admin/article`代表yii\base\Application::controllerNamespace控制器命名空间下 `admin`子目录中`article`控制器。 子目录前缀可为英文大小写字母、数字、下划线、正斜杠，其中正斜杠用来区分多级子目录(如`panels/admin`)。  

##### 控制器类命名
控制器ID遵循以下规则衍生控制器类名：  
* 将用正斜杠区分的每个单词第一个字母转为大写。注意如果控制器ID包含正斜杠，只将最后的正斜杠后的部分第一个字母转为大写；  
* 去掉中横杠，将正斜杠替换为反斜杠;  
* 增加Controller后缀;  
* 在前面增加yii\base\Application::controllerNamespace控制器命名空间.  

下面为一些示例，假设yii\base\Application::controllerNamespace控制器命名空间为 app\controllers:  
* `article` 对应 `app\controllers\ArticleController`;
* `post-comment` 对应 `app\controllers\PostCommentController`;
* `admin/post-comment` 对应 `app\controllers\admin\PostCommentController`;
* `adminPanels/post-comment` 对应 `app\controllers\adminPanels\PostCommentController`.

控制器类必须能被自动加载，所以在上面的例子中， 控制器`article` 类应在别名为`@app/controllers/ArticleController.php`的文件中定义， 控制器`admin/post3-comment`应在`@app/controllers/admin/Post2CommentController.php`文件中。  

>补充: 最后一个示例`admin/post2-comment`表示你可以将控制器放在yii\base\Application::controllerNamespace控制器命名空间下的子目录中，在你不想用模块的情况下给控制器分类，这种方式很有用。

##### 控制器部署

可通过配置yii\base\Application::controllerMap来强制上述的控制器ID和类名对应， 通常用在使用第三方不能掌控类名的控制器上。  

配置应用配置中的`application configuration`，如下所示：
```
[
        'controllerMap' => [
        // 用类名申明 "account" 控制器
                'account' => 'app\controllers\UserController',
                        // 用配置数组申明"article" 控制器
                        'article' => [
                            'class' => 'app\controllers\PostController',
                                'enableCsrfValidation' => false,
                ],
        ],
]

```
##### 默认控制器
每个应用有一个由yii\base\Application::defaultRoute属性指定的默认控制器； 当请求没有指定 路由，该属性值作为路由使用。对于yii\web\Application网页应用，它的值为`site`， 对于 yii\console\Application控制台应用，它的值为`help`， 所以URL为`http://hostname/index.php` 表示由`site` 控制器来处理。  

可以在 应用配置 中修改默认控制器，如下所示：
```
[
    'defaultRoute' => 'main',
]
```
#### 创建操作

创建操作可简单地在控制器类中定义所谓的操作方法来完成，操作方法必须是以`action`开头的公有方法。 操作方法的返回值会作为响应数据发送给终端用户，如下代码定义了两个操作`index`和`hello-world`:
```
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public function actionIndex()
    {
            return $this->render('index');
    }
        public function actionHelloWorld()
        {
                return 'Hello World';
        }
}
```
##### 操作ID
操作通常是用来执行资源的特定操作，因此，操作ID通常为动词，如`view`,`update`等。  

操作ID应仅包含英文小写字母、数字、下划线和中横杠，操作ID中的中横杠用来分隔单词。例如`view`,`update2`,`comment-post`是真实的操作ID，`view?`, `Update`不是操作ID.  

可通过两种方式创建操作ID，内联操作和独立操作，内联操作在控制器类中定义为方法；独立操作是继承yii\base\Action或它的子类的类。内联操作容易创建，在无需重用的情况下优先使用；独立操作相反，主要用于多个控制器重用，或重构为扩展。
##### 内联操作
内联操作指的是根据我们刚描述的操作方法。  
操作方法的名字是根据操作ID遵循如下规则衍生：  
* 将每个单词的第一个字母转为大写;
* 去掉中横杠;
* 增加action前缀.

例如`index`转成`actionIndex`,`hello-world`转成`actionHelloWorld`。
>注意: 操作方法的名字大小写敏感，如果方法名称为ActionIndex不会认为是操作方法， 所以请求index操作会返回一个异常，也要注意操作方法必须是公有的，私有或者受保护的方法不能定义成内联操作。

因为容易创建，内联操作是最常用的操作，但是如果你计划在不同地方重用相同的操作， 或者你想重新分配一个操作，需要考虑定义它为独立操作。  
##### 独立操作
独立操作通过继承yii\base\Action或它的子类来定义。 例如Yii发布的yii\web\ViewAction和yii\web\ErrorAction都是独立操作。  
要使用独立操作，需要通过控制器中覆盖yii\base\Controller::actions()方法在action map中申明，如下例所示：
```
public function actions()
{
    return [
        // 用类来申明"error" 操作
        'error' => 'yii\web\ErrorAction',
        // 用配置数组申明 "view" 操作
        'view' => [
            'class' => 'yii\web\ViewAction',
                'viewPrefix' => '',
        ],
    ];
}
```
如上所示，`actions()`方法返回键为操作ID、值为对应操作类名或数组configurations的数组。 和内联操作不同，独立操作ID可包含任意字符，只要在`actions()`方法中申明.  

为创建一个独立操作类，需要继承yii\base\Action 或它的子类，并实现公有的名称为`run()`的方法,`run()` 方法的角色和操作方法类似，例如：
```
<?php
namespace app\components;

use yii\base\Action;

class HelloWorldAction extends Action
{
    public function run()
    {
            return "Hello World";
                
    }

}
```
##### 操作结果
操作方法或独立操作的`run()`方法的返回值非常重要，它表示对应操作结果。  

返回值可为 响应 对象，作为响应发送给终端用户。  
* 对于yii\web\Application网页应用，返回值可为任意数据, 它赋值给yii\web\Response::data， 最终转换为字符串来展示响应内容。
* 对于yii\console\Application控制台应用，返回值可为整数， 表示命令行下执行的 yii\console\Response::exitStatus 退出状态。

在上面的例子中，操作结果都为字符串，作为响应数据发送给终端用户，下例显示一个操作通过 返回响应对象（因为yii\web\Controller::redirect()方法返回一个响应对象）可将用户浏览器跳转到新的URL。
```
public function actionForward()
{
    // 用户浏览器跳转到 http://example.com
    return $this->redirect('http://example.com');
}
```
##### 操作参数
内联操作的操作方法和独立操作的`run()`方法可以带参数，称为操作参数。 参数值从请求中获取，对于yii\web\Application网页应用， 每个操作参数的值从`$_GET`中获得，参数名作为键； 对于yii\console\Application控制台应用, 操作参数对应命令行参数。  

如下例，操作`view`(内联操作)申明了两个参数`$id`和`$version`。
```
namespace app\controllers;

use yii\web\Controller;

class PostController extends Controller
{
    public function actionView($id, $version = null)
    {
            // ...
    }
}
```
操作参数会被不同的参数填入，如下所示：  
* `http://hostname/index.php?r=post/view&id=123`： `$id`会填入`123`，`$version`仍为`null`空因为没有`version`请求参数;
* `http://hostname/index.php?r=post/view&id=123&version=2`：`$id`和`$version`分别填入`123`和`2`；
* `http://hostname/index.php?r=post/view`：会抛出yii\web\BadRequestHttpException异常 因为请求没有提供参数给必须赋值参数`$id`；
* `http://hostname/index.php?r=post/view&id[]=123`: 会抛出yii\web\BadRequestHttpException异常因为`$id` 参数收到数字值`['123']`而不是字符串.

如果想让操作参数接收数组值，需要指定`$id`为array，如下所示：
```
public function actionView(array $id, $version = null)
{
    // ...
}
```
现在如果请求为`http://hostname/index.php?r=post/view&id[]=123`，参数`$id`会使用数组值`['123']`，如果请求为`http://hostname/index.php?r=post/view&id=123`，参数`$id`会获取相同数组值，因为无类型的`123`会自动转成数组。  

##### 默认操作
每个控制器都有一个由 yii\base\Controller::defaultAction 属性指定的默认操作， 当路由 只包含控制器ID，会使用所请求的控制器的默认操作。  

默认操作默认为`index`，如果想修改默认操作，只需简单地在控制器类中覆盖这个属性，如下所示：
```
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public $defaultAction = 'home';

        public function actionHome()
        {
                return $this->render('home');
                    
        }
}
```
#### 控制器生命周期
处理一个请求时，应用主体会根据请求路由创建一个控制器，控制器经过以下生命周期来完成请求：  
1. 在控制器创建和配置后，yii\base\Controller::init() 方法会被调用。  
2. 控制器根据请求操作ID创建一个操作对象:  
* 如果操作ID没有指定，会使用yii\base\Controller::defaultAction默认操作ID；
* 如果在yii\base\Controller::actions()找到操作ID，会创建一个独立操作；
* 如果操作ID对应操作方法，会创建一个内联操作；
* 否则会抛出yii\base\InvalidRouteException异常。  
3. 控制器按顺序调用应用主体、模块（如果控制器属于模块）、控制器的`beforeAction()`方法；  
* 如果任意一个调用返回false，后面未调用的`beforeAction()`会跳过并且操作执行会被取消； action execution will be cancelled.
* 默认情况下每个`beforeAction()`方法会触发一个`beforeAction`事件，在事件中你可以追加事件处理操作；  
4. 控制器执行操作:  
* 请求数据解析和填入到操作参数；  
5. 控制器按顺序调用控制器、模块（如果控制器属于模块）、应用主体的`afterAction()`方法；  
* 默认情况下每个`afterAction()`方法会触发一个`afterAction`事件，在事件中你可以追加事件处理操作；  
6. 应用主体获取操作结果并赋值给响应.  

#### 最佳实践
在设计良好的应用中，控制器很精练，包含的操作代码简短； 如果你的控制器很复杂，通常意味着需要重构，转移一些代码到其他类中。  

归纳起来，控制器
* 可访问 请求 数据;
* 可根据请求数据调用 模型 的方法和其他服务组件;
* 可使用 视图 构造响应;
* 不应处理应被模型处理的请求数据;
* 应避免嵌入HTML或其他展示代码，这些代码最好在 视图中处理.
