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
* `adminPanels/post-comment` 对应 `app\controllers\adminPanels\PostCommentController.

####yii控制器ID命名规范

