# 对 Form（表单）和 Fieldset（字段集）加以利用
到目前我们已经实现了从数据库读取数据。在现实生活中的应用程序这并不是十分实用，毕竟多数情况下我们至少需要实现完整的增删改查功能。最普遍的添加数据到数据库的方法是让用户将数据添加到 Web `<form>` 表单标签内提交，然后我们的应用程序将用户输入保存到后台。

## 核心组件
我们想要能够准确的实现目标，而 Zend Framework 提供了所有完成目标所需要的工具。在我们直接开始编码之前，我们有必要了解一下这个工作的两个核心组件。所以我们来看看这些组件是什么，并了解它们是怎么工作的。

### Zend\Form\Fieldset (字段集)
首先你需要知道的组件就是 `Zend\Form\Fieldset`。一个 `Fieldset` 就是一个包含可重用的元素集合。你将会使用 `Fieldset` 来存储前台输入以便让后台模型处理。每个 `Model` 都准备一个 `Fieldset` 通常被认为是良好实践。

`Fieldset` 组件，然而，并不是 `Form` 表单本身，意味着你不可以只使用 `Fieldset` 却不将其附着在 `Form` 组件上。这样子做的优势是你拥有一个可以重用的元素集合，可以用于任意多个 `Form` 组件上，而不需要为了 `Model` 重新声明所有输入，因为 `Model` 已经被 `Fieldset` 代表。

### Zend\Form\Form (表单)
这是你所需要的主要部件，而且你很可能已经听说过了。`Form` 组件是所有的 Web `<form>`元素的主要容器。你可以向其添加单个元素，也可以通过 `Fieldset` 的形式添加多个元素。

## 创建你的第一个 Fieldset
要解释 `Zend\Form` 组件是如何工作的，不如通过你自己实战编码体会来的深刻。所以我们直接切入正题，创建我们的 `Blog` 模组所需的所有表单。我们首先创建包含所有需要处理的输入元素的 `Fieldset` 用来处理我们的 `Blog` 数据。

- 你需要为 `id` 属性准备一个隐藏的输入域，仅仅在编辑和删除数据时有用。
- 你需要一个文本输入框来存放 `text` 属性。
- 你需要另一个文本输入框来存放 `title` 属性。

创建文件 `/module/Blog/src/Blog/Form/PostFieldset.php` 并且添加下述代码：

	<?php
	// Filename: /module/Blog/src/Blog/Form/PostFieldset.php
	namespace Blog\Form;
	
	use Zend\Form\Fieldset;
	
	class PostFieldset extends Fieldset
	{
	   public function __construct()
	   {
	      $this->add(array(
	         'type' => 'hidden',
	         'name' => 'id'
	      ));
	
	      $this->add(array(
	         'type' => 'text',
	         'name' => 'text',
	         'options' => array(
	           'label' => 'The Text'
	         )
	      ));
	
	      $this->add(array(
	         'type' => 'text',
	         'name' => 'title',
	         'options' => array(
	            'label' => 'Blog Title'
	         )
	      ));
	   }
	}

如您所见这个类是十分有用的。我们做的事情是让我们的类 extends `Zend\Form\Fieldset `，然后我们编写一个 `__construct() ` 函数并且添加所有我们需要的元素到字段集。这个 `fieldset` 现在就能随我们意愿用于任意多个表单中了。所以接下来让我们创建第一个 `Form` 表单吧。

## 创建 PostForm
现在我们已经准备好了我们的 `PostFieldset`，还需要在 `Form` 内使用它。我们接下来需要添加一个表单的提交按钮，这样用户就能够提交数据了。所以在同一个路径 `/module/Blog/src/Blog/Form/PostForm` 下创建 `PostForm`，并且将 `PostFieldset` 添加进去：

	 <?php
	 // Filename: /module/Blog/src/Blog/Form/PostForm.php
	 namespace Blog\Form;
	
	 use Zend\Form\Form;
	
	 class PostForm extends Form
	 {
	     public function __construct()
	     {
	         $this->add(array(
	             'name' => 'post-fieldset',
	             'type' => 'Blog\Form\PostFieldset'
	         ));
	
	         $this->add(array(
	             'type' => 'submit',
	             'name' => 'submit',
	             'attributes' => array(
	                 'value' => 'Insert new Post'
	             )
	         ));
	     }
	 }

