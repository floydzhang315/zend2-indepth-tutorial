# 了解 Router

现在我们的模组有了一个非常坚固的基础。然而，我们并没有做太多的事情，准确来说，我们做的所有事情仅仅是在一个页面上显示所有 `Blog` 条目而已。在这个章节，你将会学习关于 `Router` 所有你所需要知道的事情，来创建其他路径来显示其中一个博客帖子，添加一个新的博客帖子，和编辑或者删除现有的博客帖子。

## 不同的路径类型

在我们考虑应用程序的细节之前，先看看 Zend Framework 提供的最重要的路径类型。

### Zend\Mvc\Router\Http\Literal

第一个常见的路径类型是 `Literal`（文字） 路径。和上一个章节中提到的一样，文字路径时一种匹配某个特定字符串的路径。通常是文字路径的 URL 例子如下：

- http://domain.com/blog
- http://domain.com/blog/add
- http://domain.com/about-me
- http://domain.com/my/very/deep/page
- http://domain.com/my/very/deep/page

为文字路径进行配置需要你设置好需要匹配的路径，并且需要你定义一些要使用的默认值，举例来说哪个 controller 和哪个 action 用以调用。一个文字路径的简单配置如下例所示：

	 'router' => array(
	     'routes' => array(
	         'about' => array(
	             'type' => 'literal',
	             'options' => array(
	                 'route'    => '/about-me',
	                 'defaults' => array(
	                     'controller' => 'AboutMeController',
	                     'action'     => 'aboutme',
	                 ),
	             ),
	         )
	     )
	 )

### Zend\Mvc\Router\Http\Segment

第二常见的路径类型就是 `Segment`（段）路径。当你的 url 包含变量参数时就适用段路径。这些参数经常用来确认某个您的应用程序里的对象。一些包含参数的 URL 通常都是段路径。

配置一个段路径需要花更多的精力，不过其并不难理解。你需要做的工作一开始都十分相似，你需要定义路径类型，为了确认请将其设置为 `Segment`。然后你必须去定义路径并且对其添加参数。然后和往常一样你还要定义要使用的默认值，唯一和先前不同的是你可以定义参数的默认值。新的部分是你需要定义所谓的 `constraints`（约束），它会作用于所有的段路径上，告诉 `Router` 哪些“规则”被分别应用于哪些参数上。举例来说，一个 `id` 参数只允许有属于 `integer` 的变量，并且只能刚刚好`四位数字`。一个示例配置类似下例：

	 'router' => array(
	     'routes' => array(
	         'archives' => array(
	             'type' => 'segment',
	             'options' => array(
	                 'route'    => '/news/archive/:year',
	                 'defaults' => array(
	                     'controller' => 'ArchiveController',
	                     'action'     => 'byYear',
	                 ),
	                 'constraints' => array(
	                     'year' => '\d{4}'
	                 )
	             ),
	         )
	     )
	 )

这个配置文件为 URL 定义了一个路径类似 `domain.com/news/archive/2014`。如您所见，我们的路径现在包含 `:year` 部分了。这叫做路径参数。段路径的路径参数是以冒号("`：`")为头跟着一串字符来定义的；那个字符串就是参数 `name`。

在 `constraints` 你可以看见我们有另外一个数组。这个数组包含了正则表达式规则，分别对应你的路径的每个参数。在我们这个示例中正则表达式由两部分组成，第一个是 `\d` 代表着“一个数字”，所以从零到九的任意数字都符合规则。第二个部分是 `{4}`，代表着前面的定义必须符合 4 个字符长度。所以用简单语言来说就是“四位数字”。

如果你现在调用 URL `domain.com/news/archive/123`，router 就不能完成匹配，因为我们只支持四位数字的年份。

你也许会注意到我们没有为参数 `year` 定义任何 `defaults`（默认值）。这是因为目前设定好的参数是一个 `required`（必要）参数。如果这个参数是 `optional`（可选）的，那么就必须在路径定义中加以定义。这可以通过为参数添加方括号实现。让我们来修改上述示例路径来让参数 `year` 成为可选项，并且将现在年份作为默认值：

	 'router' => array(
	     'routes' => array(
	         'archives' => array(
	             'type' => 'segment',
	             'options' => array(
	                 'route'    => '/news/archive[/:year]',
	                 'defaults' => array(
	                     'controller' => 'ArchiveController',
	                     'action'     => 'byYear',
	                     'year'       => date('Y')
	                 ),
	                 'constraints' => array(
	                     'year' => '\d{4}'
	                 )
	             ),
	         )
	     )
	 )

