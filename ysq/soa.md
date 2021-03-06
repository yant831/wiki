## SOA落地浅谈（上）

团队内部谈了很久系统分拆和面向服务，但是迟迟完全落不了地，是我们出发点错误？系统边界不合理？还是大家不理解？我想用自己的思考和解释来尝试简化一下SOA的概念，帮助大家理解。

#### 目录
1. 什么是SOA
2. SOA的设计原则
3. 面向服务的框架
4. 面向服务与面向对象开发
5. 我们如何SOA起来


#### 什么是SOA
根据wikipedia的[定义](http://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%9E%B6%E6%9E%84)：面向服务的体系结构（Service-oriented architecture）是构造分布式计算的应用程序的方法。它将应用程序功能作为服务发送给最终用户或者其他服务。它采用开放标准、与软件资源进行交互并采用表示的标准方式。

企业系统的架构师认为SOA能够帮助业务迅速和高效地响应变化的市场条件. 服务导向的架构在宏观（服务）上，而不是在微观上（对象）因此提高了重复使用性。同时，服务导向的架构可以简化与传统系统的互连和使用。

在某种意义上说，服务导向的架构可以被认为是一种演化，而不是革命。

那么，什么又是“服务”？按照OASIS的定义：Service是一种按照既定“接口”来访问一个或多个软件功能的机制（另外这种访问要符合“服务描述”中策略和限制）

#### SOA的设计原则
依旧部分引自wikipedia，我只想探讨以下几点：

1. 服务封装
2. 服务可重用性
3. 服务松耦合

类似于面向对象的高内聚、低耦合原则，面向服务的架构是在更高层次对这个原则的阐述。

再次引自wikipedia：在面向对象程式设计方法中，封装（Encapsulation）是指，一种将抽象性函式接口的实现细节部份包装、隐藏起来的方法。同时，它也是一种防止外界呼叫端，去存取物件内部实作细节的手段，这个手段是由编程语言本身来提供的。这两个概念有一些不同，但通常被混合使用。封装被视为是面向对象的四项原则之一。适当的封装，可以将物件使用接口的程式实作部份隐藏起来，不让使用者看到，同时确保使用者无法任意更改物件内部的重要资料。它可以让程式码更容易理解与维护，也加强了程式码的安全性。

面向服务的架构，是在应用系统级别上，对自身提供服务能力的表述，通过接口（API）的方式开放出来，供其它系统进行调用。其它系统只需关注API的要求和输出即可，不用关注其内部细节（平台，语言，对外部系统的依赖等等）。

经过仔细考虑过的API是能够cover住一些使用场景的，即提高自身的可重用性。比如：根据用户id来返回用户发表的最新5篇文章及其内容，可以设计为：

`getLastest5Articles(userId)`

其返回结构体如下：

Array(
	0 => array('title' => 'xxx', 'content' => 'xxx'),
	1 => array('title' => 'xxx', 'content' => 'xxx'),
)

但是随着需求发展，需要另外一个接口：根据用户id来返回用户发表的最新n篇文章及内容，改进如下：

`getLastestArticlesByNumber(userId, number)；`


业务总是变化的，而且返回内容会增加返回值，占用更多的带宽，此时，有系统又需要一个根据用户id，返回n篇最新文章，但是只要标题的：

`getLastestArticleTitles(userId, number)`

于是，三个接口看起来类似，但是又稍有不同的接口产生了。当然，可能会考虑还有自定义排序，返回不同的结构体，翻页等等。有点类似于我们在面向对象中写类的接口，是提供一个“全参数”方法，还是封装一个有明确含义的方法？按照面向对象的原则，我个人倾向于提供有具体业务含义的方法，而不要提供全参数的，因为全参数的使用上不但麻烦容易出错，一般还都会暴露实现。但是，对于面向服务的架构来说，又不能一味的遵从这个原则，即我们要在两者之前取一个平衡点，参数不要超过5个为好。比如对于上述接口来说，我们可以认为返回数，翻页作为基本属性，而是否携带内容、排序（正序和倒序）作为常用的附加属性（这点要具体视实际业务使用变化频率来定），把诸如隐私，作者自己看还是第三方看等作为业务属性，用不同的方法来区分（如getArticlesByAuthor），这样使用起来更清爽，不会让使用者困扰，又不会过多的增加新的接口。

请注意，在设计对外的API时，请一定站在使用者的角度去思考问题，即使用起来是否方便、是否没有歧义，而不能站在开发者的角度。不要用你自己发明的缩写（除非是业内通用的如HTML），尽可能完整的表达清楚接口的含义，不用怕长，因为自动补全技术已经很成熟了。如queryArticles1和queryArticles2这种，别人是不会知道其实际含义是：

* queryArticles1: getArticlesByAuthor(作者本人获取文章列表);
* queryArticles2: getArticlesByBrowser(第三方获取文章列表)。

使用实际业务含义来封装接口，不要怕麻烦，其内部实现依然是调用一系列公用方法，通过变参或者策略模式等将复杂问题简单化实施的，这些都是系统内部问题，与设计接口无关，使用者更不需要也不会关心这些问题。

另外，我想重点讨论以下耦合这个问题。

SOA的耦合，可以分为几个维度：

1. 实现上的耦合
2. 时间上的耦合
3. 版本上的耦合
4. 位置上的耦合

由于SOA天然的物理隔离，使得我们从宏观上要考虑系统边界（系统划分）和接口粒度的问题（避免分布式事务）。


##### 实现上的耦合
---
和面向对象开发类似，即使接口使用者（消费者），不需要依赖服务提供者某个特殊的实现，而一切都依赖于服务提供者的约定。这样，服务提供者的内部变更就不会影响到消费者。同样，消费者也可以根据自身需要自由选择和切换该约定的其它服务提供商。比如，新浪微博提供的API和twitter的一样，其在上线初期就可以就此吸引大批为twitter开发过客户端和程序的第三方平台加入，迅速扩张自己的应用范围，方便吸引更多的开发者加盟（因为对开发者来说学习和迁移成本很低）。

同时，我们提供的接口粒度，既不能过细（一个实现要调用多次，依赖返回值作为下一个接口的参数），又不能太粗（接口设计粒度，过多琐碎的业务规则接口）。这也需要我们在设计接口时，要充分理解业务模型，并且兼顾一定的前瞻性。高内聚，三个字，不好弄。

##### 时间上的耦合
---
这是SOA中非常核心的一个环节，典型来说就是消息队列（MessageQueue）。由于SOA会大量依赖外部系统的调用，且生产者（调用端）和消费者（被调用端）在吞吐量、可用性、复杂度等，在同一时间不尽相同，所以消息队列成了解决此问题的不二选择。由于有消息中心的存在，生产者可以在不关心答复的前提下，尽可能的提高其自身的吞吐量（立即返回消息给更高层），而留给消费者去自己消化。生产者委托更多的任务给消费者，从而解放了生产者更多的资源占用（主要是等待iowait），使生产者可以承受更大的计算任务。而消费者往往由一个集群组成，通过其内部机制，可以迅速找到合适的资源，完成工作。

典型的应用像用户注册，需要发送邮件和短信激活，前台web程序不关心发送邮件和短信的返回值，只需要将发送邮件和短信的任务委托给mq，然后直接返回告知用户请等待短信和激活邮件。mq会利用自身机制来同时执行短信和邮件发送任务，同时，也会有失败重试，错误记录等等，不但简化了前台调用，又减少了用户感官上的等待时间。同理，像我们系统中的记录日志，生成审核工单，系统间的消息通知等等，总之不需要关心返回值（即不需要根据返回值来做进一步操作）的都应该使用mq来处理。


##### 版本上的耦合
---
理论上，消费者不需要依赖服务者的某个特殊版本来正常工作，及服务提供者在升级是，需要保证接口的向下兼容性问题。当然，这是理想情况，一般来说，对于内部SOA，可以保证在一定版本范围内实现兼容，同时发布升级通知，让所有消费者在指定时间内完成接口的迁移，之后老接口就可以废弃了。而公用SOA则因为消费者不可控而麻烦很多。这样也就需要我们在设计接口的时候，尽可能保证接口的单一性（影响范围小）。版本上的松耦合当然会对我们的编程提出更高的要求和更多的维护工作，这也会强迫我们在接口设计时多站在使用者的角度去考虑问题，从而尽可能减少自己的维护工作。


##### 位置上的耦合
---
这个概念也是伴随着SOA一并到来的，简单来说，就是服务在哪？

1. 我们可以使用域名调用的方式，来简化这个问题，服务提供者在其内部，基于域名可以做负载均衡或者其他高可用的实施，以保障对外服务的可用性。
2. 另外一种流行的方式，即通过服务注册中心来发现和使用服务。服务提供者可以在服务注册中心来维护自身的可用性，自动加入或者下线。消费者通过服务注册中心来感知，然后根据策略选出一个服务者进行调用。这样，消费者就不需要自身感知服务提供者的域名，ip等固化信息，从在位置上进一步解耦了。比如现在最常用的[zookeeper](http://zh.wikipedia.org/wiki/Apache_ZooKeeper)。

我们当前使用既不是1也不是2，而是一种更原始的方式：消费者本身感知服务者ip，通过随机的策略选择出来一个进行调用。服务者的ip是写死在消费者代码中的。然后服务提供者再通过keepalived的机制，尽可能保障自身服务的可用性，来规避可能的问题。这种机制在服务相对比较稳定，且调用在可控范围内时是可行的，但是对服务提供者来说，自身的可扩展余地比较有限，如果消费者激增或者自身需要下线维护，则需要逐一修改消费者代码，并且重新发布。可以稍微调整一下来改变，及向消费者提供虚IP，然后利用lvs做物理隐藏，来规避这个问题。


#### 面向服务的框架

根据wikipedia的定义，SOA的架可以看做是各自独立的服务，通过契约相互委托，成网状相互调用。SOA通过远程调用（RPC），将依托于各种语言的模块系统化，从而在更高层面实现功能上的单一和高度可重用性。类似于面向对象的抽象，封装，高内聚、低耦合原则，SOA也因此被定义成“能够帮助业务迅速和高效地响应变化的市场条件”。面向服务和面向对象很相似，但是又有很多不同。

由于是远程调用，面向服务中的调用开销高于对象间调用的几个数量级，所以，优化远程调用是所有SOA框架必须要解决的问题。一般需要关注系统响应速度，传输数据大小，吞吐量，并发数等等。同时，也需要根据具体的业务场景，物理结构，部署结构，语言，甚至是分布式事务等等，来做出权衡。

如内网调用使用千兆甚至万兆网络，而公网调用不但带宽会少很多，而且还需要考虑安全认证等问题。

我参考了一些目前较常见的用法，一般建议如下：

* 内网多使用TCP+二进制序列化；
* 公网多使用HTTP+文本序列化（如webservice）；

目前的开源分布式框架来说，主要还是在java的世界，如Spring Remoting，[Dubbo](https://github.com/alibaba/dubbo)，[Finagle](https://github.com/twitter/finagle)，[zookeeper](http://zh.wikipedia.org/wiki/Apache_ZooKeeper)。而php则没有类似开源成熟的技术，目前还只能是各路神仙各显神通的解决，靠点边的如：[thrift](http://thrift.apache.org/)其主要还是为了解决不同语言间的相互调用问题。我们目前使用的是自己实现的HTTP+igbinary的方式，其它还有类似于HTTP+json的组合，但是考虑到php序列化的方便性，以及受限于目前GBK的编码方式，压缩传输比等综合因素，最终还是选定了[igbinary](https://github.com/igbinary/igbinary/)。

简单提一句thrift。

它最初是来自facebook的跨语言的service调用框架，支持Java，C++，Python，PHP，Node.js等几乎所有我们知道的主流语言，其采用了配置生成相关代码的方式来解决多语言平台的粘合性问题。看到《inside facebook》的话就会了解。早期facebook也是全php的，但是随着其业务的突飞猛进，内部团队的发展，各个语言平台也有其适应场景，再加上facebook是工程师范儿的文化，不可能再要求所有人都是用1-2种语言了，在美国，java和php几乎占据了绝大部分，而随着python和ruby的快速崛起，越来越多年轻的程序员选择了它们。所以thrift由其工程团队推出就是与时俱进的结果了。有兴趣的人，建议看下[《inside facebook》](http://book.douban.com/subject/2287687/)以及thrift。

php方面，去年开始，我关注到了一个新的试图解决php分布式调用问题的框架：[swoole](http://www.swoole.com/)。其号称：PHP语言的高性能网络通信框架，提供了PHP语言的异步多线程服务器，异步TCP/UDP网络客户端，异步MySQL，数据库连接池，AsyncTask，消息队列，毫秒定时器，异步文件读写，异步DNS查询。Swoole底层内置了异步非阻塞、多线程的网络IO服务器。PHP程序员仅需处理事件回调即可，无需关心底层。与Nginx/Tornado/Node.js等全异步的框架不同，Swoole既支持全异步，也支持同步。这也是我个人比较看好的一个项目，目前已经发展到1.7.3版本，会在今后的框架改进中进行尝试。有兴趣的人也可以提前了解一下其原理。

分布式事务问题，我个人也不知道如何解决。除了目前的2pc、3pc和paxos外，似乎尽可能规避分布式事务依然是最好的解决方式，有关CAP理论，推荐阅读：

* [CAP理论十二年回顾："规则"变了](http://www.cnblogs.com/shanyou/archive/2012/06/11/2545536.html);
* [可伸缩性原则](http://www.infoq.com/cn/articles/scalability-principles);

---

本来想一次性写完本篇，之前也构思了很久，真正写起来还是发现很难很简单的一次性讲明白，结果导致越写越多，然后写了又改，删了大量代码相关的东西。而且，本月的截稿日期也到了（也许这才是主要原因 :-O）。本篇还是注重概念性的东西多一些吧，下一篇会讲：

* 从前台到后台完全的跨语言分离；
* 如何结合当前我们的开发方式进行逻辑划分等；

如果大家有问题，也欢迎反馈给我。