这就是我们的表单了。并没有什么特别的，我们添加了 `PostFieldset` 到表单里，还添加了一个提交按钮，然后没别的了。现在我们来让这个表单发挥作用。

## 添加一个新 Post（帖子）
现在我们已经写好了 `PostForm`。现在想要使用它，还有几个任务需要完成。目前你要直接面对的任务是：

- 创建一个新的控制器 `WriteController`
- 添加一个 `PostService` 服务，将其设定为 `WriteController` 的依赖对象
- 添加一个 `PostForm` 表单，将其设定为 `WriteController` 的依赖对象
- 创建一个新的路径 `blog/add`，并让其转发到 `WriteController` 和它附属的 `addAction()`
- 创建一个新的视图用于显示表单

### 创建 WriteController（写控制器）
如您在任务清单上所见，我们需要一个新的控制器，而且这个控制器应该拥有两个依赖对象。一个依赖对象时 `PostService`，它也在 `ListController` 中被使用，而另一个依赖对象 `PostForm` 是全新的。由于在显示博客数据的时候，`PostFrom` 是一个 `ListController` 不需要的依赖对象，所以我们会创建一个新的控制器来让读和写两边的事务分离。首先，在配置文件中注册一个控制器工厂（controller-factory）：

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     'db'              => array( /** DB Config */ ),
	     'service_manager' => array( /** ServiceManager Config */),
	     'view_manager'    => array( /** ViewManager Config */ ),
	     'controllers'     => array(
	         'factories' => array(
	             'Blog\Controller\List'  => 'Blog\Factory\ListControllerFactory',
	             'Blog\Controller\Write' => 'Blog\Factory\WriteControllerFactory'
	         )
	     ),
	     'router'          => array( /** Router Config */ )
	 );

下一步就是编写 `WriteControllerFactory`。让 factory 返回 `WriteController` 并且在构造器中添加所需的依赖对象：

	 <?php
	 // Filename: /module/Blog/src/Blog/Factory/WriteControllerFactory.php
	 namespace Blog\Factory;
	
	 use Blog\Controller\WriteController;
	 use Zend\ServiceManager\FactoryInterface;
	 use Zend\ServiceManager\ServiceLocatorInterface;
	
	 class WriteControllerFactory implements FactoryInterface
	 {
	     public function createService(ServiceLocatorInterface $serviceLocator)
	     {
	         $realServiceLocator = $serviceLocator->getServiceLocator();
	         $postService        = $realServiceLocator->get('Blog\Service\PostServiceInterface');
	         $postInsertForm     = $realServiceLocator->get('FormElementManager')->get('Blog\Form\PostForm');
	
	         return new WriteController(
	             $postService,
	             $postInsertForm
	         );
	     }
	 }

在这个代码示例中，这里有几样事情需要注意。第一件，`WriteController` 暂时还不存在，不过我们会在下一步创建这个控制器所以我们先假设其稍后会存在。第二件事情，我们通过访问 `FormElementManager` 来获得对 `PostForm` 的读写。所有表单都应该通过 `FormElementManager` 来访问。即使我们还没有在配置文件中注册 `PostForm`，`FormElementManager` 也会自动认出表单并且将其当做 `invokables` 对象，只要你的对象没有依赖对象，你便不需要显式地注册它。

