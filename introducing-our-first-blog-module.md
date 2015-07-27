# 介绍我们第一个“博客”模块

现在我们已经对 Zend Framework 2 骨架应用程序有所了解，让我们继续来创建我们自己的模块。我们会创建一个名为“博客”的模块。这个模块会显示一个代表每个博客帖子的数据库条目清单。每个帖子都有三个属性：`id`，`text` 和 `title`。我们会创建用于提交新帖子到数据库中的表单，和修改现有帖子的表单。另外在我们整个快速开始指南中都会使用最佳实践。

## 编写一个新模块

首先我们在 `/module` 目录下创建一个新文件夹名为 `blog`。

为了能让其被 _ModuleManager_ 认作是模块，我们只需要在目标模组的名称空间（`Blog`）中创建一个名为 `Module` 的 PHP 类。
创建文件 `/module/Blog/Module.php`

     <?php
     // Filename: /module/Blog/Module.php
     namespace Blog;
    
     class Module
     {
     }

现在我们拥有一个可以被 ZF2 的 _ModuleManager_ 侦测到的模组了。让我们将这个模组添加到我们应用程序中。虽然这个模组目前还不能干任何事情，但仅仅拥有 `Module.php` 类已经能让其被 ZF2 的 _ModuleManager_ 载入。要实现这点，在主应用程序配置文件 `/config/application.config.php` 中内的模组数组中为 `Blog` 添加一个条目：

	 <?php
	 // Filename: /config/application.config.php
	 return array(
	     'modules' => array(
	         'Application',
	         'Blog'
	     ),
	
	     // ...
	 );

如果你刷新您的应用程序你应该看不见任何改变（也没有任何错误）。

这个时候，我们有必要回头讨论一下模组到底是做什么的。简而言之，一个模组是您的应用程序的一个被封装的功能集合。一个模组可以为您的应用程序添加可见功能，像我们的博客模组；一个模组也可以为应用程序内的其他模组提供背景功能，例如和第三方API进行互动。

将您的代码组织成模组形式有利于让您轻松地重用其他应用程序中的功能，或者使用社区提供的模组。

## 配置模组
我们要做的下一件事情就是为我们的应用程序添加一个路径，这样就能通过 URL `localhost:8080/blog` 来访问我们的模组。要实现这点，需要为我们的模组添加路由配置，但首先我们需要让 `ModuleManager` 知道我们的模组具有这些需要载入的配置表。

添加一个 `getConfig()` 函数到 `Module` 类中，并让其返回配置。（这个函数是在 `ConfigProviderInterface` 中声明的，尽管实际上是否要实现这个接口是可选的） 这个函数应该能返回一个 `array` 或者一个 `Traversable` 对象。继续编辑您的`/module/Blog/Module.php`:

	<?php
	 // Filename: /module/Blog/Module.php
	 namespace Blog;
	
	 use Zend\ModuleManager\Feature\ConfigProviderInterface;
	
	 class Module implements ConfigProviderInterface
	 {
	     public function getConfig()
	     {
	         return array();
	     }
	 }

做到这里我们的 `Module` 就可以被配置了。配置文件可能会变得十分大，此时将所有东西放在 `getConfig()` 中就不是最佳手段了。为了保持我们的工程的组织清晰，我们会将数组配置放在单独的文件中。在此我们创建 `/module/Blog/config/module.config.php`：

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array();

现在我们来重写 `getConfig()` 函数来添加这个刚刚建立的新文件。

	<?php
	 // Filename: /module/Blog/Module.php
	 namespace Blog;
	
	 use Zend\ModuleManager\Feature\ConfigProviderInterface;
	
	 class Module implements ConfigProviderInterface
	 {
	     public function getConfig()
	     {
	         return include __DIR__ . '/config/module.config.php';
	     }
	 }

重新装载您的应用程序，这是你便会见到所有东西仍然和之前一样，接下来我们为配置文件添加一个新路径：

	<?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     // 该行为 RouteManager 打开配置
	     'router' => array(
	         // 打开所有可能路径的配置O
	         'routes' => array(
	             // 定义一个新路径，称为 "post"
	             'post' => array(
	                 // 定义一个路径 "Zend\Mvc\Router\Http\Literal" , 基本上就是一个字符串
	                 'type' => 'literal',
	                 // 配置路径本身
	                 'options' => array(
	                     // 监听 uri "/blog" 
	                     'route'    => '/blog',
	                     // 定义默认控制器和当这个路径匹配时需要执行的动作
	                     'defaults' => array(
	                         'controller' => 'Blog\Controller\List',
	                         'action'     => 'index',
	                     )
	                 )
	             )
	         )
	     )
	 );

