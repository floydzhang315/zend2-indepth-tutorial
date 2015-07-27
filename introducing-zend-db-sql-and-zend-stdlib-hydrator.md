# 介绍 Zend\Db\Sql 和 Zend\Stdlib\Hydrator
在上一个章节中我们介绍了映射层并且创建了 `PostMapperInterface`。现在是时候将这个接口进行实现了，以便让我们能再次使用 `PostService`。作为一个指导性示例，我们会使用 `Zend\Db\Sql` 类。

## 准备数据库
在我们能开始使用数据库之前，我们应该先准备一个。在这个示例中我们会使用一个 MySQL 数据库，名称为 `blog`，并且可以在 `localhost` 上被访问。这个数据库会拥有一个叫做 `posts` 的表，表拥有三个属性 `id`、`title`、`text`，其中`id`是主键。为了演示需要，请使用这个数据库 dump：

	 CREATE TABLE posts (
	   id int(11) NOT NULL auto_increment,
	   title varchar(100) NOT NULL,
	   text TEXT NOT NULL,
	   PRIMARY KEY (id)
	 );
	
	 INSERT INTO posts (title, text)
	   VALUES  ('Blog #1',  'Welcome to my first blog post');
	 INSERT INTO posts (title, text)
	   VALUES  ('Blog #2',  'Welcome to my second blog post');
	 INSERT INTO posts (title, text)
	   VALUES  ('Blog #3',  'Welcome to my third blog post');
	 INSERT INTO posts (title, text)
	   VALUES  ('Blog #4',  'Welcome to my fourth blog post');
	 INSERT INTO posts (title, text)
	   VALUES  ('Blog #5',  'Welcome to my fifth blog post');

## Zend\Db\Sql 的一些小知识
要使用 `Zend\Db\Sql` 来查询一个数据库，你需要拥有一个可用的数据库连接。这个链接使用过任何实现 `Zend\Db\Adapter\AdapterInterface` 接口的类创建的。最方便的创建这种类的方法是通过使用（监听配置键 `db` 的） `Zend\Db\Adapter\AdapterServiceFactory`。让我们从创建所需配置条目开始，修改你的 `module.config.php ` 文件，添加一个顶级键 `db`：

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     'db' => array(
	         'driver'         => 'Pdo',
	         'username'       => 'SECRET_USERNAME',  //edit this
	         'password'       => 'SECRET_PASSWORD',  //edit this
	         'dsn'            => 'mysql:dbname=blog;host=localhost',
	         'driver_options' => array(
	             \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
	         )
	     ),
	     'service_manager' => array( /** ServiceManager Config */ ),
	     'view_manager'    => array( /** ViewManager Config */ ),
	     'controllers'     => array( /** ControllerManager Config */ ),
	     'router'          => array( /** Router Config */ )
	 );

如您所见我们已经添加了 `db` 键，并且在里面我们添加了必要的参数来创建一个驱动示例。

>**注意**：一个需要注意的重要事情是，通常你**不会**希望你的凭证存放在一个普通的配置文件中，而是希望存放在本地配置文件中，例如 `/config/autoload/db.local.php`。在使用 zend 骨架 .gitignore 文件标记时，存放在本地的文件**不会**被推送到服务器。当您共享您的代码时请务必注意这点。参考下例代码：

	<?php
	// Filename: /config/autoload/db.local.php
	return array(
	        'db' => array(
	            'driver'         => 'Pdo',
	            'username'       => 'SECRET_USERNAME',  //edit this
	            'password'       => 'SECRET_PASSWORD',  //edit this
	            'dsn'            => 'mysql:dbname=blog;host=localhost',
	            'driver_options' => array(
	                \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
	        )
	    ),
	);


