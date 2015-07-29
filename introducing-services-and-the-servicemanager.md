# 介绍 Service 和 ServiceManager
再上一个章节中我们已经学习了如何在 Zend Framework 2 中创建一个简单的“Hello World”应用程序。这是一个良好易学的开端但是应用程序本身并没实现任何事情。在这个章节我们会将 Service（服务）的概念介绍给您，通过这篇关于 `Zend\ServiceManager\ServiceManager` 的简介文章。

##什么是 Service
一个 Service 是一个执行复杂应用程序逻辑的对象。它是应用程序的一个组成部分，处理所有复杂的事物并且返回一个简单易懂的结果给您。
为了完成我们想让 `Blog`模块 完成的事情，需要创建一个 Service 来让它返回我们所需的数据。该 Service 会从某些源获得数据，然而当我们编写 Service 时并不会真的在意数据源是什么。Service 会根据一个我们定义的 `Interface` 来编写，而未来的数据提供者也要根据这个接口来实现。

##编写 PostService 
当编写一个 Service 的时候，事先定义 `Interface` 是一项常见的最佳实践。定义 `Interface` 可以很好的确保其他程序员能够方便地为我们的服务编写扩展，通过他们自己的实现方法。换句话说，他们可以编写拥有完全一样的函数名的 Service，内部实现方法完全不同，但是拥有一样的特定的输出。

在我们这个例子中，我们需要创建一个 `PostService`。这意味着首先我们要定义一个 `PostServiceInterface`。我们的Service的任务是将博客帖子的数据提供给我们。目前我们先将焦点放在只读类型的任务。想定义一个函数来让我们获取所有帖子的列表，然后我们再定义一个函数来获取单个帖子的内容。

首先我们先在 `/module/Blog/src/Blog/Service/PostServiceInterface.php` 内定义接口。

	<?php
	 // 文件名： /module/Blog/src/Blog/Service/PostServiceInterface.php
	 namespace Blog\Service;
	
	 use Blog\Model\PostInterface;
	
	 interface PostServiceInterface
	 {
	     /**
	      * 应该会分会所有博客帖子集，以便我们对其遍历。数组中的每个条目应该都是
	      * \Blog\Model\PostInterface 接口的实现
	      *
	      * @return array|PostInterface[]
	      */
	     public function findAllPosts();
	
	     /**
	      * 应该会返回单个博客帖子
	      *
	      * @param  int $id 应该被返回的帖子的标识符
	      * @return PostInterface
	      */
	     public function findPost($id);
	 }

如您所见我们定义了两个函数。第一个是 `findAllPosts()` 用来返回所有的帖子，而第二个函数是 `findPost($id)` 用来返回和`$id`标识符参数匹配的帖子。新奇的是事实上我们定义了一个返回值，然而该值还不存在。我们已经默认返回值基本都是`Blog\Model\PostInterface`类型。我们会在稍后定义这个类，目前先创建 `PostService`。

