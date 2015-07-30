# 编辑数据和删除数据

在上一个章节中我们已经学习了如何使用 `Zend\Form` 组件和 `Zend\Db` 组件来编写建立新数据集的功能。这一章节会专注于介绍编辑数据与删除数据，从而完全实现增删改查等功能。我们首先从编辑数据开始。

## 为表单绑定数据

插入数据表单和编辑数据表单之间的一个根本性的区别是，事实上在编辑数据表单中，数据已经存在。这意味着我们需要先找到一个方法从数据库获得数据，再将其预先填入表单中。幸运地，`Zend\Form` 组件提供了一个非常方便的方法来实现这些功能，并将其称之为**数据绑定**。

当你提供一个编辑数据表单的时候，需要做的事情只有将你感兴趣的对象从你的服务中进行 `bind` 和表单绑定。下例演示了如何在你的控制器内实现这一点：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Controller/WriteController.php
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
	                     die($e->getMessage());
	                     // 发生了一些数据库错误，进行记录并且让用户知道
	                 }
	             }
	         }
	
	         return new ViewModel(array(
	             'form' => $this->postForm
	         ));
	     }
	
	     public function editAction()
	     {
	         $request = $this->getRequest();
	         $post    = $this->postService->findPost($this->params('id'));
	
	         $this->postForm->bind($post);
	
	         if ($request->isPost()) {
	             $this->postForm->setData($request->getPost());
	
	             if ($this->postForm->isValid()) {
	                 try {
	                     $this->postService->savePost($post);
	
	                     return $this->redirect()->toRoute('blog');
	                 } catch (\Exception $e) {
	                     die($e->getMessage());
	                     // Some DB Error happened, log it and let the user know
	                 }
	             }
	         }
	
	         return new ViewModel(array(
	             'form' => $this->postForm
	         ));
	     }
	 }

比起 `addAction()`，`editAction()` 只有三行代码是不一样的。第一个不一样的地方是曾经用来根据路径的 `id` 参数从服务获得相关的 `Post` 对象（我们即将会对这个部分进行编写）。

第二个不一样的地方是，新代码可以让你绑定数据到 `Zend\Form` 组件上。我们在这里之可以有效使用对象是因为我们的 `PostFieldset` 使用了 hydrator 来将对象中的数据进行处理并显示。

最后，比起真的去执行 `$form->getData()`，我们只需要简单地使用之前的 `$post` 变量就可以达到目的，因为这个变量会被自动更新成表单中最新的数据，这是数据绑定功能的功劳。这些就是所有要做的事情了，现在唯一要加的东西就是添加新的编辑数据路径和编写针对该操作的视图文件。

## 添加编辑数据路径

编辑数据路径不外乎是一个普通的段路径，和 `blog/detail` 没什么区别。配置你的路径配置文件来添加一个新路径：

	 <?php
	 // 文件名： /module/Blog/config/module.config.php
	 return array(
	     'db'              => array( /** Db Config */ ),
	     'service_manager' => array( /** ServiceManager Config */ ),
	     'view_manager'    => array( /** ViewManager Config */ ),
	     'controllers'     => array( /** ControllerManager Config* */ ),
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
	                     ),
	                     'edit' => array(
	                         'type' => 'segment',
	                         'options' => array(
	                             'route'    => '/edit/:id',
	                             'defaults' => array(
	                                 'controller' => 'Blog\Controller\Write',
	                                 'action'     => 'edit'
	                             ),
	                             'constraints' => array(
	                                 'id' => '\d+'
	                             )
	                         )
	                     ),
	                 )
	             )
	         )
	     )
	 );

## 创建编辑数据模板

下一个需要做的事情就是创建新的模板 `blog/write/edit`：

你需要对视图端做的所有改变，仅仅是将目前的 `id` 传给 `url()` viewHelper。要实现这点你有两种选择：第一种是将 ID 通过参数数组传值，如下例所示：

    $this->url('blog/edit', array('id' => $id));

这样做的缺点是 `$id` 变量不可用，因为我们没有将其指定给视图。然而 `Zend\Mvc\Router` 组件给了我们一个很不错的功能来重用目前已经匹配的参数。这可以通过设定 viewHelper 的最后一个参数为 `true` 实现：

    $this->url('blog/edit', array(), true);
 
#### 检查状态

如果现在打开你的浏览器并且在 `localhost:8080/blog/edit/1` 打开编辑数据表单，就可以看见表单已经包含你已经选中的博客帖子的数据。而且当你提交表单的时候就会发现数据已经被成功变更。然而悲剧的是提交按钮仍然包含这段文字 `Insert new Post` (插入新帖子)。这个问题也可以通过修改视图解决：

	 <!-- Filename: /module/Blog/view/blog/write/add.phtml -->
	 <h1>WriteController::editAction()</h1>
	 <?php
	 $form = $this->form;
	 $form->setAttribute('action', $this->url('blog/edit', array(), true));
	 $form->prepare();
	
	 $form->get('submit')->setValue('Update Post');
	
	 echo $this->form()->openTag($form);
	
	 echo $this->formCollection($form);
	
	 echo $this->form()->closeTag();