接下来我们要做的事情就是利用 `AdapterServiceFactory`。这是一个 `ServiceManager` 条目，看上去像下面这样：

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     'db' => array(
	         'driver'         => 'Pdo',
	         'username'       => 'SECRET_USERNAME',  //edit this
	         'password'       => 'SECRET_PASSWORD',  //edit this
	         'dsn'            => 'mysql:dbname=blog;host=localhost',
	         'driver_options' => array(
	             \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
	         )
	     ),
	     'service_manager' => array(
	         'factories' => array(
	             'Blog\Service\PostServiceInterface' => 'Blog\Service\Factory\PostServiceFactory',
	             'Zend\Db\Adapter\Adapter'           => 'Zend\Db\Adapter\AdapterServiceFactory'
	         )
	     ),
	     'view_manager'    => array( /** ViewManager Config */ ),
	     'controllers'     => array( /** ControllerManager Config */ ),
	     'router'          => array( /** Router Config */ )
	 );

请注意名为 `Zend\Db\Adapter\Adapter` 的新 Service。现在调用这个 Service 终会得到一个正在运行的，实现 `Zend\Db\Adapter\AdapterInterface` 接口的一个示例，这取决于我们指定的驱动。

当 Adapter 就位时，我们现在可以对数据库进行查询了。查询的构造最好通过 `Zend\Db\Sql` 的 “QueryBuilder” 功能完成，若要进行 select 查询则使用 `Zend\Db\Sql\Sql`；若要进行 insert 查询则使用 `Zend\Db\Sql\Insert`；若要进行 update 或者 delete 查询则分别使用 `Zend\Db\Sql\Update` 和 `Zend\Db\Sql\Delete`。这些组件的基本工作流是：

1. 建立一个使用 `Sql`、`Insert`、`Update` 或 `Delete`的查询
2. 从 `Sql` 对象创建一个 SQL 语句
3. 执行查询
4. 对结果做点什么

知道这些之后我们现在可以编写 `PostMapperInterface` 接口的实现了。

## 编写映射器的实现
我们的映射器实现会存放在和它的接口同样的名称空间内。现在开始创建一个类，称之为 `ZendDbSqlMapper` 然后 implements `PostMapperInterface`。

现在回想我们之前学到的东西。为了让 `Zend\Db\Sql` 能工作我们需要一个可用的 `AdapterInterface` 接口的实现。这是一个要求，所以会通过构造器注入进行注入。创建一个 `__construct()` 函数来接收 `AdapterInterface` 作为参数，并且将其存放在类中：

	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	 use Zend\Db\Adapter\AdapterInterface;
	
	 class ZendDbSqlMapper implements PostMapperInterface
	 {
	     /**
	      * @var \Zend\Db\Adapter\AdapterInterface
	      */
	     protected $dbAdapter;
	
	     /**
	      * @param AdapterInterface  $dbAdapter
	      */
	     public function __construct(AdapterInterface $dbAdapter)
	     {
	         $this->dbAdapter = $dbAdapter;
	     }
	
	     /**
	      * @param int|string $id
	      *
	      * @return PostInterface
	      * @throws \InvalidArgumentException
	      */
	     public function find($id)
	     {
	     }
	
	     /**
	      * @return array|PostInterface[]
	      */
	     public function findAll()
	     {
	     }
	 }

如您从以往的课程学到的内容，每当我们被要求参数，就要为这个类编写 factory。所以现在去为我们的映射器实现创建一个 factory 吧。

现在我们可以将映射器实现注册成一个 Service 了。如果你能回想起以往的章节，或者你有去查看一下当前的错误信息，你就会注意到我们通过调用 `Blog\Mapper\PostMapperInterface` Service 来获取映射器的实现。修改配置文件，以便让这个键能调用我们刚调用的 factory 类。

	 <?php
	 // Filename: /module/Blog/config/module.config.php
	 return array(
	     'db'              => array( /** Db Config */ ),
	     'service_manager' => array(
	         'factories' => array(
	             'Blog\Mapper\PostMapperInterface'   => 'Blog\Factory\ZendDbSqlMapperFactory',
	             'Blog\Service\PostServiceInterface' => 'Blog\Service\Factory\PostServiceFactory',
	             'Zend\Db\Adapter\Adapter'           => 'Zend\Db\Adapter\AdapterServiceFactory'
	         )
	     ),
	     'view_manager'    => array( /** ViewManager Config */ ),
	     'controllers'     => array( /** ControllerManager Config */ ),
	     'router'          => array( /** Router Config */ )
	 );