下一步，创建我们的控制器。请确保通过输入依赖对象的接口并且添加 `addAction()` 函数来提示依赖对象。

	 <?php
	 // Filename: /module/Blog/src/Blog/Controller/WriteController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Form\FormInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	
	 class WriteController extends AbstractActionController
	 {
	     protected $postService;
	
	     protected $postForm;
	
	     public function __construct(
	         PostServiceInterface $postService,
	         FormInterface $postForm
	     ) {
	         $this->postService = $postService;
	         $this->postForm    = $postForm;
	     }
	
	     public function addAction()
	     {
	     }
	 }

接下来创建新路径：

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     'db'              => array( /** Db Config */ ),
	     'service_manager' => array( /** ServiceManager Config */ ),
	     'view_manager'    => array( /** ViewManager Config */ ),
	     'controllers'     => array( /** Controller Config */ ),
	     'router'          => array(
	         'routes' => array(
	             'blog' => array(
	                 'type' => 'literal',
	                 'options' => array(
	                     'route'    => '/blog',
	                     'defaults' => array(
	                         'controller' => 'Blog\Controller\List',
	                         'action'     => 'index',
	                     )
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
	                                 'id' => '\d+'
	                             )
	                         )
	                     ),
	                     'add' => array(
	                         'type' => 'literal',
	                         'options' => array(
	                             'route'    => '/add',
	                             'defaults' => array(
	                                 'controller' => 'Blog\Controller\Write',
	                                 'action'     => 'add'
	                             )
	                         )
	                     )
	                 )
	             )
	         )
	     )
	 );

最后我们创建一个没实际作用的模板文件：

	 <!-- Filename: /module/Blog/view/blog/write/add.phtml -->
	 <h1>WriteController::addAction()</h1>

#### 检查当前状态
如果你视图访问新的路径 `localhost:8080/blog/add`，那么你应该会看见下面这样的错误信息：

	 Fatal error: Call to a member function insert() on a non-object in
	 {libraryPath}/Zend/Form/Fieldset.php on line {lineNumber}

如果你看见的和上面写出来的不一样，那么请回过头认真检查一下先前的步骤是否准确地跟随着教程，并且检查你所有的文件。接下来，假设你已经看见了这个错误信息，让我们来寻找原因并且解决它！

上述错误是非常常见的，但是它的解决方法却没有那么直观。它看上去像是 `Zend/Form/Fieldset.php` 中出现了一个错误，但实际上却不是这个情况。这个错误信息让你知道了你在创建你的表单的时候有一些事情出现了差错。事实上，当同时创建 `PostForm` 表单和 `PostFieldset` 的时候，我们忘记了一些非常，非常重要的事情。

>**注意**：当重写 `Zend\Form` 组件的 `__construct()`  函数的时候，请永远不要忘记调用 `parent::__construct()`！

由于缺少了 `parent::__construct()` 调用，表单和字段集都不能正确的初始化。让我们通过在表单和字段级中调用父级构造器来修正这个问题。为了能拥有更好的可伸缩性我们也会包含能够接收多个参数的 `__construct()`  函数的签名。

	 <?php
	 // Filename: /module/Blog/src/Blog/Form/PostForm.php
	 namespace Blog\Form;
	
	 use Zend\Form\Form;
	
	 class PostForm extends Form
	 {
	     public function __construct($name = null, $options = array())
	     {
	         parent::__construct($name, $options);
	
	         $this->add(array(
	             'name' => 'post-fieldset',
	             'type' => 'Blog\Form\PostFieldset'
	         ));
	
	         $this->add(array(
	             'type' => 'submit',
	             'name' => 'submit',
	             'attributes' => array(
	                 'value' => 'Insert new Post'
	             )
	         ));
	     }
	 }

