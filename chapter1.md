_这七个系列的文章现在完成：_

1. _微服务简介（本文）_
2. _构建微服务：使用API​​网关_
3. _构建微服务：微服务架构中的进程间通信_
4. _微服务架构中的服务发现_
5. _事件驱动的数据管理微服务_
6. _选择微服务部署策略_
7. _将重组重构为微服务_

_您还可以下载完整的文章集，以及使用NGINX Plus实现微服务的信息，作为电子书 -_[_微服务：从设计到部署_](https://www.nginx.com/resources/library/designing-deploying-microservices/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)_。_

微服务目前得到了很多关注：文章，博客，社交媒体讨论和会议演讲。他们迅速走向[Gartner炒作周期](http://www.gartner.com/technology/research/methodologies/hype-cycle.jsp)的高峰期。与此同时，软件界也有怀疑者，认为微服务没有什么新意。Naysayers声称这个想法只是SOA的一个重塑。然而，尽管有炒作和怀疑，[微服务架构模式](http://microservices.io/patterns/microservices.html)具有显着的好处 - 特别是当涉及到实现敏捷开发和交付复杂的企业应用程序。

这篇博文是关于设计，构建和部署微服务的七部分系列中的第一篇。您将了解该方法以及如何与更传统的[单片建筑模式](http://microservices.io/patterns/monolithic.html)进行比较。本系列将描述微服务架构的各种元素。您将了解Microservices Architecture模式的优点和缺点，无论它对您的项目有意义，以及如何应用它。

让我们先来看看为什么你应该考虑使用微服务。

## 建筑单片应用

让我们想象你开始建立一个全新的出租车应用程序，旨在与Uber和Hailo竞争。在一些初步会议和需求收集之后，您可以手动创建一个新项目，或者使用Rails，Spring Boot，Play或Maven附带的一个生成器。这个新的应用程序将具有模块化的[六角形架构](http://www.infoq.com/news/2014/10/exploring-hexagonal-architecture)，如下图所示：

![](https://cdn.wp.nginx.com/wp-content/uploads/2016/04/Richardson-microservices-part1-1_monolithic-architecture.png "模块化，但仍是单片，架构用作示例微服务应用的基础")

应用程序的核心是业务逻辑，它由定义服务，域对象和事件的模块实现。围绕核心的是与外部世界接口的适配器。适配器的示例包括数据库访问组件，生成和使用消息的消息传递组件以及公开API或实现UI的Web组件。

尽管具有逻辑模块化架构，但应用程序被打包并部署为一个整体。实际格式取决于应用程序的语言和框架。例如，许多Java应用程序打包为WAR文件，并部署在应用程序服务器（如Tomcat或Jetty）上。其他Java应用程序打包为自包含可执行JAR。类似地，Rails和Node.js应用程序打包为目录层次结构。

以这种风格编写的应用程序是非常常见的。它们很容易开发，因为我们的IDE和其他工具专注于构建单个应用程序。这些类型的应用程序也很容易测试。您可以通过简单地启动应用程序并使用Selenium测试UI来实现端到端测试。单片应用也很容易部署。您只需将打包的应用程序复制到服务器。您还可以通过在负载平衡器后运行多个副本来扩展应用程序。在项目的早期阶段，它工作得很好。

## 走向单片地狱

不幸的是，这种简单的方法有很大的局限性。成功的应用程序有一个随着时间增长的习惯，并最终变得巨大。在每个sprint期间，您的开发团队实现了更多的故事，这当然意味着添加许多代码行。经过几年，你的小，简单的应用程序将成长为一个[巨大的巨石](http://microservices.io/patterns/monolithic.html)。为了给出极端的例子，我最近谈到一个开发人员，他正在编写一个工具来分析其数百万行代码（LOC）应用程序中的数千个JAR之间的依赖关系。我相信它花了许多开发人员多年来一起努力创造这样的野兽。

一旦你的应用程序成为一个庞大，复杂的单片，你的开发组织可能在一个痛苦的世界。任何敏捷开发和交付的尝试都会岌岌可危。一个主要的问题是应用程序是非常复杂的。对于任何一个开发者来说，它太大了，不能完全理解。因此，修正错误和实现新功能正确变得困难和耗时。更重要的是，这往往是一个向下的螺旋。如果代码库很难理解，那么更改将不会正确。你最终会得到一个怪异，不可理解的[大泥球](http://www.laputan.org/mud/)。

应用程序的庞大规模也会减慢开发速度。应用程序越大，启动时间越长。例如，在[最近的一项调查中，](http://plainoldobjects.com/2015/05/13/monstrous-monoliths-how-bad-can-it-get/)一些开发人员报告启动时间长达12分钟。我也听说过应用程序需要长达40分钟才能启动的轶事。如果开发人员经常需要重新启动应用程序服务器，那么他们大部分的时间都会等待，他们的生产力将受到影响。

大的，复杂的单片应用的另一个问题是它是连续部署的障碍。今天，SaaS应用程序的最新技术是每天将更改推入生产多次。这对于复杂的整体来说是非常困难的，因为您必须重新部署整个应用程序以更新它的任何一个部分。我之前提到的冗长的启动时间也不会有帮助。此外，由于更改的影响通常不是很好理解，很可能您必须进行广泛的手动测试。因此，连续部署是不可能做到的。

当不同模块具有冲突的资源需求时，单片应用也可能难以扩展。例如，一个模块可能实现CPU密集型图像处理逻辑，并且将理想地部署在Amazon[EC2计算优化实例中](http://aws.amazon.com/about-aws/whats-new/2013/11/14/announcing-new-amazon-ec2-compute-optimized-instances/)。另一个模块可能是内存数据库，最适合用于[EC2内存优化实例](http://aws.amazon.com/about-aws/whats-new/2014/04/10/r3-announcing-the-next-generation-of-amazon-ec2-memory-optimized-instances/)。但是，由于这些模块一起部署，您必须妥协选择硬件。

单片应用的另一个问题是可靠性。因为所有模块都在同一进程内运行，任何模块中的错误（例如内存泄漏）都可能会导致整个进程崩溃。此外，由于应用程序的所有实例是相同的，该错误将影响整个应用程序的可用性。

最后但并非最不重要的是，单片应用程序使得采用新的框架和语言非常困难。例如，让我们想象你有两百万行代码使用XYZ框架。将整个应用程序重写以使用较新的ABC框架将是非常昂贵的（在时间和成本上），即使该框架明显更好。因此，采用新技术有巨大的障碍。你在项目开始时所做的任何技术选择都被困住了。

总而言之：你有一个成功的关键业务应用程序已经成长为一个巨大的巨型，很少，如果有的话，开发人员理解。它是使用过时的，非生产性的技术，使招聘有才华的开发人员困难。该应用程序难以扩展，并且不可靠。因此，敏捷开发和应用程序的交付是不可能的。

那么你能做什么呢？

## 微服务 - 解决复杂性

许多组织，如亚马逊，eBay和[Netflix](http://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)，通过采用现在所谓的[微服务架构模式](http://microservices.io/patterns/microservices.html)解决了这个问题。而不是构建一个怪异的，单一的应用程序，这个想法是将您的应用程序分成一组较小的，互连的服务。

服务通常实现一组不同的特征或功能，例如订单管理，客户管理等。每个微服务是具有其自己的六边形架构的小型应用，该六角形架构包括业务逻辑以及各种适配器。一些微服务会暴露其他微服务或应用程序客户端消耗的API。其他微服务可能实现Web UI。在运行时，每个实例通常是云VM或Docker容器。

例如，上述系统的可能分解如下图所示：

![](https://cdn.wp.nginx.com/wp-content/uploads/2016/04/Richardson-microservices-part1-2_microservices-architecture.png "微服务架构为一个示例的乘驾应用程序，每个微服务提出一个RESTful API")

应用程序的每个功能区域现在由其自己的微服务实现。此外，Web应用程序被分成一组更简单的Web应用程序（例如一个用于乘客和一个用于我们出租车示例中的司机）。这使得更容易为特定用户，设备或专门用例部署不同的体验。

每个后端服务公开一个REST API，大多数服务使用其他服务提供的API。例如，驱动程序管理使用通知服务器告诉可用的驱动程序有关潜在的行程。UI服务调用其他服务以便呈现网页。服务还可以使用异步的基于消息的通信。服务间通信将在本系列后面更详细地介绍。

一些REST API还暴露给驱动程序和乘客使用的移动应用程序。但是，应用程序不能直接访问后端服务。相反，通信由被称为[API网关的](http://microservices.io/patterns/apigateway.html)中介调解。API网关负责负载平衡，缓存，访问控制，API计量和监控等任务，[可以使用NGINX有效实施](http://www.nginx.com/solutions/api-gateway/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)。本系列的后续文章将涵盖[API网关](https://www.nginx.com/blog/building-microservices-using-an-api-gateway/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)。

![](https://cdn.wp.nginx.com/wp-content/uploads/2016/04/Richardson-microservices-part1-3_scale-cube.png "“缩放立方体”，在Y轴上具有功能分解成微服务")

微服务架构模式对应于[缩放立方体](http://microservices.io/articles/scalecube.html)的Y轴缩放，[缩放立方体](http://microservices.io/articles/scalecube.html)是来自优秀的可扩展性[_艺术的_](http://theartofscalability.com/)可扩展性的3D模型。其他两个缩放轴是X轴缩放，其包括在负载平衡器后运行应用程序的多个相同副本，以及Z轴缩放（或数据分区），其中请求的属性（例如，主键的行或客户的身份）用于将请求路由到特定服务器。

应用程序通常一起使用三种类型的缩放。Y轴缩放将应用程序分解为微服务，如上面本节第一幅图所示。在运行时，X轴扩展在负载均衡器后运行每个服务的多个实例，以实现吞吐量和可用性。某些应用程序还可能使用Z轴缩放来分割服务。下图显示了在Amazon EC2上运行的Docker如何部署Trip Management服务。

![](https://cdn.wp.nginx.com/wp-content/uploads/2016/04/Richardson-microservices-part1-4_dockerized-application.png "用于租用服务的示例微服务应用程序，部署在Docker容器中并由负载平衡器负责")

在运行时，Trip Management服务由多个服务实例组成。每个服务实例都是一个Docker容器。为了实现高可用性，容器在多个云VM上运行。在服务实例前面是[负载均衡器，例如](http://www.nginx.com/solutions/load-balancing/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)NGINX，通过实例分配请求。负载均衡器还可以处理其他问题，如[缓存](http://www.nginx.com/resources/admin-guide/content-caching/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)，[访问控制](http://www.nginx.com/resources/admin-guide/restricting-access/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)，[API计量](http://www.nginx.com/solutions/api-gateway/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)和[监控](http://www.nginx.com/products/live-activity-monitoring/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)。

微服务架构模式显着影响应用程序和数据库之间的关系。不是与其他服务共享单个数据库模式，每个服务都有自己的数据库模式。一方面，这种方法与企业级数据模型的想法不一致。此外，它经常导致一些数据的重复。但是，如果您希望从微服务中受益，则必须为每个服务提供数据库模式，因为它确保了松散耦合。下图显示了示例应用程序的数据库体系结构。

![](https://cdn.wp.nginx.com/wp-content/uploads/2015/05/intro-microservices.png "数据库架构在示例微服务应用程序的乘车服务")  


每个服务都有自己的数据库。此外，服务可以使用最适合其需要的类型的数据库，所谓的多语言持久性架构。例如，查找靠近潜在乘客的驱动程序的驱动程序管理必须使用支持高效地理查询的数据库。

表面上，微服务架构模式类似于SOA。使用这两种方法，该体系结构包括一组服务。然而，考虑微服务架构模式的一种方式是它是没有商业化和[Web服务规范](http://en.wikipedia.org/wiki/List_of_web_service_specifications)（WS- \*）和企业服务总线（ESB）的感知行为的SOA。基于微服务的应用程序更喜欢简单，轻量级的协议，如REST，而不是WS- \*。他们也非常避免使用ESB，而是在微服务本身中实现类似ESB的功能。微服务架构模式也拒绝SOA的其他部分，例如规范模式的概念。

## 微服务的好处

微服务架构模式有很多重要的好处。首先，它解决复杂性的问题。它将一个怪异的单片应用程序分解为一组服务。虽然功能的总量不变，但应用程序已分解为可管理的块或服务。每个服务具有以RPC或消息驱动的API的形式的明确定义的边界。微服务架构模式实施了一个模块化级别，在实践中使用单片代码库极难实现。因此，个别服务的开发速度更快，更容易理解和维护。

第二，这种架构使得每个服务能够由专注于该服务的团队独立开发。开发人员可以自由选择任何技术有意义，前提是该服务符合API合同。当然，大多数组织希望避免完全无政府状态并限制技术选择。然而，这种自由意味着开发人员不再需要使用在新项目开始时存在的可能过时的技术。当编写新服务时，他们可以选择使用当前技术。此外，由于服务相对较小，使用当前技术重写旧服务变得可行。

第三，微服务架构模式使每个微服务能够独立部署。开发人员从来不需要协调对他们的服务本地的更改的部署。这些类型的更改可以在测试完成后立即部署。UI团队可以，例如，执行A \| B测试，并快速迭代UI更改。微服务架构模式使得连续部署成为可能。

最后，微服务架构模式使每个服务能够独立扩展。您只能部署满足其容量和可用性约束的每个服务的实例数。此外，您可以使用最符合服务资源要求的硬件。例如，您可以在EC2计算优化实例上部署CPU密集型图像处理服务，并在EC2内存优化实例上部署内存数据库服务。

## 微服务的缺点

正如弗雷德布鲁克斯几乎30年前写的那样，没有银子弹。像其他技术一样，Microservices架构也有缺点。一个缺点是名称本身。微服务一词过分强调服务大小。事实上，有些开发人员主张构建极其细粒度的10-100 LOC服务。虽然小型服务是更好的，但重要的是要记住，这是一种手段，而不是主要目标。微服务的目标是充分分解应用程序，以便于敏捷应用程序的开发和部署。

微服务的另一个主要缺点是微服务应用程序是分布式系统的事实导致的复杂性。开发人员需要选择和实现基于消息传递或RPC的进程间通信机制。此外，他们还必须编写代码来处理部分失败，因为请求的目的地可能很慢或不可用。虽然这些都不是火箭科学，但它比在单片应用中复杂得多，在单片应用中模块通过语言级方法/过程调用彼此调用。

微服务的另一个挑战是分区数据库架构。更新多个业务实体的业务事务很常见。这些类型的事务在单片应用程序中实现很简单，因为存在单个数据库。但是，在基于微服务的应用程序中，您需要更新由不同服务所拥有的多个数据库。使用分布式事务通常不是一个选项，而且不仅仅是因为[CAP定理](http://en.wikipedia.org/wiki/CAP_theorem)。它们根本不受到许多当今高度可扩展的NoSQL数据库和消息传递代理的支持。您最终必须使用最终一致性方法，这对开发人员更具挑战性。

测试微服务应用程序也要复杂得多。例如，使用诸如Spring Boot的现代框架，编写用于启动单片Web应用程序并测试其REST API的测试类非常简单。相反，服务的类似测试类将需要启动该服务及其所依赖的任何服务（或至少为这些服务配置存根）。再次，这不是火箭科学，但重要的是不要低估这样做的复杂性。

微服务架构模式的另一个主要挑战是实施跨多个服务的更改。例如，假设您正在实现一个需要更改服务A，B和C的故事，其中A取决于B，B取决于C.在单片应用程序中，您可以简单地更改相应的模块，集成更改，并一次部署它们。相比之下，在微服务架构模式中，您需要仔细规划和协调对每个服务的更改的推出。例如，您需要更新服务C，然后是服务B，最后是服务A.幸运的是，大多数更改通常只影响一个服务，需要协调的多服务更改相对较少。

部署基于微服务的应用程序也要复杂得多。单个应用程序只是部署在传统负载均衡器后面的一组相同的服务器上。每个应用程序实例都配置有基础架构服务（如数据库和消息代理）的位置（主机和端口）。相反，微服务应用通常由大量服务组成。例如，根据[Adrian Cockcroft](https://twitter.com/adrianco)，[Hailo有160种不同的服务](https://sudo.hailoapp.com/services/2015/03/09/journey-into-a-microservice-world-part-3/)，Netflix有超过600[种](https://twitter.com/adrianco)。每个服务将有多个运行时实例。这是需要配置，部署，扩展和监控的更多移动部件。此外，您还需要实现一个服务发现机制（在后面的文章中讨论），使服务能够发现与其通信所需的任何其他服务的位置（主机和端口）。传统的故障单和基于手动的操作方法无法扩展到这种复杂程度。因此，成功部署微服务应用程序需要开发人员更好地控制部署方法以及高水平的自动化。

一种自动化的方法是使用现成的PaaS，如[Cloud Foundry](http://www.cloudfoundry.org/)。PaaS为开发人员提供了一种轻松部署和管理其微服务的方法。它使他们免受诸如采购和配置IT资源等问题的影响。同时，配置PaaS的系统和网络专业人员可以确保遵守最佳做法和公司策略。自动化微服务部署的另一种方法是开发基本上是您自己的PaaS。一个典型的起点是使用诸如[Kubernetes](http://kubernetes.io/)的集群解决方案，结合诸如Docker的技术。在本系列的后面，我们将看看如何[基于软件的应用程序交付](http://www.nginx.com/products/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)方法，如NGINX，轻松处理缓存，访问控制，

## 概要

构建复杂应用程序本质上很困难。单片式架构仅对简单，轻量级应用程序有意义。如果你将它用于复杂的应用程序，你将会陷入一个痛苦的世界。微服务架构模式是复杂的，不断发展的应用程序的更好的选择，尽管有缺点和实施挑战。

在后面的博客文章中，我将深入了解微服务架构模式的各个方面的细节，并讨论诸如服务发现，服务部署选项以及将整体应用程序重构为服务的策略等主题。

敬请关注…

_**编辑** - 这七个系列的文章现在完成：_

1. _微服务简介（本文）_
2. _构建微服务：使用API​​网关_
3. _构建微服务：微服务架构中的进程间通信_
4. _微服务架构中的服务发现_
5. _事件驱动的数据管理微服务_
6. _选择微服务部署策略_
7. _将重组重构为微服务_

_您还可以下载完整的文章集，以及使用NGINX Plus实现微服务的信息，作为电子书 -_[_微服务：从设计到部署_](https://www.nginx.com/resources/library/designing-deploying-microservices/?utm_source=introduction-to-microservices&utm_medium=blog&utm_campaign=Microservices)_。_

  


