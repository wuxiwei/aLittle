为什么要引入环境的概念呢？  
让我们假设这种非常典型的情形：你们是一个开发团队，成员有Alice, Bob, Charlie等很多人，你们在自己的计算机上进行开发。其中，Alice使用Linux操作系统，Bob使用Mac OS，而Charlie使用Windows，每个人使用的IDE也不尽相同。这都没问题，PHP可以在不同平台上跑得很好。  
问题其实不在PHP，不在Yii。问题在于每个成员在开发的同时，都在自己本地进行编程，都在本地搭建了自己的开发和测试环境。因此，每个人用于所用的数据库名称、用户名、密码都是根据自己的喜好命名的。这此信息是作为配置项保存在Yii应用的配置文件中的。  
这在代码的版本控制上，出现了麻烦。每次各成员向代码库提交代码时，由于他们都修改了配置文件中的用户名、密码等信息为本地信息。因此，在提交代码时，产生了冲突。虽然这个冲突不难解决，但是负责维护代码库的Alice觉得这很无聊，也很苦恼。同时，Bob是个敏感的人，他认为自己本地的数据库连接信息包含了用户名、密码等信息，关系到自己的安全，不应该提交到团队的代码仓库中去。于是，每次提交前，他都要把这些信息从配置文件中删除，然后再提交。提交完了再改回来。不然代码连不上数据库呀。因此，Bob也很苦恼。而Charlie是负责把经过测试的代码部署到产品服务器上。由于开发端的环境和产品端的完全不同，于是每次部署前，他不得不小心地对照原来产品端的配置文件，将开发端的配置，改成产品端的。  
于是Bob提议，将配置文件排除在代码库之外，不对配置文件进行版本控制。这样Alice不用去解决无聊的代码冲突了，Bob也不用每次都删除配置再提交了，Charlie也不用每次拉取代码后第一件事就是把数据库连接信息改成服务端的了。  
这确实解决了一些问题。但不得不说，你们的代码库是不完整的，里面缺少一个环境配置文件。更为要命的事，某天你们的团队经过讨论，认为需要在配置文件中加入新的配置项，或者修改一个配置项，那么你们将不得不通知所有成员，自己手动更新。因为你们未能将该配置文件纳入版本管理，不能实现自动分发。  
毫无疑问，这些来回来去运行环境的切换，一定烦透你了。于是贴心的Yii引入了环境的概念来解决这个问题。其实，Yii1.1中还没有这个特性，Yii2.0的基础模板应用也没这个功能。这是Yii2.0高级模板引入的技术。在实际运用中，各开发团队，特别是大型团队，已经摸索出自己的一套环境配置和部署的操作办法了，Yii2.0环境功能，也是需要与团队现有的环境部署规则相契合的，并非另起一炉。
#### 环境的目录结构
首先了解Yii各环境文件。前面我们讲到，每个Yii环境就是一组配置文件，包含了入口脚本`index.php`和各类配置文件。其实他们都放在`/path/to/digpage.com/environments`目录下面，我们看看这个目录都有哪些东西
```
.
├── dev
│   ├── backend
│   │   ├── config
│   │   │   ├── main-local.php
│   │   │   └── params-local.php
│   │   └── web
│   │       ├── index-test.php
│   │       └── index.php
│   ├── common
│   │   └── config
│   │       ├── main-local.php
│   │       └── params-local.php
│   ├── console
│   │   └── config
│   │       ├── main-local.php
│   │       └── params-local.php
│   ├── frontend
│   │   ├── config
│   │   │   ├── main-local.php
│   │   │   └── params-local.php
│   │   └── web
│   │       ├── index-test.php
│   │       └── index.php
│   └── yii
├── prod
│   ├── backend
│   │   ├── config
│   │   │   ├── main-local.php
│   │   │   └── params-local.php
│   │   └── web
│   │       └── index.php
│   ├── common
│   │   └── config
│   │       ├── main-local.php
│   │       └── params-local.php
│   ├── console
│   │   └── config
│   │       ├── main-local.php
│   │       └── params-local.php
│   ├── frontend
│   │   ├── config
│   │   │   ├── main-local.php
│   │   │   └── params-local.php
│   │   └── web
│   │       └── index.php
│   └── yii
└── index.php
```
从上面的目录结构图中，可以看到，环境目录下有3个东东：  
* 目录 `dev`
* 目录 `prod`
* 文件 `index.php`

其中，`dev`和`prod`结构相同，分别又包含了4个目录和1个文件：  
* `frontend` 目录，用于前台的应用，包含了存放配置文件的`config`目录和存放web入口脚本的`web`目录
* `backend` 目录，用于后台应用，内容与`frontend`相同
* `console` 目录，用于命令行应用，仅包含了`config`目录，因为命令行应用不需要web入口脚本，因此没有`web`目录。
* `common` 目录，用于各web应用和命令行应用通用的环境配置，仅包含了`config`目录，因为不同应用不可能共用相同的入口脚本。 注意这个`common`的层级低于环境的层级，也就是说，他的通用，仅是某一环境下通用，并非所有环境下通用。
* `yii` 文件，是命令行应用的入口脚本文件。

