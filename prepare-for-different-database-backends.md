# 为不同的数据库后台做准备
再上一个章节我们创建了一个 `PostService` 类来返回博客帖子的数据。虽然那个章节作为一个简单易懂的教程十分称职，但是在现实世界应用中却是十分不使用的。没有人会想要每次有一个新帖子产生就去修改一次源代码。幸运的是我们都了解数据库。我们所需要的就是去学习如何通过 ZF2 应用程序和数据库进行互动。

不过这里有一个问题。目前有许多数据库后台系统，例如 SQL 类数据库和非 SQL 类数据库。虽然在现实世界中你会直接使用一个但是你认为最合适的解决方案，但是在实际数据库操作前面创建多一个抽象层来抽象化数据库操作会是更好的实践。我们将这层称之为映射层(Mapper-Layer)。

## 什么是数据库抽象化？
术语“数据库抽象化”听上去好像挺令人困惑的，实际上这是一个非常简单的概念。假设有一个 SQL 数据库和一个非 SQL 数据库。两者都有增删改查操作所对应的函数。例如要在数据库中查询某列数据，在 MySQL 中你会使用这个命令 `mysqli_query('SELECT foo FROM bar')`；但如果你是用的是 ORM for MongoDB，那么你就要使用类似这样的命令 `$mongoODM->getRepository('bar')->find('foo')`。两种数据库引擎都会给你同样的结果但是其执行方法确实完全不同的。

所以如果我们一开始使用一个 SQL 数据库，然后将这些代码直接写进我们的 `PostService` 中。一年之后，却决定迁移到一个非 SQL 数据库上，那么我们将不得不删除所有之前的代码并且重新编写新代码。再过几年，新的状况又出现了，然后我们又要删除所有代码然后重新编写...这显然不是最好的实现方法，这也正说明了为何数据库抽象化/映射层是如此的实用。

我们要做的事情计划本上就是创建一个新的接口。这个接口定义了数据库操作应该**如何**运作，但是实际实现是留空的。我们不要停留在理论上，现在开始进行编码实践吧。

## 创建 PostMapperInterface
首先我们来思考一下有什么可能的数据库操作，我们需要：

寻找一个博客帖子
寻找所有博客帖子
插入新的博客帖子
更新已有的博客帖子
删除已有的博客帖子

上面提到的这些功能都是目前我想到的最重要的功能。考虑到 `insert()` 和 `update` 函数都是对数据库进行写入，所以将两者合并到一个 `save()` 函数是个不错的主意，让 `save()` 函数根据情况调用合适的函数。

首先我们在 `Blog\Mapper` 名称空间下创建一个新文件，叫做 `PostMapperInterface.php`，然后参考下例添加内容：

	<?php
	 // 文件名： /module/Blog/src/Blog/Mapper/PostMapperInterface.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	
	 interface PostMapperInterface
	 {
	     /**
	      * @param int|string $id
	      * @return PostInterface
	      * @throws \InvalidArgumentException
	      */
	     public function find($id);
	
	     /**
	      * @return array|PostInterface[]
	      */
	     public function findAll();
	 }

如您所见我们定义了两个不同的函数。我们觉得一个映射实现（mapper-implementation）应该拥有一个 `find()` 函数来返回一个实现了 `PostInterface` 的对象。然后还要拥有一个叫做 `findAll()` 的函数来返回一个实现了 `PostInterface` 的对象的数组。在这里没有添加 `save()` 和 `delete()` 函数，因为我们目前只考虑只读部分的功能，当然稍后这些功能也会被补全。