现在我们添加了一个叫做 `blog` 的路径，该路径负责监听 URL `localhost:8080/blog`。每当有人访问这个路径时，`Blog\Controller\List` 类中的 `indexAction()` 函数就会被执行。不过，这个控制器暂时还不存在，所以如果你重新装载页面你就会看见这个错误信息：

	 A 404 error occurred
	 Page not found.
	 The requested controller could not be mapped to an existing controller class.
	
	 Controller:
	 Blog\Controller\List(resolves to invalid controller class or alias: Blog\Controller\List)
	 No Exception available

现在我们需要告诉我们的模组要去哪里寻找这个叫做 `Blog\Controller\List` 的控制器。要做到这点我们必须将这个 key 添加到 `controllers` 配置 key，在 `/module/Blog/config/module.config.php` 内。

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     'controllers' => array(
	         'invokables' => array(
	             'Blog\Controller\List' => 'Blog\Controller\ListController'
	         )
	     ),
	     'router' => array( /** Route Configuration */ )
	 );

这个配置文件定义了 `Blog\Controller\List` 是在名称空间 `Blog\Controller` 下的 `ListController` 的别名。重新装载页面应该会看见以下信息：

    ( ! ) Fatal error: Class 'Blog\Controller\ListController' not found in {libPath}/Zend/ServiceManager/AbstractPluginManager.php on line {lineNumber}