请注意我们现在的路径的一个部分是可选的了。不单止参数 `year` 是可选的，连分离 `year` 和 URL 串 `archive` 的斜杠也是可选的了，只有在参数 `year` 存在的时候才能存在。

## 不同的路径概念

当想着应用程序的整体的时候，你就会清晰意识到有许多种路径需要被匹配。当编写这些路径的时候你有两种选择：第一种选择是付出少一点时间在编写路径上，但是在匹配的时候会慢一些；第二种选择是编写多一些十分显式地路径，这样匹配会快一些，但是需要多一些工作来对其一一定义。我们来看看两种方案。


### 泛用型路径

泛用型路径是一种路径，能匹配许多 URL。你也许还记得这个概念，来自于 Zend Framework 1，那个时候你甚至几乎不需要考虑路径问题，因为我们有一条“上帝路径”用于所有事情。你只需要定义 controller、action 和所有参数在一个路径上。

这种方法的一大优势是你可以在开发中节省一大堆时间。然而，劣势就是，匹配这种路径需要耗费长一点的时间，因为每次匹配都需要检查很多变量。不过，只要你不要做得太过分，这是一个可行的概念。因为如此， ZendSkeletonApplication（Zend 骨架应用程序）也使用了一个非常泛用的路径。让我们来看看一个泛用型路径：

	 'router' => array(
	     'routes' => array(
	         'default' => array(
	             'type' => 'segment',
	             'options' => array(
	                 'route'    => '/[:controller[/:action]]',
	                 'defaults' => array(
	                     '__NAMESPACE__' => 'Application\Controller',
	                     'controller'    => 'Index',
	                     'action'        => 'index',
	                 ),
	                 'constraints' => [
	                     'controller' => '[a-zA-Z][a-zA-Z0-9_-]*',
	                     'action'     => '[a-zA-Z][a-zA-Z0-9_-]*',
	                 ]
	             ),
	         )
	     )
	 )

让我们仔细看看这个配置中定义了什么：`route` 部分现在包含两个可选参数，`controller` 和 `action`。`action` 参数只有在 `controller` 参数存在的前提下才是可选的。

`defaults` 字段看上去也有一点点不一样。`__NAMESPACE__` 总会被用来和 `controller` 参数连接在一起。所以举个例子，当 `controller` 参数是“news”时，从 `Router` 调用的 `controller` 就会变成 `Application\Controller\news`；如果参数是“archive”，那么 `Router` 会调用控制器 `Application\Controller\archive`。

`defaults` 字段的确是十分直接的。而这两个参数`controller` 和 `action`，则只需要跟随 PHP 标准的传统，必须以 `a-z` 开头，大小写皆可，然后后面可以接上几乎无限长度的字母、数字、下划线或者横杠。

这种方案的**一个巨大的劣势**是，不单是匹配这种路径会稍微慢一点，还有一点是这种方法根本没有任何错误检测机制。举个例子，当你想要调用一个像 `domain.com/weird/doesntExist` 的 URL 时，`controller` 就会变成 “Application\Controller\weird”，`action` 会变成 “doesntExistAction” 。看到名字相信您也猜得出来这些 `controller` 和 `action` 都不存在。这个路径仍然能够匹配成功，但是一个异常会被抛出，因为 `Router` 无法找到所请求的资源，最终我们会收到 404 回应。

### 使用 child_routes 定义的显式路径

显式路径的实现是通过您自行定义所有可能的路径实现的。若要使用这种方案，你也同样有两种选择。

#### 不使用配置结构

也许最容易理解的编写显式路径的方法就是去编写许多顶层路径。所有路径都有一个显式名称，但是有一大堆重复部分。我们不得不每一次都从新定义要使用的默认 `controller`，而且在配置文件内也没有任何结构可言。让我们看看如何能让这类配置文件更有结构性。

#### 使用 child_routes 增强结构性