如您所见我们的 `PostForm` 现在接受两个参数分别定义我们的表单的名字和一些列的设置。两个参数都会被传给父对象。如果你仔细观察我们是如何添加 `PostFieldset` 的，便会发现我们为字段集赋予了一个名字。这些选项都会在 `PostFieldset` 创建时通过 `FormElementManager` 传出。不过要让这些正常工作，我们需要在字段集里面做同样的工作：

 <?php
 // Filename: /module/Blog/src/Blog/Form/PostFieldset.php
 namespace Blog\Form;

 use Zend\Form\Fieldset;

 class PostFieldset extends Fieldset
 {
     public function __construct($name = null, $options = array())
     {
         parent::__construct($name, $options);

         $this->add(array(
             'type' => 'hidden',
             'name' => 'id'
         ));

         $this->add(array(
             'type' => 'text',
             'name' => 'text',
             'options' => array(
                 'label' => 'The Text'
             )
         ));

         $this->add(array(
             'type' => 'text',
             'name' => 'title',
             'options' => array(
                 'label' => 'Blog Title'
             )
         ));
     }
 }

重新载入你的应用程序，你便可以看见你想要的结果了。

## 显示表单
现在我们在 `WriteController` 里有了我们的 `PostForm `，是时候将这个表单传递给视图，并让其通过指定的来自 `Zend\Form` 组件的 `ViewHelpers` 来进行渲染。首先修改你的控制器，让表单被传递到视图。

	  <?php
	 // Filename: /module/Blog/src/Blog/Controller/WriteController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Form\FormInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class WriteController extends AbstractActionController
	 {
	     protected $postService;
	
	     protected $postForm;
	
	     public function __construct(
	         PostServiceInterface $postService,
	         FormInterface $postForm
	     ) {
	         $this->postService = $postService;
	         $this->postForm    = $postForm;
	     }
	
	     public function addAction()
	     {
	         return new ViewModel(array(
	             'form' => $this->postForm
	         ));
	     }
	 }

然后我们需要修改视图来让表单得以正确渲染：
	
	 <!-- Filename: /module/Blog/view/blog/write/add.phtml -->
	 <h1>WriteController::addAction()</h1>
	 <?php
	 $form = $this->form;
	 $form->setAttribute('action', $this->url());
	 $form->prepare();
	
	 echo $this->form()->openTag($form);
	
	 echo $this->formCollection($form);
	
	 echo $this->form()->closeTag();

首先，我们告诉了表单它应该将它的数据发送给目前的 URL 然后我们告诉表单让其 `prepare()`（准备）自己（这个函数会触发一些内部操作）。

>**注意**： HTML 表单可以通过 `POST` 或者 `GET` 方式来进行传输。 ZF2 默认是使用 `POST`，所以你不需要对此进行显式的设定。但是如果你希望使用 `GET` 方式，只需要在调用 `prepare()` 之前设置好这个特定的属性： `$form->setAttribute('method', 'GET');`

接下类我们会使用几个 `ViewHelpers` 来负责帮我们渲染表单。使用 Zend Framework 渲染表单的方法有很多种，不过使用 `formCollection()` 可能是最快的方法。

刷新您的浏览器，现在就能看见你的表单被正确显示出来了。然而，现在提交表单的话，我们只能看见先前提交的表单原封不动的回显出来。很简单，这是因为我们还没有为控制器添加任何逻辑。

