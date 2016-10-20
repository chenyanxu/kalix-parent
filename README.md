# kalix- 基于OSGI的快速开发平台

## 总体架构图

![Extensions Tab Screenshot](https://github.com/chenyanxu/kalix-parent/blob/master/construct.png)

> 我们的技术框架的核心是OSGI，说到 OSGi，首先要讲模块化的软件开发思想。随着科技和需求的发展变化，现在的软件正在变的越来越庞大，伴随而来的是软件的复杂度呈指数级增长，使得现代软件无论是在开发还是维护上都变的越来越困难。为了摆脱这样的困境，软件架构师们化整为零，将庞大的软件集合划分为一个个易于开发和维护的模块。模块的出现，从根本上解决了上面的问题。

> 模块化的优势体现在：
* 使资源能够合理分配：将软件模块化后，就可以分配独立的团队去处理独立的模块，从而将资源合理分配。这样既便于管理，又会降低整个软件的设计的复杂性。因为每个独立的团队可以专心去设计和实现其模块，而不用通盘考虑整个软件的复杂性。
* 增加软件重用性：每一个模块都是一个独立功能的封装，可以被轻松的用到其他的软件开发过程中，节省了资源和成本。
* 易于开发和维护：软件分成一个个模块，每一个模块都被独立的开发和维护，当软件出现问题后，就能很容易的定位到出问题的模块，降低了软件的复杂度和维护成本。

> 下面对我们使用的核心的技术进行简要介绍：

## OSGI

> OSGi 是一门非常成熟的技术，因为它已经存在16年了，OSGi 联盟负责管理并制定相关的规范。该联盟在 2009 年 9 月发布了最新版的 OSGi Service Platform V4.2 规范，在企业专家组的大力推动下，新的 Service Compendium V4.2 规范中引入了 Blueprint Services, Remote Services, Bundle Tracker 等用以支持企业级应用的新标准，从而为 OSGi Service Platform 进军企业级市场奠定了基础。

![OSGI struct](https://github.com/chenyanxu/kalix-parent/blob/master/osgi.jpg)