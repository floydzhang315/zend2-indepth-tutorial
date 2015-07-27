# 了解 Router
现在我们的模组有了一个非常坚固的基础。然而，我们并没有做十分多的事情，准确来说，我们做的所有事情仅仅是在一个页面上显示所有 `Blog` 条目而已。在这个章节，你将会学习关于 `Router` 所有你所需要知道的事情，来创建其他路径来显示其中一个博客帖子，添加一个新的博客帖子，和编辑或者删除现有的博客帖子。

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