>**注意**：请记住这个教程仅仅聚焦于面向对象编程视角。像这样子渲染表单，不应用任何样式表是无法反映出绝大多数设计师关于一个美丽的表单的想法的。您将会在 [Zend\Form\View\Helper](http://framework.zend.com/manual/current/en/modules/zend.form.view.helpers.html#zend-form-view-helpers) 章节中学习到更多关于表单渲染的内容。

## 几乎适用于所有类型表单的控制器逻辑
编写一个控制器来处理表单工作流是非常简单的，而且基本上对于应用程序中每一种表单的手段都是一样的。

1. 你会想先检查目前的请求是否一个 POST 请求，这意味着确认表单是否被发出。
2. 如果表单已经被发出，你会想要：
	- 将表单的 POST 数据存放起来
	- 检查这个表单数据是否合法
3. 如果表单通过了检测，你会想要：
	- 将表单数据传给你的服务进行处理
	- 将用户重定向到刚才输入的数据的详情页面或者一些概览页面
4. 针对所有其他情形，你会希望表单继续被显示，有时会伴随一些错误信息

然而要实现上述所有功能并不需要你想象中那么多的代码。首先，按照下例修改你的 `WriteController` 代码：

	 <?php
	 // Filename: /module/Blog/src/Blog/Controller/WriteController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Form\FormInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class WriteController extends AbstractActionController
	 {
	     protected $postService;
	
	     protected $postForm;
	
	     public function __construct(
	         PostServiceInterface $postService,
	         FormInterface $postForm
	     ) {
	         $this->postService = $postService;
	         $this->postForm    = $postForm;
	     }
	
	     public function addAction()
	     {
	         $request = $this->getRequest();
	
	         if ($request->isPost()) {
	             $this->postForm->setData($request->getPost());
	
	             if ($this->postForm->isValid()) {
	                 try {
	                     $this->postService->savePost($this->postForm->getData());
	
	                     return $this->redirect()->toRoute('blog');
	                 } catch (\Exception $e) {
	                     // Some DB Error happened, log it and let the user know
	                 }
	             }
	         }
	
	         return new ViewModel(array(
	             'form' => $this->postForm
	         ));
	     }
	 }

这个示例代码应该十分简单明了。首先我们将目前的请求保存到一个本地变量中。然后我们检查目前的请求是不是一个 POST 请求，如果是，将请求的 POST 数据存储到表单中。如果表单经过检查之后确认是有效的，我们就试图将表单数据通过我们的服务进行保存，然后将用户重定向到路径 `blog`。如果在这途中任何时候出现了任何错误，我们就再次将表单显示出来。

现在提交表单的话，我们会遇到下述错误：

	 Fatal error: Call to undefined method Blog\Service\PostService::savePost() in
	 /module/Blog/src/Blog/Controller/WriteController.php on line 33

我们通过拓展 `PostService` 来修复这个错误，同时不要忘记更改 `PostServiceInterface` 的签名！

	 <?php
	 // Filename: /module/Blog/src/Blog/Service/PostServiceInterface.php
	 namespace Blog\Service;
	
	 use Blog\Model\PostInterface;
	
	 interface PostServiceInterface
	 {
	     /**
	      * Should return a set of all blog posts that we can iterate over. Single entries of the array are supposed to be
	      * implementing \Blog\Model\PostInterface
	      *
	      * @return array|PostInterface[]
	      */
	     public function findAllPosts();
	
	     /**
	      * Should return a single blog post
	      *
	      * @param  int $id Identifier of the Post that should be returned
	      * @return PostInterface
	      */
	     public function findPost($id);
	
	     /**
	      * Should save a given implementation of the PostInterface and return it. If it is an existing Post the Post
	      * should be updated, if it's a new Post it should be created.
	      *
	      * @param  PostInterface $blog
	      * @return PostInterface
	      */
	     public function savePost(PostInterface $blog);
	 }

如您所见 `savePost()` 函数已经被添加，并且需要在 `PostService` 里被实现。

	 <?php
	 // Filename: /module/Blog/src/Blog/Service/PostService.php
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
	
	     /**
	      * {@inheritDoc}
	      */
	     public function savePost(PostInterface $post)
	     {
	         return $this->postMapper->save($post);
	     }
	 }

现在我们对 `postMapper` 做出了假设，所以需要扩展我们的 `PostMapperInterface` 接口以及其实现。首先我们拓展接口：

	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/PostMapperInterface.php
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
	
	     /**
	      * @param PostInterface $postObject
	      *
	      * @param PostInterface $postObject
	      * @return PostInterface
	      * @throws \Exception
	      */
	     public function save(PostInterface $postObject);
	 }

