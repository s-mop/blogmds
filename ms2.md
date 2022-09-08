---
title: 回顾《Microservices》系列之二：围绕“业务能力”进行组织
date: 2021-11-12 16:53:00
---
> 本文是笔者作为企业架构咨询顾问，以《Microservices》文章作为主轴，在不同客户企业经历各式各样服务化项目后的所感所想 
# 按技术划分团队的弊端

> ...often management focuses on the **technology layer, leading to UI teams, server-side logic teams, and database teams**.
> 
> ...simple changes can **lead to a cross-team** project taking time and budgetary approval. 
> 
> ...force the **logic into whichever application** they have access to. **Logic everywhere** in other words. 

原文首先表述了传统按技术划分团队（UI、业务逻辑、数据库）的两个结果：

-   跨团队协作带来的审批和时间投入
-   把逻辑做进离组织更近的系统应用中

其中第二个结果是为了避免第一个结果的选择，但这很可能是默默埋下的技术债

现实工作中，往往更接近用户一侧的团队面对更多的需求细节和变更，比如UI团队，而按照上述第二种选择的思路就会引起大量业务逻辑停留在前端界面中。这明显是会对后续维护和迭代造成负面影响

# 全功能团队的陷阱

> The microservice approach to division is different, splitting up into services **organized around business capability**.
> 
> ...the teams are cross-functional, including  user-experience, database, and project management.

 第二段文字提到微服务的处理方式是按照业务能力划分团队，并且是包含了用户界面、数据库、系统集成和项目管理的跨功能团队。

在这其中，数据库与微服务团队的自治关系在不少实践中已经逐渐清晰与完善。系统集成在自动化API文档、自动化测试脚本等工具也逐渐成熟

但在我看到的团队里，用户界面和项目管理这两点还很难做到“跨功能团队”级别的效果，其中：

-   关于用户体验层面，微前端一定程度上可以帮忙解决部分问题。但同时，对于用户体验层面，业务常常无法划清界限，且很可能有依赖关系。在我看来，即使是团队由非常有经验的工程师组成，要解决这个问题还是有很多探索性的工作要尝试。特别是对用户体验工程师和产品，需要有更高的抽象概念及领域边界识别能力要求
-   关于项目管理。我认为这可能是影响微服务是否能产生最高价值的关键。从产品出发的每次迭代，往往涉及多个业务领域，而一个微服务团队往往无法专注于单一的产品迭代，因为需求可能来自不同的客户端、渠道、平台等。微服务团队与产品迭代之间的资源冲突是否能高效解决，不单单是微服务团队、产品、PO某一单一角色甚至团队决定的。而这个复杂的组织关系又往往是大型企业和项目需要面对、解决的问题。其中一个办法就是投入更多的人去做协调、对齐等知识传达工作。那么是否有一种方法能解决这个痛点，目前看来是需要一个能给微服务项目管理和产品管理之间找到平衡点的工具或方法。并且这个工具和方法在我看来很难像XP、Scrum、甚至大规模敏捷那样普遍适用，而更多的是需要对团队协作有资深经验的人员在对企业有足够深入的了解之后才能给出对应方案。

# 关于颗粒度
> ...main issue we have seen here, is that they tend to be organised around too many contexts

第三段文字中有这样一句话，这个是针对微服务及团队应该划分到多小，文中也提到不应该让一个微服务由太多的上下文组织而成。

这种担心原因很明显，防止过多本不相干的上下文缠绕在一起，模糊了边界。

但这里我想说的是在团队有了一定服务化、业务领域识别的的基础上，可以不用强制团队或者服务按照边界一比一划分。

在原文中一个6人团队管理6个微服务的例子中，就是团队与服务1：N的关系。这种情况有利于团队文化、能力的构建与传播。

另外，真的一定要把边界清晰的不同上下文拆分成不同服务吗？我认为不是必须。只要我们可以保证不同上下文之间没有非必须的交互、控制好边界，也可以通过目录、代码包、module等形式来帮助边界的清晰。只要团队对于清晰边界的识别能力在、并且恪守了必要原则，那么在需要的时候、借助系列第一章中“Explicit Published Interface”相关方法拆分成单独服务其实是手到擒来的事情，毕竟进程间通信和进程内通信的成本有显著差异

《Microservices》原文：  
[https://martinfowler.com/articles/microservices.html](https://martinfowler.com/articles/microservices.html)