环境目录下的`index.php`并不是通常所说的web入口脚本，它其实是个定义文件。定义了可以使用的环境，打开这个文件看一眼:
```
return [
    'Development' => [
        'path' => 'dev',
        'setWritable' => [
            'backend/runtime',
            'backend/web/assets',
            'frontend/runtime',
            'frontend/web/assets',
        ],
        'setExecutable' => [
            'yii',
        ],
        'setCookieValidationKey' => [
            'backend/config/main-local.php',
            'frontend/config/main-local.php',
        ],
    ],
    'Production' => [
        'path' => 'prod',
        'setWritable' => [
            'backend/runtime',
            'backend/web/assets',
            'frontend/runtime',
            'frontend/web/assets',
        ],
        'setExecutable' => [
            'yii',
        ],
        'setCookieValidationKey' => [
            'backend/config/main-local.php',
            'frontend/config/main-local.php',
        ],
    ],
];
```
不用深入去研究，也可以大致猜到它定义了`Development Production`两个环境，聪明如你，肯定用脚都能猜得出来。其中：  
* `path`指明了当前环境所对应的配置文件存放目录。如`Productions`环境对应的目录为`prod`。
* `setWritable`指明了需要 init 脚本设定为可写模式的目录，这些目录Yii应用在运行时会写入。
* `setExecutable`指明了要将哪个文件设为可执行。这个文件就是命令行应用的入口文件了。
* `setCookieValidationsKey`指明了向哪个文件写入`cookieValidationKey`配置项。

对于分散于各处的`web`和`config`目录而言，它们也是有共性的。
* 凡是`web`目录，存放的都是web应用的入口脚本，一个`index.php`和一个测试版本的`index-test.php`
* 凡是`config`目录，存放的，都是本地配置信息`main-local.php`和`params-local.php`