当 adapter 就绪之后，你就能刷新博客站点 `localhost:8080/blog`，然后发现 `ServiceNotFoundException` 异常已经消失，取而代之的是一个 PHP 警告信息：

	Warning: Invalid argument supplied for foreach() in /module/Blog/view/blog/list/index.phtml on line 13
	 ID  Text    Title

这是因为实际上我们的映射器还不会返回任何东西。让我们来修改一下 `findAll()` 函数来从数据库表中返回所有博客帖子。

	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Zend\Db\Adapter\AdapterInterface;
	
	 class ZendDbSqlMapper implements PostMapperInterface
	 {
	     /**
	      * @var \Zend\Db\Adapter\AdapterInterface
	      */
	     protected $dbAdapter;
	
	     /**
	      * @param AdapterInterface  $dbAdapter
	      */
	     public function __construct(AdapterInterface $dbAdapter)
	     {
	         $this->dbAdapter = $dbAdapter;
	     }
	
	     /**
	      * @param int|string $id
	      *
	      * @return \Blog\Entity\PostInterface
	      * @throws \InvalidArgumentException
	      */
	     public function find($id)
	     {
	     }
	
	     /**
	      * @return array|\Blog\Entity\PostInterface[]
	      */
	     public function findAll()
	     {
	         $sql    = new Sql($this->dbAdapter);
	         $select = $sql->select('posts');
	
	         $stmt   = $sql->prepareStatementForSqlObject($select);
	         $result = $stmt->execute();
	
	         return $result;
	     }
	 }

上述代码应该看上去十分直接。可惜的是，再次刷新页面会发现另外一个错误信息。

让我们暂时先不要返回 `$result` 变量，然后生成一个 dump 来看看到底这里发生了什么。更改 `findAll()` 函数来做一个 `$result` 变量的数据 dump：

	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	 use Zend\Db\Adapter\AdapterInterface;
	 use Zend\Db\Sql\Sql;
	
	 class ZendDbSqlMapper implements PostMapperInterface
	 {
	     /**
	      * @var \Zend\Db\Adapter\AdapterInterface
	      */
	     protected $dbAdapter;
	
	     /**
	      * @param AdapterInterface  $dbAdapter
	      */
	     public function __construct(AdapterInterface $dbAdapter)
	     {
	         $this->dbAdapter = $dbAdapter;
	     }
	
	     /**
	      * @param int|string $id
	      *
	      * @return PostInterface
	      * @throws \InvalidArgumentException
	      */
	     public function find($id)
	     {
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
	
	         \Zend\Debug\Debug::dump($result);die();
	     }
	 }

刷新应用程序后你应该能看见以下输出：

	 object(Zend\Db\Adapter\Driver\Pdo\Result)#303 (8) {
	   ["statementMode":protected] => string(7) "forward"
	   ["resource":protected] => object(PDOStatement)#296 (1) {
	     ["queryString"] => string(29) "SELECT `posts`.* FROM `posts`"
	   }
	   ["options":protected] => NULL
	   ["currentComplete":protected] => bool(false)
	   ["currentData":protected] => NULL
	   ["position":protected] => int(-1)
	   ["generatedValue":protected] => string(1) "0"
	   ["rowCount":protected] => NULL
	 }