另一个定义显式路径的选择就是使用 `child_routes`（子路径）。子路径从他们各自的父母中继承所有的 `options`。换句话说就是：当 `controller` 没有任何变化时，你不需要重新对其进行定义。我们来看看这个例子：
	
	 'router' => array(
	     'routes' => array(
	         'news' => array(
	             'type' => 'literal',
	             'options' => array(
	                 'route'    => '/news',
	                 'defaults' => array(
	                     'controller' => 'NewsController',
	                     'action'     => 'showAll',
	                 ),
	             ),
	             // 定义 "/news" 自身就可以被匹配，不一定需要子路径
	             'may_terminate' => true,
	             'child_routes' => array(
	                 'archive' => array(
	                     'type' => 'segment',
	                     'options' => array(
	                         'route'    => '/archive[/:year]',
	                         'defaults' => array(
	                             'action'     => 'archive',
	                         ),
	                         'constraints' => array(
	                             'year' => '\d{4}'
	                         )
	                     ),
	                 ),
	                 'single' => array(
	                     'type' => 'segment',
	                     'options' => array(
	                         'route'    => '/:id',
	                         'defaults' => array(
	                             'action'     => 'detail',
	                         ),
	                         'constraints' => array(
	                             'id' => '\d+'
	                         )
	                     ),
	                 ),
	             )
	         ),
	     )
	 )

这个路径配置可能需要一点详细解释。首先我们有一个新的配置条目，称作 `may_terminate`。这个属性定义了其父路径可以被单独匹配，不再需要任何子路径。换句话说就是所有下述路径都是有效的：

- /news
- /news/archive
- /news/archive/2014
- /news/42

如果，同时，你若设置了 `may_terminate => false`，那么其父路径只能用于所有其 `child_routes` 的全局默认继承路径。换句话说：只有 `child_routes` 可以被匹配，所以有效路径剩下：

- /news/archive
- /news/archive/2014
- /news/42

可见父路径本身不能被匹配。

接下来我们还有一个新条目，叫做 `child_routes`。着这里我们可以定义追加到父路径上的新路径。实际上你自己定义成子路径的路径和你在顶层定义的路径在本质上没有不同。 唯一会产生区别的时候在共享默认值的重定义时。

使用这种形式的配置的一大优点是，你显式定义了所有路径，所以绝对不会遇到和泛用型路径一样的问题，例如试图访问不存在的控制器。第二个优势就是这种路径在匹配的时候会比泛用型路径更快。最后的一个优势就是你可以很轻松的查看所有可能的路径。

虽然最终这些方案很大程度取决于你的个人喜好，不过请记住，针对显式路径的除错比针对泛用性路径的除错会简单很多。

## 针对我们的博客模组的一个实用例子

现在我们知道如何配置新路径了，让我们先创建一个路径用来显示单个数据库里的 `Blog`。我们希望能够通过内部 ID 来识别博客帖子。由于那个 ID 是一个变量参数，所以我们需要 `Segment` 路径类型的路径。进一步的，我们还想将这个路径设置为 `blog` 的子路径：
  
	 <?php
	 // 文件名： /module/Blog/config/module.config.php
	 return array(
	     'db'              => array( /** DB Config */ ),
	     'service_manager' => array( /* ServiceManager Config */ ),
	     'view_manager'    => array( /* ViewManager Config */ ),
	     'controllers'     => array( /* ControllerManager Config */ ),
	     'router' => array(
	         'routes' => array(
	             'blog' => array(
	                 'type' => 'literal',
	                 'options' => array(
	                     'route'    => '/blog',
	                     'defaults' => array(
	                         'controller' => 'Blog\Controller\List',
	                         'action'     => 'index',
	                     ),
	                 ),
	                 'may_terminate' => true,
	                 'child_routes'  => array(
	                     'detail' => array(
	                         'type' => 'segment',
	                         'options' => array(
	                             'route'    => '/:id',
	                             'defaults' => array(
	                                 'action' => 'detail'
	                             ),
	                             'constraints' => array(
	                                 'id' => '[1-9]\d*'
	                             )
	                         )
	                     )
	                 )
	             )
	         )
	     )
	 );

现在我们设置好了一个新路径来显示单个博客帖子。我们已经对参数 `id` 规定了其只能是正整数。数据库条目的主键 ID 通常从 0 开始所以我们的正则表达式 `constraints` 会稍微复杂一点点。基本上我们告诉转发器参数 `id` 字段需要是以一到九的数字作为开头，然后可以接上零位到无限多位的数字。

这个路径会和其父路径调用一样的 `controller`，但取而代之的它会调用 `detailAction()` 。前往你的浏览器并且请求 URL `http://localhost:8080/blog/2`，你将会看到如下错误信息：
 
	 A 404 error occurred
	
	 Page not found.
	 The requested controller was unable to dispatch the request.
	
	 Controller:
	 Blog\Controller\List
	
	 No Exception available