## 实现删除功能

最后终于是时候来删除一些数据了。我们从创建一个新路径和添加一个新控制器开始。

	 <?php
	 // 文件名： /module/Blog/config/module.config.php
	 return array(
	     'db'              => array( /** Db Config */ ),
	     'service_manager' => array( /** ServiceManager Config */ ),
	     'view_manager'    => array( /** ViewManager Config */ ),
	     'controllers'     => array(
	         'factories' => array(
	             'Blog\Controller\List'   => 'Blog\Factory\ListControllerFactory',
	             'Blog\Controller\Write'  => 'Blog\Factory\WriteControllerFactory',
	             'Blog\Controller\Delete' => 'Blog\Factory\DeleteControllerFactory'
	         )
	     ),
	     'router'          => array(
	         'routes' => array(
	             'post' => array(
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
	                     ),
	                     'edit' => array(
	                         'type' => 'segment',
	                         'options' => array(
	                             'route'    => '/edit/:id',
	                             'defaults' => array(
	                                 'controller' => 'Blog\Controller\Write',
	                                 'action'     => 'edit'
	                             ),
	                             'constraints' => array(
	                                 'id' => '\d+'
	                             )
	                         )
	                     ),
	                     'delete' => array(
	                         'type' => 'segment',
	                         'options' => array(
	                             'route'    => '/delete/:id',
	                             'defaults' => array(
	                                 'controller' => 'Blog\Controller\Delete',
	                                 'action'     => 'delete'
	                             ),
	                             'constraints' => array(
	                                 'id' => '\d+'
	                             )
	                         )
	                     ),
	                 )
	             )
	         )
	     )
	 );

请注意这里我们制定了又一个控制器 `Blog\Controller\Delete`，这是因为实际上这个控制器不会要求 `PostForm`。更进一步，甚至连 `Zend\Form` 组件都不要求使用的一个完美示例就是 `DeleteForm`。首先我们先创建我们的控制器：

**Factory**

	 <?php
	 // 文件名： /module/Blog/src/Blog/Factory/DeleteControllerFactory.php
	 namespace Blog\Factory;
	
	 use Blog\Controller\DeleteController;
	 use Zend\ServiceManager\FactoryInterface;
	 use Zend\ServiceManager\ServiceLocatorInterface;
	
	 class DeleteControllerFactory implements FactoryInterface
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
	
	         return new DeleteController($postService);
	     }
	 }

**Controller**

	 <?php
	 // 文件名： /module/Blog/src/Blog/Controller/DeleteController.php
	 namespace Blog\Controller;
	
	 use Blog\Service\PostServiceInterface;
	 use Zend\Mvc\Controller\AbstractActionController;
	 use Zend\View\Model\ViewModel;
	
	 class DeleteController extends AbstractActionController
	 {
	     /**
	      * @var \Blog\Service\PostServiceInterface
	      */
	     protected $postService;
	
	     public function __construct(PostServiceInterface $postService)
	     {
	         $this->postService = $postService;
	     }
	
	     public function deleteAction()
	     {
	         try {
	             $post = $this->postService->findPost($this->params('id'));
	         } catch (\InvalidArgumentException $e) {
	             return $this->redirect()->toRoute('blog');
	         }
	
	         $request = $this->getRequest();
	
	         if ($request->isPost()) {
	             $del = $request->getPost('delete_confirmation', 'no');
	
	             if ($del === 'yes') {
	                 $this->postService->deletePost($post);
	             }
	
	             return $this->redirect()->toRoute('blog');
	         }
	
	         return new ViewModel(array(
	             'post' => $post
	         ));
	     }
	 }

如您所见这里并没有什么新东西。我们将 `PostService` 注入到控制器，然后在 action 里面我们首先检查目标博客帖子是否存在。如果存在我们就检测请求是不是一个 POST 请求，如果是的话再看 POST 数据里面有没有一个叫做 `delete_confirmation` 参数存在，如果这个参数存在而且其值为 `yes`，那么就通过 `PostService` 的 `deletePost()` 函数对目标博客帖子进行删除。

在您实际编写这段代码的时候会注意到你没有得到关于 `deletePost()` 自动完成提示，这是因为我们还未将其添加到服务/接口上。现在我们将这个函数添加到接口上，并且在服务里面对其进行实现：