这个错误告诉我们应用程序知道要载入哪个类，但是并不知道在哪里能找到它。要修正这个问题，我们需要为我们的 `Module` 配置 [autoloading](http://www.php.net/manual/en/language.oop5.autoload.php)。 Autoloading 是一个过程，让 PHP 能自动根据需求载入类。至于我们的 Module，我们通过添加 `getAutoloaderConfig()` 函数到我们的 Module 类来实现这点。（这个函数是在 [AutoloaderProviderInterface](https://github.com/zendframework/zf2/:current_branch/library/Zend/ModuleManager/Feature/AutoloaderProviderInterface.php) 中定义的，尽管实际上类是否要实现这个接口是可选的）

	 <?php
	 // Filename: /module/Blog/Module.php
	 namespace Blog;
	
	 use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
	 use Zend\ModuleManager\Feature\ConfigProviderInterface;
	
	 class Module implements
	     AutoloaderProviderInterface,
	     ConfigProviderInterface
	 {
	     /**
	      * 返回一个 array 传给 Zend\Loader\AutoloaderFactory.
	      *
	      * @return array
	      */
	     public function getAutoloaderConfig()
	     {
	         return array(
	             'Zend\Loader\StandardAutoloader' => array(
	                 'namespaces' => array(
	                     // Autoload 所有类从名称空间 'Blog' 来自于 '/module/Blog/src/Blog'
	                     __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
	                 )
	             )
	         );
	     }
	
	     /**
	      * 返回配置来和应用程序配置合并
	      *
	      * @return array|\Traversable
	      */
	     public function getConfig()
	     {
	         return include __DIR__ . '/config/module.config.php';
	     }
	 }

这看上去好像很多东西被改变了，不过不用害怕。我们已经添加了一个 `getAutoloaderConfig()函数来提供
Zend\Loader\StandardAutoloader 的配置。这个配置文件告诉应用程序`__名称空间__`(Blog) 中的类都能在
__目录__.'/src/'.__名称空间__ 中找到 (
/module/Blog/src/Blog)。


<tt class="docutils literal"><span class="pre">Zend\Loader\StandardAutoloader</span></tt> 使用了 PHP 社区领导的标准，称为
<cite>PSR-0</cite> （<https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>）。
在其他东西之中，这个标准为 PHP 定义了一个方法来映射类名称到文件系统。所以有了这个配置后，应用程序就知道我们的
<tt class="docutils literal"><span class="pre">Blog\Controller\ListController</span></tt> 类应该在
<tt class="docutils literal"><span class="pre">/module/Blog/src/Blog/Controller/ListController.php</span></tt>中存在。

如果你现在刷新浏览器，你还是会见到同样的错误，即使我们已经配置了 autoloader，因为我们还需要创建控制器类。我们立刻创建这个文件：

	 <?php
	 // Filename: /module/Blog/src/Blog/Controller/ListController.php
	 namespace Blog\Controller;
	
	 class ListController
	 {
	 }

再次重新载入页面，现在我们终于看见不一样的内容了。新的错误信息（类似下文）：

	 A 404 error occurred
	 Page not found.
	 The requested controller was not dispatchable.
	
	 Controller:
	 Blog\Controller\List(resolves to invalid controller class or alias: Blog\Controller\List)
	
	 Additional information:
	 Zend\Mvc\Exception\InvalidControllerException
	
	 File:
	 {libraryPath}/Zend/Mvc/Controller/ControllerManager.php:{lineNumber}
	 Message:
	 Controller of type Blog\Controller\ListController is invalid; must implement Zend\Stdlib\DispatchableInterface

这是因为我们的控制器必须实现 [ZendStdlibDispatchableInterface](https://github.com/zendframework/zf2/:current_branch/library/Zend/Stdlib/DispatchableInterface.php) 接口才能被 ZendFramework 的 MVC 层运行。ZendFramework 提供了一些该接口的基础控制器实现，叫做 [AbstractActionController](https://github.com/zendframework/zf2/:current_branch/library/Zend/Mvc/Controller/AbstractActionController.php)，我们稍后会用。让我们先修改我们的控制器：

	 <?php
	 // Filename: /module/Blog/src/Blog/Controller/ListController.php
	 namespace Blog\Controller;
	
	 use Zend\Mvc\Controller\AbstractActionController;
	
	 class ListController extends AbstractActionController
	 {
	 }

是时候再次刷新站点的页面了，您应该会看见有一个新的错误消息：

	 An error occurred
	 An error occurred during execution; please try again later.
	
	 Additional information:
	 Zend\View\Exception\RuntimeException
	
	 File:
	 {libraryPath}/library/Zend/View/Renderer/PhpRenderer.php:{lineNumber}
	 Message:
	 Zend\View\Renderer\PhpRenderer::render: Unable to render template "blog/list/index"; resolver could not resolve to a file

现在应用程序告诉你一个视图模板文件没法被渲染，这也是可预见的，毕竟我们还没有将其创建。应用程序期望它在 `/module/Blog/view/blog/list/index.phtml`这个位置。创建这个文件并且放置一些假内容在上面：

	 <!-- Filename: /module/Blog/view/blog/list/index.phtml -->
	 <h1>Blog\ListController::indexAction()</h1>

在我们继续之前，先让我们快速的看一下我们将该文件放置在何处。请注意视图文件是放置在 `/view` 子目录下的，并不是在 `/src` 子目录下，因为他们不是 PHP 类文件，而是用于渲染 HTML 的模板文件。下面的路径需要一些解释，不过也非常简单。首先我们有全小写的名称空间，跟着一个小写的控制器名称（没有后缀‘controller’），最后跟上我们正在访问的动作的名称（同样地，没有后缀‘action’）。总的来看像这样：`/view/{namespace}/{controller}/{action}.phtml`。这已经成为了社区标准不过也有潜在的随时被你改变的可能。不过光是创建这个文件是不够的，这就带出了这次快速开始指南的最后主题：我们需要让应用程序知道上哪去寻找视图文件。通过修改我们的模组配置文件 `module.config.php` 来实现：

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     'view_manager' => array(
	         'template_path_stack' => array(
	             __DIR__ . '/../view',
	         ),
	     ),
	     'controllers' => array( /** Controller Configuration */),
	     'router'      => array( /** Route Configuration */ )
	 );

上面的配置告诉应用程序目录 `/module/Blog/view` 中拥有视图文件来匹配上述默认样式。重要的是，你需要记住你不单能将视图文件指定给您的模组，同时也能用于覆盖其他模组的视图文件。

现在刷新您的站点。终于我们可以看见错误消息之外的内容了。

恭喜！你不单止创建了一个简单的“Hello World”风格模组，还学会了许多错误消息和他们的起因。如果到现在你还没被磨死，请继续我们的快速开始指南，一起来创建一个能真正干点事情的模组。