如您所见，没有任何数据被返回，取而代之的是我们得到了一些 `Result` 对象的 dump，并且里面并不包含任何有用的数据。不过这是一个错误的推论，这个 `Result` 对象只有在当你实际试图访问的时候才会拥有信息。所以若要利用 `Result` 对象内的数据，最佳方案是将 `Result` 对象传给 `ResultSet` 对象，只要查询成功就可以这样做。
	
	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	 use Zend\Db\Adapter\AdapterInterface;
	 use Zend\Db\Adapter\Driver\ResultInterface;
	 use Zend\Db\ResultSet\ResultSet;
	 use Zend\Db\Sql\Sql;
	
	 class ZendDbSqlMapper implements PostMapperInterface
	 {
	     /**
	      * @var \Zend\Db\Adapter\AdapterInterface
	      */
	     protected $dbAdapter;
	
	     /**
	      * @param AdapterInterface  $dbAdapter
	      */
	     public function __construct(AdapterInterface $dbAdapter)
	     {
	         $this->dbAdapter = $dbAdapter;
	     }
	
	     /**
	      * @param int|string $id
	      *
	      * @return PostInterface
	      * @throws \InvalidArgumentException
	      */
	     public function find($id)
	     {
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
	             $resultSet = new ResultSet();
	
	             \Zend\Debug\Debug::dump($resultSet->initialize($result));die();
	         }
	
	         die("no data");
	     }
	 }

再次刷新页面，你应该能看见 `ResultSet` 对象的 dump 现在拥有属性 `["count":protected] => int(5)`，这意味着我们有5列元组在我们的数据库中。

	 object(Zend\Db\ResultSet\ResultSet)#304 (8) {
	   ["allowedReturnTypes":protected] => array(2) {
	     [0] => string(11) "arrayobject"
	     [1] => string(5) "array"
	   }
	   ["arrayObjectPrototype":protected] => object(ArrayObject)#305 (1) {
	     ["storage":"ArrayObject":private] => array(0) {
	     }
	   }
	   ["returnType":protected] => string(11) "arrayobject"
	   ["buffer":protected] => NULL
	   ["count":protected] => int(2)
	   ["dataSource":protected] => object(Zend\Db\Adapter\Driver\Pdo\Result)#303 (8) {
	     ["statementMode":protected] => string(7) "forward"
	     ["resource":protected] => object(PDOStatement)#296 (1) {
	       ["queryString"] => string(29) "SELECT `posts`.* FROM `posts`"
	     }
	     ["options":protected] => NULL
	     ["currentComplete":protected] => bool(false)
	     ["currentData":protected] => NULL
	     ["position":protected] => int(-1)
	     ["generatedValue":protected] => string(1) "0"
	     ["rowCount":protected] => int(2)
	   }
	   ["fieldCount":protected] => int(3)
	   ["position":protected] => int(0)
	 }

另外一个非常有趣的属性是 `["returnType":protected] => string(11) "arrayobject"`。这告诉了我们所有的数据库条目都会以 `ArrayObject` 的形式返回。这产生了一个小问题，因为 `PostMapperInterface` 要求我们返回一个 `PostInterface` 对象数组，幸运的是这里有非常简单的办法让我们来解决它。在上面的例子中我们使用了默认的 `ResultSet` 对象。其实这里还有 `HydratingResultSet` 对象来将给出的数据充水成给出的对象。意思就是，如果我们告诉 `HydratingResultSet` 对象来使用数据库数据为我们生成 `post` 对象，那么它就能做到。让我们修改一下我们的代码：

	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	 use Zend\Db\Adapter\AdapterInterface;
	 use Zend\Db\Adapter\Driver\ResultInterface;
	 use Zend\Db\ResultSet\HydratingResultSet;
	 use Zend\Db\Sql\Sql;
	
	 class ZendDbSqlMapper implements PostMapperInterface
	 {
	     /**
	      * @var \Zend\Db\Adapter\AdapterInterface
	      */
	     protected $dbAdapter;
	
	     /**
	      * @param AdapterInterface  $dbAdapter
	      */
	     public function __construct(AdapterInterface $dbAdapter)
	     {
	         $this->dbAdapter = $dbAdapter;
	     }
	
	     /**
	      * @param int|string $id
	      *
	      * @return PostInterface
	      * @throws \InvalidArgumentException
	      */
	     public function find($id)
	     {
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
	             $resultSet = new HydratingResultSet(new \Zend\Stdlib\Hydrator\ClassMethods(), new \Blog\Model\Post());
	
	             return $resultSet->initialize($result);
	         }
	
	         return array();
	     }
	 }