然后我们实现 save 函数：

	<?php
	// Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	namespace Blog\Mapper;
	
	use Blog\Model\PostInterface;
	use Zend\Db\Adapter\AdapterInterface;
	use Zend\Db\Adapter\Driver\ResultInterface;
	use Zend\Db\ResultSet\HydratingResultSet;
	use Zend\Db\Sql\Insert;
	use Zend\Db\Sql\Sql;
	use Zend\Db\Sql\Update;
	use Zend\Stdlib\Hydrator\HydratorInterface;
	
	class ZendDbSqlMapper implements PostMapperInterface
	{
	   /**
	    * @var \Zend\Db\Adapter\AdapterInterface
	    */
	   protected $dbAdapter;
	
	   /**
	    * @var \Zend\Stdlib\Hydrator\HydratorInterface
	    */
	   protected $hydrator;
	
	   /**
	    * @var \Blog\Model\PostInterface
	    */
	   protected $postPrototype;
	
	   /**
	    * @param AdapterInterface  $dbAdapter
	    * @param HydratorInterface $hydrator
	    * @param PostInterface    $postPrototype
	    */
	   public function __construct(
	      AdapterInterface $dbAdapter,
	      HydratorInterface $hydrator,
	      PostInterface $postPrototype
	   ) {
	      $this->dbAdapter      = $dbAdapter;
	      $this->hydrator       = $hydrator;
	      $this->postPrototype  = $postPrototype;
	   }
	
	   /**
	    * @param int|string $id
	    *
	    * @return PostInterface
	    * @throws \InvalidArgumentException
	    */
	   public function find($id)
	   {
	      $sql    = new Sql($this->dbAdapter);
	      $select = $sql->select('posts');
	      $select->where(array('id = ?' => $id));
	
	      $stmt   = $sql->prepareStatementForSqlObject($select);
	      $result = $stmt->execute();
	
	      if ($result instanceof ResultInterface && $result->isQueryResult() && $result->getAffectedRows()) {
	         return $this->hydrator->hydrate($result->current(), $this->postPrototype);
	      }
	
	      throw new \InvalidArgumentException("Blog with given ID:{$id} not found.");
	   }
	
	   /**
	    * @return array|PostInterface[]
	    */
	   public function findAll()
	   {
	      $sql    = new Sql($this->dbAdapter);
	      $select = $sql->select('posts');
	
	      $stmt   = $sql->prepareStatementForSqlObject($select);
	      $result = $stmt->execute();
	
	      if ($result instanceof ResultInterface && $result->isQueryResult()) {
	         $resultSet = new HydratingResultSet($this->hydrator, $this->postPrototype);
	
	         return $resultSet->initialize($result);
	      }
	
	      return array();
	   }
	
	   /**
	    * @param PostInterface $postObject
	    *
	    * @return PostInterface
	    * @throws \Exception
	    */
	   public function save(PostInterface $postObject)
	   {
	      $postData = $this->hydrator->extract($postObject);
	      unset($postData['id']); // Neither Insert nor Update needs the ID in the array
	
	      if ($postObject->getId()) {
	         // ID present, it's an Update
	         $action = new Update('posts');
	         $action->set($postData);
	         $action->where(array('id = ?' => $postObject->getId()));
	      } else {
	         // ID NOT present, it's an Insert
	         $action = new Insert('posts');
	         $action->values($postData);
	      }
	
	      $sql    = new Sql($this->dbAdapter);
	      $stmt   = $sql->prepareStatementForSqlObject($action);
	      $result = $stmt->execute();
	
	      if ($result instanceof ResultInterface) {
	         if ($newId = $result->getGeneratedValue()) {
	            // When a value has been generated, set it on the object
	            $postObject->setId($newId);
	         }
	
	         return $postObject;
	      }
	
	      throw new \Exception("Database error");
	   }
	}
在这里 `save()` 函数处理了两种情况：`insert` 和 `update` 流程。首先我们提取 `Post` 对象，因为我们需要数组数据来实现 `Insert` 和 `Update`。然后，我们从数组中删除了 `id` ，因为对一个元组进行更新的时候，我们不需要更新 `id` 属性；同时，我们插入一个新元组的时候也不需要 `id` 字段，所以两种情况均不需要 `id` 这个字段，将其简单去除即可。