**Interface**

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
	
	     /**
	      * 应该会保存给出了的 PostInterface 实现并且返回。如果是已有的帖子那么帖子
	      * 应该被更新，如果是新帖子则应该去创建。
	      *
	      * @param  PostInterface $blog
	      * @return PostInterface
	      */
	     public function savePost(PostInterface $blog);
	
	     /**
	      * 应该删除给出的 PostInterface 的一个实现，如果删除成功就返回 true
	      * 否则返回 false.
	      *
	      * @param  PostInterface $blog
	      * @return bool
	      */
	     public function deletePost(PostInterface $blog);
	 }

**Service**

	 <?php
	 // 文件名： /module/Blog/src/Blog/Service/PostService.php
	 namespace Blog\Service;
	
	 use Blog\Mapper\PostMapperInterface;
	 use Blog\Model\PostInterface;
	
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
	
	     /**
	      * {@inheritDoc}
	      */
	     public function deletePost(PostInterface $post)
	     {
	         return $this->postMapper->delete($post);
	     }
	 }

现在我们认为 `PostMapperInterface` 应该有 `delete()` 函数。我们还没对其进行实现所以先将其加入 `PostMapperInterface`。

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
	
	     /**
	      * @param PostInterface $postObject
	      *
	      * @param PostInterface $postObject
	      * @return PostInterface
	      * @throws \Exception
	      */
	     public function save(PostInterface $postObject);
	
	     /**
	      * @param PostInterface $postObject
	      *
	      * @return bool
	      * @throws \Exception
	      */
	     public function delete(PostInterface $postObject);
	 }

现在我们已经在接口内声明了函数，是时候在 `ZendDbSqlMapper` 内对其进行实现了：

	 <?php
	 // 文件名： /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	 use Zend\Db\Adapter\AdapterInterface;
	 use Zend\Db\Adapter\Driver\ResultInterface;
	 use Zend\Db\ResultSet\HydratingResultSet;
	 use Zend\Db\Sql\Delete;
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
	
	     protected $hydrator;
	
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
	      * {@inheritDoc}
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
	      * {@inheritDoc}
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
	      * {@inheritDoc}
	      */
	     public function save(PostInterface $postObject)
	     {
	         $postData = $this->hydrator->extract($postObject);
	         unset($postData['id']); // Insert 和 Update 都不需要数组中存在 ID
	
	         if ($postObject->getId()) {
	             // ID 存在，是一个 Update
	             $action = new Update('post');
	             $action->set($postData);
	             $action->where(array('id = ?' => $postObject->getId()));
	         } else {
	             // ID 不存在，是一个Insert
	             $action = new Insert('post');
	             $action->values($postData);
	         }
	
	         $sql    = new Sql($this->dbAdapter);
	         $stmt   = $sql->prepareStatementForSqlObject($action);
	         $result = $stmt->execute();
	
	         if ($result instanceof ResultInterface) {
	             if ($newId = $result->getGeneratedValue()) {
	                 // 每当一个值被生成时，将其赋给对象
	                 $postObject->setId($newId);
	             }
	
	             return $postObject;
	         }
	
	         throw new \Exception("Database error");
	     }
	
	     /**
	      * {@inheritDoc}
	      */
	     public function delete(PostInterface $postObject)
	     {
	         $action = new Delete('posts');
	         $action->where(array('id = ?' => $postObject->getId()));
	
	         $sql    = new Sql($this->dbAdapter);
	         $stmt   = $sql->prepareStatementForSqlObject($action);
	         $result = $stmt->execute();
	
	         return (bool)$result->getAffectedRows();
	     }
	 }

`delete` 语句应该看上去十分熟悉，毕竟这事情和你之前做的所有其他类型查询基本是一样的工作。当这些都弄好了之后我们可以继续编写我们的视图文件，然后我们就能删除博客帖子了。

	 <!-- Filename: /module/Blog/view/blog/delete/delete.phtml -->
	 <h1>DeleteController::deleteAction()</h1>
	 <p>
	     Are you sure that you want to delete
	     '<?php echo $this->escapeHtml($this->post->getTitle()); ?>' by
	     '<?php echo $this->escapeHtml($this->post->getText()); ?>'?
	 </p>
	 <form action="<?php echo $this->url('blog/delete', array(), true) ?>" method="post">
	     <input type="submit" name="delete_confirmation" value="yes">
	     <input type="submit" name="delete_confirmation" value="no">
	 </form>

## 总结

在这个章节中我们学习了在 `Zend\Form` 组件中的数据绑定是如何工作的，并且通过它完成了我们的更新数据程序。然后我们还学了如何使用 HTML 表单和在不依赖 `Zend\Form` 组件的前提下检查表单数据，最终我们完成了一个完整的博客帖子增删改查示例。

在下个章节中我们会重新概括一次我们完成的所有事情。然后会谈谈我们使用的设计模式和回答在这整个教程实践中经常出现的一些问题。