这是因为实际上控制器尝试访问 `detailAction()` 函数，但是这个函数尚未存在。所以我们现在立刻去创建它。前往你的 `ListController` 然后添加 action。返回一个空白的 `ViewModel` 然后刷新页面：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Controller/ListController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class ListController extends AbstractActionController
	 {
	     /**
	      * @var \Blog\Service\PostServiceInterface
	      */
	     protected $postService;
	
	     public function __construct(PostServiceInterface $postService)
	     {
	         $this->postService = $postService;
	     }
	
	     public function indexAction()
	     {
	         return new ViewModel(array(
	             'posts' => $this->postService->findAllPosts()
	         ));
	     }
	
	     public function detailAction()
	     {
	         return new ViewModel();
	     }
	 }

现在你可以看见那些熟悉的错误信息了，提示模板无法被渲染。让我们立刻创建这个模板，并且假设我们会得到一个 `Post` 对象来查看我们博客的详细信息。在 `/view/blog/list/detail.phtml` 中创建一个新的视图文件：
	
	 <!-- FileName: /module/Blog/view/blog/list/detail.phtml -->
	 <h1>Post Details</h1>
	
	 <dl>
	     <dt>Post Title</dt>
	     <dd><?php echo $this->escapeHtml($this->post->getTitle());?></dd>
	     <dt>Post Text</dt>
	     <dd><?php echo $this->escapeHtml($this->post->getText());?></dd>
	 </dl>

观察这个模板，我们可以期望变量 `$this->post` 是一个 `Post` 模型的实例。现在对 `ListController` 进行修改，好让 `Post` 被传递出去。

	 <?php
	 // 文件名： /module/Blog/src/Blog/Controller/ListController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class ListController extends AbstractActionController
	 {
	     /**
	      * @var \Blog\Service\PostServiceInterface
	      */
	     protected $postService;
	
	     public function __construct(PostServiceInterface $postService)
	     {
	         $this->postService = $postService;
	     }
	
	     public function indexAction()
	     {
	         return new ViewModel(array(
	             'posts' => $this->postService->findAllPosts()
	         ));
	     }
	
	     public function detailAction()
	     {
	         $id = $this->params()->fromRoute('id');
	
	         return new ViewModel(array(
	             'post' => $this->postService->findPost($id)
	         ));
	     }
	 }

如果你刷新你的应用程序，现在你就能看到我们的 `Post` 的详细信息被显示出来了。不过，我们做的事情中还存在一个小问题。虽然我们将自己的 Service 设定成每当没有 `Post` 匹配给出的 `id` 时会抛出一个 `\InvalidArgumentException` 异常，但我们还没能利用这个功能。前往你的浏览器并且打开这个 URL `http://localhost:8080/blog/99`。你会看见如下错误信息：

	 An error occurred
	 An error occurred during execution; please try again later.
	
	 Additional information:
	 InvalidArgumentException
	
	 File:
	 {rootPath}/module/Blog/src/Blog/Service/PostService.php:40
	
	 Message:
	 Could not find row 99

这看上去还是比较丑陋的，所以我们的 `ListController` 应该准备一些手段来应付 `PostService` 抛出的 `InvalidArgumentException` 异常。每当一个无效的 `Post` 被请求时，我们希望用户能被重定向到 Post 总览页面。让我们通过添加 try-catch 语句来对 `PostService` 进行调用：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Controller/ListController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class ListController extends AbstractActionController
	 {
	     /**
	      * @var \Blog\Service\PostServiceInterface
	      */
	     protected $postService;
	
	     public function __construct(PostServiceInterface $postService)
	     {
	         $this->postService = $postService;
	     }
	
	     public function indexAction()
	     {
	         return new ViewModel(array(
	             'posts' => $this->postService->findAllPosts()
	         ));
	     }
	
	     public function detailAction()
	     {
	         $id = $this->params()->fromRoute('id');
	
	         try {
	             $post = $this->postService->findPost($id);
	         } catch (\InvalidArgumentException $ex) {
	             return $this->redirect()->toRoute('blog');
	         }
	
	         return new ViewModel(array(
	             'post' => $post
	         ));
	     }
	 }

现在只要你访问一个无效的 `id`，就会被重定向到 `blog` 路径，也就是博客帖子的列表，完美！