#### 环境配置的生效规则
说了这么多，现在串起来看。运行`init`脚本就会将某一环境的系列文件复制到当前的文件中，这些文件就是`index.php`,`yii`入口文件和`*-local.php`配置文件。复制到哪呢？复制到了`/path/to/digpage.com/`目录下面，并覆盖`frontend backend console common`中对应的`config`目录和入口脚本（`index.php`或`yii`，`common`中没有入口脚本） 。  
在初始情况下，即未运行过`init`脚本之前，各应用目录（frontend, beckend, console）和通用目录（common），都是已经有一些配置文件了。就是config目录下的`bootstrap.php main.php params.php`。  
总的讲，`*.php`与环境、配置相关的文件都有哪些呢？有表示主配置的`main.php main-local.php`，有表示全局参数的`params.php params-local.php`，表示引导阶段的`bootstrap.php`，有表示入口脚本的`yii`和`index.php`。  
运行了`init`脚本后，环境中的文件也被复制出来。这些配置文件成套地分布在各应用目录和通用目录的`config`目录下。而`index.php`入口脚本则分布在各应用目录的`web`目录下，`yii`入口脚本则只放在应用根目录下。  
其中，所有的`*-local.php`都来自于你选用的环境，表示本地配置的意思。他们不会被写入到代码仓库中。当然，这些环境，也就是整个`/path/to/digpage.com/environments`目录都会被写入代码仓库。  
而所有不带`*-local.php`的main和params配置文件，都不是环境的内容。但在最终的运行环境中，他们是起作用的。  
上面讲到的配置文件有很多，有前台、后台、命令行和common的，有带local的、不带local的，有params、main等，看起来好复杂的样子。那么一个环境发生作用时，这些文件是怎么个顺序呢？ 这要看看[入口文件index.php](#)部分的内容，但总的原则是：  
* 前台、后台和命令行的配置文件间，互不干扰，各管各的。没有先后顺序一说。因为Yii在任意时间，要么是在跑前台，要么是在跑后台。还记得么？他们是不同的应用，他们是独立的。但是，这里有个common，通用于前台、后台等。common的内容被前台或后台的覆盖。
* local和不带local的。明显的，local的是本地配置文件，不带local的是团队间通用的配置。因此，local的覆盖不带local的。
* params, main。这2类文件表示的配置内容并不重叠，他们逻辑上不存在谁覆盖谁的问题。如果看看源代码，可以发现，params只是main配置的一部分。而main的内容，是作为参数传递给应用的构造函数。因此，这两者不存在谁覆盖谁的问题。

#### 环境的使用
环境在具体使用上，把握这么几个原则：  
* 与前后台无关，且与环境无关的配置项，写到`digpage.com\common\config\main.php`中去。不要写到环境中去，也不要写到前台或后台的配置文件中去。比如，当使用`FileCache`作为缓存时，这是与环境无关、与前后台无关的，或者说所有环境下，前后台的配置都相同。有关配置项要写到`digpage.com\common\config\main.php`中去。
* 无关环境的配置，不要写到环境中去，写到应用的配置中去。如，应用的ID，无论是开发环境还是产品环境，ID是不会变的。但前后台ID是不同的。 因此，ID的配置项写到`digpage.com\frontend\config\main.php`中去。而不要写到`digpage.com\environment\frontend\config\main-local.php`中去。
* 与环境有关，但与前后台无关的配置项，要写到环境的`common`配置中去。比如，有的应用前后台使用的数据库是一致的，因此，其`db`配置项应该是一样的。但在开发时，所使用的数据库服务与产品时的数据库服务肯定是不一样的。这种情况下，要所配置项写到`digpage.com\environments\dev\common\main-local.php`
* 环境配置文件只提供框架。凡是环境配置文件，对于敏感信息，如，数据库地址、用户名、密码、API Key等信息， 一律留空，供团队成员在调用`init`后自行填写。
* 本地配置绝不提交代码库。因此，所有的应用目录（frontend, backend等，并非environment目录）下的，所有`*-local.php`都不提交代码库。这点Yii已经通过`.gitignore`为我们做好了。关于`.gitignore`的有关信息，可以看看Git文档的有关内容。

这样做能达到什么效果呢？
* 整个代码仓库中，不会有任何的敏感信息。这个代码仓库即使被外部获取，其危害程度也仅限于代码。数据库、Web Service等密码还不至于泄露。
* 增加、删除、更新配置项的所有修改，都可以通过版本控制系统向整个团队分发。
* 所有团队成员只需要记住权限范围内的敏感信息，就可以完成工作。本地开发的队友，只要有本地的数据库连接信息就可以开发。负责测试的队友，只需要知道测试数据库服务器连接信息就可进行测试。负责部署的队友，只需要知道产品数据库的连接信息就可以完成部署。所有团队成员只需要记住各自权限范围内有限的几个敏感信息就可以了。

不足之处就是每次`pull`代码时，如果配置文件有更新，需要团队成员调用`init`将新的配置文件覆盖本地配置。然后需要手动填入敏感信息。但这种情况在初期配置不太稳定的情况下，根据团队迭代频率，一般一天一次。而等开发进入正常阶段后，配置文件相对稳定，极少有需要修改的。
#### 注意 cookieValidationKey
另外还有一个需要读者朋友们留意的地方，就是每次调用`init`脚本切换、更新环境配置时，`cookieValidationKey`都会被重新生成的随机串所覆盖。这往往会导致更新前后`cookieValidationKey`不一致。从安全角度来讲，定时不定时地更新`cookieValidationKey`也是无可厚非的。但同时我们也要看到由此产生的一个副作用，那就是原先保存在用户机器上的cookie在下次访问时，会全部失效。 一个简单表现就是已经设置了自动登录的用户，在更新`cookieValidationKey`后，全部需要重新登录。  
这在大部分情况下，我们认为这是可以接受的。相信大家在使用互联网的过程中，也有遇到过重新登录的情况。因此，通常我们也可以很放心的使用`init`脚本，而不必特别地去关注自动生成的`cookieValidationKey`。  
这种情况下，`cookieValidationKey`是一个纯粹的、与本地环境密切相关的配置项，我们不用太操心它。  
但是，有的情况下，`cookieValidationKey`则不宜由`init`脚本来自动生成，而需要运维人员人工进行干预。  
需要人工干预的情况，一般出现在对cookie要求比较严格的场景。如，当你的应用采用分布式架构提供服务，同时运行在多个节点的时候。有的负载均衡策略会将同一用户的先后2次请求随机分配给不同的节点进行处理。而如果这两个节点的`cookieValidationKey`不一致，那么就会出现用户就会收到很奇怪的错误信息。  
因此，在分布式情况下，最简便的处理方式还是通过人工干预，确保各节点的`cookieValidationKey`始终一致。  
这种情况下，`cookieValidationKey`已经不是一个纯粹的本地环境配置项了，最多算是一个环境级别的配置项，而与本地、本机没有多大关系了。这种情况下，应当将`cookieValidationKey`与其他诸如数据库密码等敏感信息等同视之，由具有权限的、负责运维的人员掌握。并在每次调用`init`脚本后，将所掌握的`cookieValidationKey`填写到相应的配置项中去。  
这里舍弃掉了init脚本自动生成的`cookieValidationKey`。更新前后，`cookieValidationKey`未发生改变，因此，对于应用的用户而言没什么影响。  
幸运的是，绝大多数情况下，我们还无需对于`cookieValidationKey`操太多的心，可以完全由init脚本自动生成。  
其实，对于Yii开发团队而言，刚开始的时候，是将`cookieValidationKey`放在非环境配置文件`main.php`中的。后来，觉得这个配置项与虽然不一定是本地相关的，但起码与环境有关，因此后来还是放在环境配置文件`main-local.php`中。  
只是，由于安全上的考虑，对于未设置`cookieValidationKey`的情况会抛出异常。同时，又为了方便开发者，又采用了让`init`脚本生成随机串的方式来自动配置。

引用：[环境和配置文件 — 深入理解Yii2.0](http://www.digpage.com/environment.html)