我们又进行了几项更改。首先我们将普通的 `ResultSet` 替换成 `HydratingResultSet`。这个对象要求两个参数，第一个是使用的充水器类型（`hydrator`），第二个是充水的目标对象。充水器，简单来说，就是一个对象用来将任意类型的数据从一种格式转换成另一种。我们现在的输入类型是 `ArrayObject`，但是我们想要 `Post` 模型。而 `ClassMethods` 充水器会搞定这个转换问题，通过调用我们的 `Post` 模型的 getter 和 setter 函数。

比起去 dump `$result` 变量，我们现在直接返回初始化过的 `HydratingResultSet` 对象，从而得以访问里面存储的数据。如果我们得到了一些不是 `ResultInterface` 的实例的返回值，那么就会返回一个空数组。

刷新页面，现在你就能看见你所有的博客帖子了，很好！

## 重构隐藏依赖对象
在我们完成的事情中，还有一件事情并没有做到最佳实践。我们同时使用充水器和一个对象在下述文件内：

	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	 use Zend\Db\Adapter\AdapterInterface;
	 use Zend\Db\Adapter\Driver\ResultInterface;
	 use Zend\Db\ResultSet\HydratingResultSet;
	 use Zend\Db\Sql\Sql;
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
	 }

现在我们的映射器需要更多的参数来更新 `ZendDbSqlMapperFactory` 并且注入那些参数了。

	 <?php
	 // Filename: /module/Blog/src/Blog/Factory/ZendDbSqlMapperFactory.php
	 namespace Blog\Factory;
	
	 use Blog\Mapper\ZendDbSqlMapper;
	 use Blog\Model\Post;
	 use Zend\ServiceManager\FactoryInterface;
	 use Zend\ServiceManager\ServiceLocatorInterface;
	 use Zend\Stdlib\Hydrator\ClassMethods;
	
	 class ZendDbSqlMapperFactory implements FactoryInterface
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
	         return new ZendDbSqlMapper(
	             $serviceLocator->get('Zend\Db\Adapter\Adapter'),
	             new ClassMethods(false),
	             new Post()
	         );
	     }
	 }

当这些就绪之后，你可以再次刷新应用程序，就能看见你的博客帖子再次显示出来了。我们的映射器现在拥有一个很不错的架构，并且没有更多隐含的依赖对象了。

## 完成映射器
在我们跨越到下一个章节之前，先快速地通过为 `find()` 函数编写一个实现来完成映射器。

	 <?php
	 // Filename: /module/Blog/src/Blog/Mapper/ZendDbSqlMapper.php
	 namespace Blog\Mapper;
	
	 use Blog\Model\PostInterface;
	 use Zend\Db\Adapter\AdapterInterface;
	 use Zend\Db\Adapter\Driver\ResultInterface;
	 use Zend\Db\ResultSet\HydratingResultSet;
	 use Zend\Db\Sql\Sql;
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
	 }


这个 `find()` 函数看上去真的很像 `findAll()` 函数。这里只有三点简单的差别：首先我们需要为查询添加一个条件，让其只选择一行，这通过使用 `Sql` 对象的 `where()` 函数实现。然后我们也要检查 `$result` 变量内是否有元组在内，这通过 `getAffectedRows()` 函数实现。返回语句会被注入的冲水器充水成同样被注入的原型中。

这次，但我们找不到任何元组时，会抛出一个 `\InvalidArgumentException` 异常，以便应用程序可以方便的处理这种状况。

## 总结
完成这个章节之后，你现在了解了如何通过 `Zend\Db\Sql` 类来查询数据，也学习了关于 ZF2 新关键组件之一的 `Zend\Stdlib\Hydrator` 的知识。而且你再一次证明了你可以驾驭恰当的依赖对象注入。

在下一个章节中，我们会更进一步地了解 router，这样能让我们在模组内做更多动作。