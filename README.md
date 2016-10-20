# kalix- 基于OSGI的快速开发平台

## 总体架构图

![Extensions Tab Screenshot](https://github.com/chenyanxu/kalix-parent/blob/master/src/image/construct.png)

> 我们的技术框架的核心是OSGI，说到 OSGi，首先要讲模块化的软件开发思想。随着科技和需求的发展变化，现在的软件正在变的越来越庞大，伴随而来的是软件的复杂度呈指数级增长，使得现代软件无论是在开发还是维护上都变的越来越困难。为了摆脱这样的困境，软件架构师们化整为零，将庞大的软件集合划分为一个个易于开发和维护的模块。模块的出现，从根本上解决了上面的问题。

> 模块化的优势体现在：
* 使资源能够合理分配：将软件模块化后，就可以分配独立的团队去处理独立的模块，从而将资源合理分配。这样既便于管理，又会降低整个软件的设计的复杂性。因为每个独立的团队可以专心去设计和实现其模块，而不用通盘考虑整个软件的复杂性。
* 增加软件重用性：每一个模块都是一个独立功能的封装，可以被轻松的用到其他的软件开发过程中，节省了资源和成本。
* 易于开发和维护：软件分成一个个模块，每一个模块都被独立的开发和维护，当软件出现问题后，就能很容易的定位到出问题的模块，降低了软件的复杂度和维护成本。

> 下面对我们使用的核心的技术进行简要介绍：

## OSGI

> OSGi 是一门非常成熟的技术，因为它已经存在16年了，OSGi 联盟负责管理并制定相关的规范。该联盟在 2009 年 9 月发布了最新版的 OSGi Service Platform V4.2 规范，在企业专家组的大力推动下，新的 Service Compendium V4.2 规范中引入了 Blueprint Services, Remote Services, Bundle Tracker 等用以支持企业级应用的新标准，从而为 OSGi Service Platform 进军企业级市场奠定了基础。

![OSGI struct](https://github.com/chenyanxu/kalix-parent/blob/master/src/image/osgi.jpg)

![OSGI class](https://github.com/chenyanxu/kalix-parent/blob/master/src/image/osgi-class.png)

> 通过OSGI类图，我们可以看到，与传统开发相比较，service消费类直接依赖于service接口类，完全不知道具体实现类的存在，这样就做到了真正的面向接口编程。service监听类负责osgi服务的监听。
  OSGI带给我们的好处：
* 模块化开发（bundle）
* 管理依赖（import和export）
* 真正的面向接口编程
* 版本化的bundle
* 动态部署

> OSGI和传统开发的比较:

![OSGI compare](https://github.com/chenyanxu/kalix-parent/blob/master/src/image/osgi-compare.png)

## Apache Ｋaraf

![karaf架构图](https://github.com/chenyanxu/kalix-parent/blob/master/src/image/karaf.png)

> Karaf是Apache旗下的一个开源项目.Karaf同时也是一个基于OSGi的运行环境,Karaf提供了一个轻量级的OSGi容器,可以用于部署各种组件,应用程序.Karaf提供了很多特性用于帮助开发者和用户更加灵活的部署应用,例如:热部署,动态配置,几种日志处理系统,本地系统集 成,可编程扩展控制台,ssh远程访问,内置安装认证机制等等.同时Karaf作为一款成熟而且优秀的OSGi运行环境以及容器已经被诸多Apache项目作为基础容器,例如:Apache Geronimo, ApacheServiceMix, Fuse ESB,由此可见Karaf在性能,功能和稳定性上都是个不错的选择。我们选择Karaf作为OSGI容器，所有的应用都运行其上。

##　Apache Shrio

![shiro架构图](https://github.com/chenyanxu/kalix-parent/blob/master/src/image/shiro.png)

> Apache Shiro（日语“堡垒（Castle）”的意思）是一个强大易用的Java安全框架，提供了认证、授权、加密和会话管理功能，可为任何应用提供安全保障 - 从命令行应用、移动应用到大型网络及企业应用。
  Shiro为解决下列问题（应用安全的四要素）提供了保护应用的API：

* 认证 - 用户身份识别，常被称为用户“登录”；
* 授权 - 访问控制；
* 密码加密 - 保护或隐藏数据防止被偷窥；
* 会话管理 - 每用户相关的时间敏感的状态。

> Shiro还支持一些辅助特性，如Web应用安全、单元测试和多线程，它们的存在强化了上面提到的四个要素。
> 我们使用Shiro最为身份验证的框架。

##　Apache OpenJPA

> OpenJPA 是 Apache 组织提供的开源项目，它实现了 EJB 3.0 中的 JPA 标准，为开发者提供功能强大、使用简单的持久化数据管理框架。OpenJPA 封装了和关系型数据库交互的操作，让开发者把注意力集中在编写业务逻辑上。OpenJPA 可以作为独立的持久层框架发挥作用，也可以轻松的与其它 Java EE 应用框架或者符合 EJB 3.0 标准的容器集成。

> 除了对 JPA 标准的支持之外，OpenJPA 还提供了非常多的特性和工具支持让企业应用开发变得更加简单，减少开发者的工作量，包括允许数据远程传输/离线处理、数据库/对象视图统一工具、使用缓存（Cache）提升企业应用效率等。

### 数据远程传输 / 离线处理
> JPA 标准规定的运行环境是 "本地" 和 "在线" 的。本地是指 JPA 应用中的 EntityManager 必须直接连接到指定的数据库，而且必须和使用它的代码在同一个 JVM 中。在线是指所有针对实体的操作必须在一个 EntityManager 范围中运行。这两个特征，加上 EntityManager 是非序列化的，无法在网络上传输，导致 JPA 应用无法适用于企业应用中的 C/S 实现模式。OpenJPA 扩展了这部分接口，支持数据的远程传输和离线处理。

### 数据库 / 对象视图统一工具
> 使用 OpenJPA 开发企业应用时，保持数据库和对象视图的一致性是非常重要的工作，OpenJPA 支持三种模式处理数据库和对象视图的一致性：正向映射（Forward Mapping）、反向映射（Reverse Mapping）、中间匹配（Meet-in-the-Middle Mapping），并且为它们提供了相应的工具支持。

### 正向映射
> 是指使用 OpenJPA 框架中提供的 org.apache.openjpa.jdbc.meta.MappingTool 工具从开发者提供的实体以及在实体中提供的对象 / 关系映射注释生成相应的数据库表。 反向映射 是指 OpenJPA 框架中提供的 org.apache.openjpa.jdbc.meta.ReverseMappingTool 工具从数据库表生成符合 JPA 标准要求的实体以及相应的对象 / 关系映射注释内容。

### 中间匹配
> 是指开发者负责创建数据库表、符合 JPA 标准的实体和相应的对象 / 关系映射注释内容，使用 OpenJPA 框架中提供的 org.apache.openjpa.jdbc.meta.MappingTool 工具校验二者的一致性。

### 使用缓存提升效率
> 性能是企业应用重点关注的内容之一，缓存是提升企业系统性能的重要手段之一。OpenJPA 针对数据持久化提供多种层次、多方面的缓存支持，包括数据、查询、汇编查询的缓存等。这些缓存的应用可以大幅度的提高企业应用的运行效率。

## Apache Camel

![camel架构图](https://github.com/chenyanxu/kalix-parent/blob/master/src/image/camel.png)

> Apache Camel是Apache基金会下的一个开源项目,它是一个基于规则路由和中介引擎，提供企业集成模式的Java对象的实现，通过应用程序接口（或称为陈述式的Java领域特定语言（DSL））来配置路由和中介的规则。领域特定语言意味着Apache Camel支持你在的集成开发工具中使用平常的，类型安全的，可自动补全的Java代码来编写路由规则，而不需要大量的XML配置文件。同时，也支持在Spring中使用XML配置定义路由和中介规则。

> Camel提供的基于规则的路由(Routing)引擎，可以使用Camel定义的DSL语言，方便的定义出各种Routing。

> 如下例：

```java
from(“file://xxxx").to("activemq://xxxx")
// 将某文件，读入并写入到ActiveMQ的JMS中。
```
> 我们使用Camel作为数据交换的框架。