## 重构 PostService
现在我们定义了我们的映射层应该如何工作，我们可以在 `PostService` 内对其进行调用。要开始重构，我们要先清空我们的类并且删除所有现有的内容，然后实现 `PostServiceInterface` 接口，现在你的 `PostService` 应该看上去像这样：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Service/PostService.php
	 namespace Blog\Service;
	
	 use Blog\Mapper\PostMapperInterface;
	
	 class PostService implements PostServiceInterface
	 {
	     /**
	      * @var \Blog\Mapper\PostMapperInterface
	      */
	     protected $postMapper;
	
	     /**
	      * @param PostMapperInterface $postMapper
	      */
	     public function __construct(PostMapperInterface $postMapper)
	     {
	         $this->postMapper = $postMapper;
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findAllPosts()
	     {
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findPost($id)
	     {
	     }
	 }

第一件你需要牢记于心的事情是这个接口并不是在 `PostService` 中实现的，而是在这里用作依赖对象，一个被要求的依赖对象，所以我们需要创建 `__construct()` 函数来接收任意这个接口所需的实现作为参数。同时你也要创建一个 protected 变量来存放参数。

当你完成上述内容之后，我们需要一个 `PostMapperInterface` 接口的实现来让我们的 `PostService`得以运作。由于什么都不存在，所以我们现在是没法让我们的应用程序运作的，刷新您的浏览器就能看见如下 PHP 错误：

	 Catchable fatal error: Argument 1 passed to Blog\Service\PostService::__construct()
	 must implement interface Blog\Mapper\PostMapperInterface, none given,
	 called in {path}\module\Blog\src\Blog\Service\PostServiceFactory.php on line 19
	 and defined in {path}\module\Blog\src\Blog\Service\PostService.php on line 17

不过我们的正在做的东西的权力取决于我们**可以**做出的假设。这个 `PostService` 总会接收到一个映射器作为参数。所以在我们的 `find*()` 函数中我们**可以**假设其存在。回想 `PostMapperInterface` 定义了一个 `find($id)` 函数和一个 `findAll()` 函数。让我们在 Service 函数里面上面提到的函数：
	
	 <?php
	 // 文件名： /module/Blog/src/Blog/Service/PostService.php
	 namespace Blog\Service;
	
	 use Blog\Mapper\PostMapperInterface;
	
	 class PostService implements PostServiceInterface
	 {
	     /**
	      * @var \Blog\Mapper\PostMapperInterface
	      */
	     protected $postMapper;
	
	     /**
	      * @param PostMapperInterface $postMapper
	      */
	     public function __construct(PostMapperInterface $postMapper)
	     {
	         $this->postMapper = $postMapper;
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findAllPosts()
	     {
	         return $this->postMapper->findAll();
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function findPost($id)
	     {
	         return $this->postMapper->find($id);
	     }
	 }

看着这些代码，你就能发现我们使用 `postMapper` 来获取我们所需要的数据。这个过程是如何发生的再也不和 `PostService` 有任何关系。`PostService` 只知道他会接收到什么类型的数据，而这是唯一重要的事情。

## PostService 拥有依赖对象
现在我们介绍了 `PostMapperInterface` 是 `PostService` 的一个依赖对象，我们再也没办法将这个 Service 定义为 `invokable` 了，因为它有了依赖对象。所以我们需要为这个 Service 创建一个 Factory，就和我们为 `ListController` 做的一样。首先更改配置文件，将其从 `invokable` 数组中移动至 `factories` 数组，然后赋予合适的 factory 类，如下例：

 <?php
 // 文件名： /module/Blog/config/module.config.php
 return array(
     'service_manager' => array(
         'factories' => array(
             'Blog\Service\PostServiceInterface' => 'Blog\Factory\PostServiceFactory'
         )
     ),
     'view_manager' => array( /** ViewManager Config */ ),
     'controllers'  => array( /** ControllerManager Config */ ),
     'router'       => array( /** Router Config */ )
 );

完成上述配置文件之后我们需要创建 `Blog\Factory\PostServiceFactory` 类，所以现在我们来实现它：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Factory/PostServiceFactory.php
	 namespace Blog\Factory;
	
	 use Blog\Service\PostService;
	 use Zend\ServiceManager\FactoryInterface;
	 use Zend\ServiceManager\ServiceLocatorInterface;
	
	 class PostServiceFactory implements FactoryInterface
	 {
	     /**
	      * Create service
	      *
	      * @param ServiceLocatorInterface $serviceLocator
	      * @return mixed
	      */
	     public function createService(ServiceLocatorInterface $serviceLocator)
	     {
	         return new PostService(
	             $serviceLocator->get('Blog\Mapper\PostMapperInterface')
	         );
	     }
	 }

这个工作完成之后你现在应该能看到 `ServiceNotFoundException` 异常了，由 `ServiceManager` 抛出，告诉你所请求的 Service 无法被找到。
	
	 Additional information:
	 Zend\ServiceManager\Exception\ServiceNotFoundException
	 File:
	 {libraryPath}\Zend\ServiceManager\ServiceManager.php:529
	 Message:
	 Zend\ServiceManager\ServiceManager::get was unable to fetch or create an instance for Blog\Mapper\PostMapperInterface

## 总结
我们在此写上本章节的最终结语，事实上我们已经成功的将数据库操作逻辑隔离在我们的 Service 之外。现在，若情况需要的话，我们可以根据我们的需求来方便的对实际数据库操作进行变更了。
在下一章节我们会通过 `Zend\Db\Sql` 来创建关于 `PostMapperInterface ` 的实现。