在 `/module/Blog/src/Blog/Service/PostService.php` 内创建类 `PostService`，请确保实现 `PostServiceInterface` 和它所依赖的函数（我们会稍后补全这些函数）。然后您应该有一个类看上去如同下文：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Service/PostService.php
	 namespace Blog\Service;
	
	 class PostService implements PostServiceInterface
	 {
	     /**
	      * {@inheritDoc}
	      */
	     public function findAllPosts()
	     {
	         // 文件名： TODO: Implement findAllPosts() method.
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findPost($id)
	     {
	         // 文件名： TODO: Implement findPost() method.
	     }
	 }

## 编写所需的 Model 文件
因为我们的 `PostService` 会返回 Model，所以也要创建它们。请确保先为 Model 编写 `Interface`！我们来创建 `/module/Blog/src/Blog/Model/PostInterface.php` 和 `/module/Blog/src/Blog/Model/Post.php`。首先是 `/module/Blog/src/Blog/Model/Post.php`：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Model/PostInterface.php
	 namespace Blog\Model;
	
	 interface PostInterface
	 {
	     /**
	      * 会返回博客帖子的 ID
	      *
	      * @return int
	      */
	     public function getId();
	
	     /**
	      * 会返回博客帖子的标题
	      *
	      * @return string
	      */
	     public function getTitle();
	
	     /**
	      * 会返回博客帖子的文本
	      *
	      * @return string
	      */
	     public function getText();
	 }

请注意我们在这里只创造了 getter 函数。这是因为我们目前不关心数据时如何进入 `Post` 类，我们只关心我们如何能通过这些 getter 函数访问这些属性。

然后现在我们对照接口创建合适的 Model 文件。请确保设置所需的类属性并且补全我们的 `PostInterface` 接口定义的 getter 函数。尽管我们的接口不关心 setter 函数，我们还是编写相应的函数来让我们的测试数据得以写入。然后您应该有一个类类似下文：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Model/Post.php
	 namespace Blog\Model;
	
	 class Post implements PostInterface
	 {
	     /**
	      * @var int
	      */
	     protected $id;
	
	     /**
	      * @var string
	      */
	     protected $title;
	
	     /**
	      * @var string
	      */
	     protected $text;
	
	     /**
	      * {@inheritDoc}
	      */
	     public function getId()
	     {
	         return $this->id;
	     }
	
	     /**
	      * @param int $id
	      */
	     public function setId($id)
	     {
	         $this->id = $id;
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function getTitle()
	     {
	         return $this->title;
	     }
	
	     /**
	      * @param string $title
	      */
	     public function setTitle($title)
	     {
	         $this->title = $title;
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function getText()
	     {
	         return $this->text;
	     }
	
	     /**
	      * @param string $text
	      */
	     public function setText($text)
	     {
	         $this->text = $text;
	     }
	 }

## 为我们的 PostService 赋予生机
现在我们拥有我们所需的 Model 文件，可以为 `PostService` 类赋予生机了。为了让 Service-层 清晰易懂，我们目前只会让 `PostService` 直接返回一些事先写死的（硬编码的）内容。在 `PostService` 内创建一个名为 `$data` 的属性，并将其定义为我们的 Model 类型数组。如下例编辑 `PostService`：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Service/PostService.php
	 namespace Blog\Service;
	
	 class PostService implements PostServiceInterface
	 {
	     protected $data = array(
	         array(
	             'id'    => 1,
	             'title' => 'Hello World #1',
	             'text'  => 'This is our first blog post!'
	         ),
	         array(
	             'id'     => 2,
	             'title' => 'Hello World #2',
	             'text'  => 'This is our second blog post!'
	         ),
	         array(
	             'id'     => 3,
	             'title' => 'Hello World #3',
	             'text'  => 'This is our third blog post!'
	         ),
	         array(
	             'id'     => 4,
	             'title' => 'Hello World #4',
	             'text'  => 'This is our fourth blog post!'
	         ),
	         array(
	             'id'     => 5,
	             'title' => 'Hello World #5',
	             'text'  => 'This is our fifth blog post!'
	         )
	     );
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findAllPosts()
	     {
	         // 文件名： TODO: Implement findAllPosts() method.
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findPost($id)
	     {
	         // 文件名： TODO: Implement findPost() method.
	     }
	 }

我们现在有一些数据了，来修改 `find*()` 函数来使其返回合适的 Model 文件：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Service/PostService.php
	 namespace Blog\Service;
	
	 use Blog\Model\Post;
	
	 class PostService implements PostServiceInterface
	 {
	     protected $data = array(
	         array(
	             'id'    => 1,
	             'title' => 'Hello World #1',
	             'text'  => 'This is our first blog post!'
	         ),
	         array(
	             'id'     => 2,
	             'title' => 'Hello World #2',
	             'text'  => 'This is our second blog post!'
	         ),
	         array(
	             'id'     => 3,
	             'title' => 'Hello World #3',
	             'text'  => 'This is our third blog post!'
	         ),
	         array(
	             'id'     => 4,
	             'title' => 'Hello World #4',
	             'text'  => 'This is our fourth blog post!'
	         ),
	         array(
	             'id'     => 5,
	             'title' => 'Hello World #5',
	             'text'  => 'This is our fifth blog post!'
	         )
	     );
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findAllPosts()
	     {
	         $allPosts = array();
	
	         foreach ($this->data as $index => $post) {
	             $allPosts[] = $this->findPost($index);
	         }
	
	         return $allPosts;
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findPost($id)
	     {
	         $postData = $this->data[$id];
	
	         $model = new Post();
	         $model->setId($postData['id']);
	         $model->setTitle($postData['title']);
	         $model->setText($postData['text']);
	
	         return $model;
	     }
	 }

如您所见，现在我们两个函数都拥有合适的返回值了。请注意从技术角度而言目前的实现距离完美还有一大段距离，不过我们会在未来大幅度地改进这个 Service。至少我们目前拥有了一个能运行的 Service 来给我们一些数据，而这些数据吻合我们先前定义的 `PostServiceInterface` 接口。

## 将 Service 带进 Controller
现在我们写好了 `PostService`，接下来就想在我们的 Controller(控制器) 内访问这个 Service。为了实现这点我们即将带出一个新主题，称为“Dependency Injection”（依赖对象注入），简称“DI”。

当我们谈论依赖关系注入的时候，其实就是在谈如何为类设置依赖对象的问题。最常见的形式，“Constructor Injection”（构造器注入），就是用于指定该类全时需要的所有依赖对象的一种方法。

在这个例子中，我们想让博客模组 `ListController` 和我们的 `PostService` 互动。这意味着 `PostService` 类是类 `ListController` 的一个依赖对象（`ListController` 依赖 `PostService`），没有了 `PostService`，`ListController`也无法正常运作。为了确保 `ListController` 总能得到相应的依赖对象，我们会在 `ListController` 的构造器 `__construct()` 中事先定义好依赖对象。请将 `ListController` 修改成下例所示：

	<?php
	 // 文件名： /module/Blog/src/Blog/Controller/ListController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	
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
	 }

如您所见，我们的 `__construct()` 函数现在有了一个必要的参数。现在再也不能没有将符合`PostserviceInterface`接口的类实例作为参数传递的情况下调用这个类了。如果你现在返回浏览器并通过 url `localhost:8080/blog` 重新载入您的工程，下面的错误信息将映入您的眼帘：

	 ( ! ) Catchable fatal error: Argument 1 passed to Blog\Controller\ListController::__construct()
	       must be an instance of Blog\Service\PostServiceInterface, none given,
	       called in {libraryPath}\Zend\ServiceManager\AbstractPluginManager.php on line {lineNumber}
	       and defined in \module\Blog\src\Blog\Controller\ListController.php on line 15
	
这个错误是预料之中的，它告诉你我们的 `ListController` 期望接收 `PostServiceInterface` 接口的一个实现作为参数。
那么我们如何确保 `ListController` 会接收到这样的一个实现？解决问题之道在于告诉应用程序如何创建 `Blog\Controller\ListController` 的实例。如果你还记得我们如何创造控制器的话，类似的，在模组配置文件中的 `invokable` 数组中添加一个条目：

	<?php
	 // 文件名： /module/Blog/config/module.config.php
	 return array(
	     'view_manager' => array( /** ViewManager Config */ ),
	     'controllers'  => array(
	         'invokables' => array(
	             'Blog\Controller\List' => 'Blog\Controller\ListController'
	         )
	     ),
	     'router' => array( /** Router Config */ )
	 );

一个 `invokable` 类是一个可以在没有任何参数下构造的类。由于我们的 `Blog\Controller\ListController` 现在需要有构造参数了，所以需要对此进行一点连带修改。`ControllerManager` 负责实例化控制器，也支持使用 `factories`。`factory` 类用于创建其他类的实例。现在我们为我们的 `ListController` 创建一个 `factory` 类，先将我们的配置文件按照下例修改：

	 <?php
	 // 文件名： /module/Blog/config/module.config.php
	 return array(
	     'view_manager' => array( /** ViewManager Config */ ),
	     'controllers'  => array(
	         'factories' => array(
	             'Blog\Controller\List' => 'Blog\Factory\ListControllerFactory'
	         )
	     ),
	     'router' => array( /** Router Config */ )
	 );

如您所见，再也没有 `invokable` 键了，取而代之的是 `factories` 键。并且，我们的控制器名称 `Blog\Controller\List` 已经被改成不直接匹配 `Blog\Controller\ListController` 类，而是调用一个叫做 `Blog\Factory\ListControllerFactory` 的类。此时如果你刷新浏览器，你会看见另外一个错误信息：

	 An error occurred
	 An error occurred during execution; please try again later.
	
	 Additional information:
	 Zend\ServiceManager\Exception\ServiceNotCreatedException
	
	 File:
	 {libraryPath}\Zend\ServiceManager\AbstractPluginManager.php:{lineNumber}
	
	 Message:
	 While attempting to create blogcontrollerlist(alias: Blog\Controller\List) an invalid factory was registered for this instance type.

这个信息应该相对容易理解。`Zend\Mvc\Controller\ControllerManager` 正在访问 `Blog\Controller\List`，其内部表示是 `blogcontrollerlist`。当它试图执行这个操作的时候发现要调用这个控制器，需要先调用一个 factory 类，然而它并不能找到这个 factory 类，所以产生了我们刚才看见的错误。这也是理所当然的，毕竟我们还没有写 factory 类，所以接下来我们来干这件事。

## 编写一个 Factory 类
在 Zend Framework 2 中的 Factory 类总是需要实现 `Zend\ServiceManager\FactoryInterface` 接口。实现这个类可以让 ServiceManager 知道函数 `createService()` 应该被调用。并且 `createService()` 其实期望接收一个 *ServiceLocatorInterface* 的实现作为参数，这样 *ServiceManager* 就可以通过先前提到的依赖对象注入来对该参数进行注入了。首先我们来实现 factory 类：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Factory/ListControllerFactory.php
	 namespace Blog\Factory;
	
	 use Blog\Controller\ListController;
	 use Zend\ServiceManager\FactoryInterface;
	 use Zend\ServiceManager\ServiceLocatorInterface;
	
	 class ListControllerFactory implements FactoryInterface
	 {
	     /**
	      * Create service
	      *
	      * @param ServiceLocatorInterface $serviceLocator
	      *
	      * @return mixed
	      */
	     public function createService(ServiceLocatorInterface $serviceLocator)
	     {
	         $realServiceLocator = $serviceLocator->getServiceLocator();
	         $postService        = $realServiceLocator->get('Blog\Service\PostServiceInterface');
	
	         return new ListController($postService);
	     }
	 }

现在东西看起来好复杂！让我们先来看看 `$realServiceLocator`。当使用一个 Factory-类 时，其本身会被 `ControllerManager` 调用，事实上它会将**自己**以 `$serviceLocator` 名义进行注入。然而面我们需要真正的 `ServiceManager` 来访问 Service-类，这就是为什么我们去调用 `getServiceLocator()` 函数来获取真正的 `ServiceManager`。

在我们设定好 `$realServiceLocator`(真正的ServiceLocator) 之后，去尝试获得一个叫做 `Blog\Service\PostServiceInterface` 的 Service。该操作应该会得到一个匹配 `PostServiceInterface` 的 Service，然后再将获得的 Service 传给 `ListController`，然后 `ListController` 再被返回。 
 
请注意尽管我们还没有注册一个叫做 `Blog\Service\PostServiceInterface` 的 Service，也请不要期待有天上掉馅饼那样自动生成的好事发生，毕竟我们只是给了 Service 一个接口的名称。刷新您的浏览器，然后就能看见下述错误信息：

	 An error occurred
	 An error occurred during execution; please try again later.
	
	 Additional information:
	 Zend\ServiceManager\Exception\ServiceNotFoundException
	
	 File:
	 {libraryPath}\Zend\ServiceManager\ServiceManager.php:{lineNumber}
	
	 Message:
	 Zend\ServiceManager\ServiceManager::get was unable to fetch or create an instance for Blog\Service\PostServiceInterface
	
和我们预想的完全一致。应用程序中的某处，目前是在我们的 factory 类，要求一个叫做 `Blog\Service\PostServiceInterface` 的 Service，但是却还不认识这个 Service，所以没有办法创建被请求的实例。

## 注册 Service
注册一个 Service 和注册 Controller 一样简单。我们只需要修改 `module.config.php` 文件，添加一个叫做 `service_manager` 新键，其也拥有 `invokable` 和 `factories`。和我们将两者包含在 `controllers` 数组内一样，具体请参照下例配置文件：

	 <?php
	 // 文件名： /module/Blog/config/module.config.php
	 return array(
	     'service_manager' => array(
	         'invokables' => array(
	             'Blog\Service\PostServiceInterface' => 'Blog\Service\PostService'
	         )
	     ),
	     'view_manager' => array( /** View Manager Config */ ),
	     'controllers'  => array( /** Controller Config */ ),
	     'router'       => array( /** Router Config */ )
	 );

如您所见，现在我们添加了一个新的 Service 来监听名称 `Blog\Service\PostServiceInterface` 并且指向我们自己的实现（`Blog\Service\PostService`）。由于我们的 Service 没有任何依赖对象，所以我们能够将这个 Service 添加到 `invokable` 数组中。现在去试试打开浏览器并刷新页面，你应该看不到任何错误信息了，而是和上一个教程一样的页面内容。

## 在我们的 Controller 中使用 Service
现在让我们在 `ListController` 中使用 `PostService`。要实现这点我们需要复写默认 `indexAction()` 函数并且将 `PostService` 的值返回到视图。参照下例修改 `ListController`：

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
	 }

请注意我们的控制器 import 了另外的类。我们需要 import `Zend\View\Model\ViewModel`，因为这基本是您的 Controller 会返回的数据类型。当返回 `ViewModel` 的一个实例时，你总能对所谓的“视图变量”进行赋值。在本示例中我们对一个叫做 `$posts` 的变量进行了赋值，将其设为 `PostService` 中的 `findAllPosts()` 函数的返回值。在这个特例中返回值是一个 `Blog\Model\Post` 类的数组。此时刷新浏览器会发现尚未有任何东西有不同，因为我们很明显需要先修改我们的视图文件来显示我们所需要的数据。

>**注意**：事实上你并不一定需要返回 `ViewModel` 的实例。当您返回一个普通的 php `array`，它也会隐式地被转换成 `ViewModel`。所以简而言之： `return new ViewModel(array('foo' => 'bar'));` 和 `return array('foo' => 'bar');` 是完全等价的。


## 访问视图变量
若想将变量推送到视图的时候，有两种方法实现。要不就是直接像这个代码 `$this->posts`、要不就是隐式地像 `$posts`。两种方法都是一样的，然而，隐式调用 `$posts` 时程序流会走多一步 `__call()` 函数。

我们来修改视图文件，让其显示一个所有博客帖子（由 `PostService` 返回）的列表：

	 <!-- 文件名： /module/Blog/view/blog/list/index.phtml -->
	 <h1>Blog</h1>
	
	 <?php foreach ($this->posts as $post): ?>
	 <article>
	     <h1 id="post<?= $post->getId() ?>"><?= $post->getTitle() ?></h1>
	     <p>
	         <?= $post->getText() ?>
	     </p>
	 </article>
	 <?php endforeach ?>

此处我们简单地对 `$this->posts` 使用一个 `foreach` 循环。由于数组中每一个条目都是 `Blog\Model\Post` 类型的，所以可以使用其对应的 getter 函数来获得我们想要的内容。

## 小结
在本章结束之时，我们已经掌握如何和 `ServiceManager` 进行互动，同时也知道了依赖对象注入是个什么概念，也学会了将我们的 service 返回的变量通过 controller 传给 view(视图)，然后在视图脚本中遍历整个数组。

在下一个章节中，我们会简单了解当我们想从数据库中获得数据时所需要做的事情。