在我们去除了 `id` 字段之后，检查那些动作需要被调用。如果 `Post` 对象拥有一个 `id` 集，我们便创建一个新的 `Update` 对象，否则我们创建一个 `Insert` 对象。我们将数据传给合适的 action 然后数据会被传给 `Sql` 对象，最终进行真正的数据库操作。

最后，我们检查我们是否接收到一个有效的结果，检查一下有没有新产生的 `id`。如果是的话，我们调用我们博客的 `setId()` 函数并且将对象返回。

让我们再次提将我们的表单，看看这次会得到什么。

	 Catchable fatal error: Argument 1 passed to Blog\Service\PostService::savePost()
	 must implement interface Blog\Model\PostInterface, array given,
	 called in /module/Blog/src/Blog/Controller/InsertController.php on line 33
	 and defined in /module/Blog/src/Blog/Service/PostService.php on line 49

表单，默认的时候，会将数据以数组形式传给你。不过我们的 `PostService` 却期待接收到的对象是 `PostInterface` 的一个实现。这意味着我们需要找到一个方法来将这个数组数据转换成对象数据。如果你还记得上一章节，就会知道要通过充水器实现。

>**注意**：在更新查询中，你会注意到我们添加了一个条件语句让其只更新与给出的 id 匹配的元组：
>`$action->where(array('id = ?' => $postObject->getId()));`
>你会看见条件是：**id equals ?**。这个问号代表着 POST 对象的 id。同样的，你可以添加一个条件语句来更新（或者选择）所有大于给定 id 的元组：
>`$action->where(array('id > ?' => $postObject->getId()));`
>这些操作符可以用于所有类型的条件语句：`=`、`>`、`<`、`>=` 和 `<=`。

## 让 Zend\Form 和 Zend\Stdlib\Hydrator 协调工作
在我们继续前进并且将充水器放进表单之前，先让我们对表单的数据做一个 dump。这样做可以让我们很方便的注意到充水器做的所有变更。根据下例修改你的 `WriteController`：

	 <?php
	 // Filename: /module/Blog/src/Blog/Controller/WriteController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Form\FormInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class WriteController extends AbstractActionController
	 {
	     protected $postService;
	
	     protected $postForm;
	
	     public function __construct(
	         PostServiceInterface $postService,
	         FormInterface $postForm
	     ) {
	         $this->postService = $postService;
	         $this->postForm    = $postForm;
	     }
	
	     public function addAction()
	     {
	         $request = $this->getRequest();
	
	         if ($request->isPost()) {
	             $this->postForm->setData($request->getPost());
	
	             if ($this->postForm->isValid()) {
	                 try {
	                     \Zend\Debug\Debug::dump($this->postForm->getData());die();
	                     $this->postService->savePost($this->postForm->getData());
	
	                     return $this->redirect()->toRoute('blog');
	                 } catch (\Exception $e) {
	                     // Some DB Error happened, log it and let the user know
	                 }
	             }
	         }
	
	         return new ViewModel(array(
	             'form' => $this->postForm
	         ));
	     }
	 }

做好之后请再次提交表单。你现在应该能看到数据 dump，和下例差不多的情形：

	 array(2) {
	   ["submit"] => string(16) "Insert new Post"
	   ["post-fieldset"] => array(3) {
	     ["id"] => string(0) ""
	     ["text"] => string(3) "foo"
	     ["title"] => string(3) "bar"
	   }
	 }

现在让你的字段集将数据注水成 `Post` 对象是非常简单的。你需要做的事情仅仅是指定注水器和对象原型，如下例所示：

 <?php
 // Filename: /module/Blog/src/Blog/Form/PostFieldset.php
 namespace Blog\Form;

 use Blog\Model\Post;
 use Zend\Form\Fieldset;
 use Zend\Stdlib\Hydrator\ClassMethods;

 class PostFieldset extends Fieldset
 {
     public function __construct($name = null, $options = array())
     {
         parent::__construct($name, $options);

         $this->setHydrator(new ClassMethods(false));
         $this->setObject(new Post());

         $this->add(array(
             'type' => 'hidden',
             'name' => 'id'
         ));

         $this->add(array(
             'type' => 'text',
             'name' => 'text',
             'options' => array(
                 'label' => 'The Text'
             )
         ));

         $this->add(array(
             'type' => 'text',
             'name' => 'title',
             'options' => array(
                 'label' => 'Blog Title'
             )
         ));
     }
 }

