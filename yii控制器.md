####控制器ID（Yii内部调用控制器时的唯一标识）

通常情况下，控制器用来处理请求有关的资源类型，因此控制器ID通常为和资源有关的名词。 例如使用`article`作为处理文章的控制器ID。  

控制器ID应仅包含英文小写字母、数字、下划线、中横杠和正斜杠， 例如`article`和`post-comment`是真是的控制器ID，`article?`,`PostComment`,`admin\post`不是控制器ID。  

控制器Id可包含子目录前缀，例如`admin/article`代表yii\base\Application::controllerNamespace控制器命名空间下 `admin`子目录中`article`控制器。 子目录前缀可为英文大小写字母、数字、下划线、正斜杠，其中正斜杠用来区分多级子目录(如`panels/admin`)。  

#### 控制器类命名
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

#### 控制器部署

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
#### 默认控制器
每个应用有一个由yii\base\Application::defaultRoute属性指定的默认控制器； 当请求没有指定 路由，该属性值作为路由使用。对于yii\web\Application网页应用，它的值为`site`， 对于 yii\console\Application控制台应用，它的值为`help`， 所以URL为`http://hostname/index.php` 表示由`site` 控制器来处理。  

可以在 应用配置 中修改默认控制器，如下所示：
```
[
    'defaultRoute' => 'main',
]
```