如您所见我们做了两件事情，我们告知了字段集使用 `ClassMethods` 充水器，然后我们还告知了它应该返回给我们 `Blog` 模型、不过，当你再次提交表单的时候你会注意到什么都没有改变。我们仍然只得到数组数据，而不是对象。

这是因为事实上表单本身不知道自己需要返回一个对象。当表单不知道自己要返回什么的时候就会默认递归使用 `ArraySeriazable` 充水器。要改变这点，我们需要让我们的 `PostFieldset` 变成所谓的 `base_fieldset`。

`base_fieldset` 基本上就告诉了表单“这个表单是关于我的，请不要操心其他数据，只操心我就好”。而且当表单意识到这个字段集是来真的，它就会乖乖使用字段集提供的冲水器，并且将我们想要的对象返回出来。修改你的 `PostForm` 并且将 `PostFieldset` 设置成 `base_fieldset`：

	 <?php
	 // Filename: /module/Blog/src/Blog/Form/PostForm.php
	 namespace Blog\Form;
	
	 use Zend\Form\Form;
	
	 class PostForm extends Form
	 {
	     public function __construct($name = null, $options = array())
	     {
	         parent::__construct($name, $options);
	
	         $this->add(array(
	             'name' => 'post-fieldset',
	             'type' => 'Blog\Form\PostFieldset',
	             'options' => array(
	                 'use_as_base_fieldset' => true
	             )
	         ));
	
	         $this->add(array(
	             'type' => 'submit',
	             'name' => 'submit',
	             'attributes' => array(
	                 'value' => 'Insert new Post'
	             )
	         ));
	     }
	 }

现在再次提交你的表单。你应该能看见如下输出：

	 object(Blog\Model\Post)#294 (3) {
	   ["id":protected] => string(0) ""
	   ["title":protected] => string(3) "foo"
	   ["text":protected] => string(3) "bar"
	 }

现在你可以回滚你的 `WriteController` 到它的先前状态了，来让表单数据传给 `PostService`。

	 <?php
	 // Filename: /module/Blog/src/Blog/Controller/WriteController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Form\FormInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class WriteController extends AbstractActionController
	 {
	     protected $postService;
	
	     protected $postForm;
	
	     public function __construct(
	         PostServiceInterface $postService,
	         FormInterface $postForm
	     ) {
	         $this->postService = $postService;
	         $this->postForm    = $postForm;
	     }
	
	     public function addAction()
	     {
	         $request = $this->getRequest();
	
	         if ($request->isPost()) {
	             $this->postForm->setData($request->getPost());
	
	             if ($this->postForm->isValid()) {
	                 try {
	                     $this->postService->savePost($this->postForm->getData());
	
	                     return $this->redirect()->toRoute('blog');
	                 } catch (\Exception $e) {
	                     // Some DB Error happened, log it and let the user know
	                 }
	             }
	         }
	
	         return new ViewModel(array(
	             'form' => $this->postForm
	         ));
	     }
	 }

你现在只需要再次发送表单就能够随心所欲任意添加博客帖子了，不错！

## 总结
在这个章节中，你学习了许多关于 `Zend\Form` 组件的知识。你也知道了 `Zend\Stdlib\Hydrator` 在 `Zend\Form` 组件扮演多么重要的角色，并且通过使用这两个组件你已经能为博客模组创建插入表单。

再下一个章节中，我们会创建更新和删除程序，从而完成博客模组的所有增删改查功能。