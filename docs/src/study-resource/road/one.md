---
title: 前言
shortTitle: 前言
date: 2024-11-12
---


## 编程语言学习路线

::: tip

- [C/C++ 语言学习路线](./02.C语言学习路线.md)
- [Python 学习路线](./01.Python学习路线.md)
- [Java 学习路线](./01.Java推荐路线.md)
- [Go 学习路线](./01.Golang学习路线.md)
- [前端学习路线](./01.前端推荐路线.md)


:::

##  引言

现在大家最缺的不是学习资料，而是详实的学习路线规划、方向分析，没有正确的学习路线，有力也使不出。

本文给大家总结了从 0 基础开始学习计算机的完整路线，包括：基础入门、核心课程学习、编程实践、进阶学习、职业规划等等方面。

基础入门：推荐比占的《计算机科学速成课》视频课程。作者以非常多的图示，非常生动的给大家介绍了计算机的发展历史以及体系结构概念，非常易懂。

核心课程：操作系统/组成原理/数据结构/数据库/网络。推荐了优秀的学习视频课程和书籍资料，以及方法论经验。

方向选择：给大家介绍了目前就业主流的方向，以及个方向的优劣，如何选择。

进阶学习：如何就一个方向深入学习，拓展自己的深度。

实战练兵：计算机是一门实践性很强的学科，因此必须加强实践，提高自己的编程内功。给大家推荐了一些实战的方向：项目、比赛、等等。

## 整体思路

如何针对有限的时间进行合理规划，我看很多文章上来就是给一堆书籍、一堆视频课程，我认为最重要的是捋清整体思路。也就是方法论，把握方向，然后再开始按部就班的开始自己的学习历程。

**一开始就陷入海量课程和书籍，容易迷失、失去信心**。

**1、首先是打牢基础。**

打牢基础！打牢基础！重要的事情说三遍！

基础非常重要，不管你以后从事什么方向，应用开发也好，大数据开发也要，客户端工程师也好，要想吃得开必须依赖这些基础课程：**操作系统、组成原理、计算机网络、数据结构、算法、数据库**，再加上学习一门语言。

<b>这里要说明的是，很多大学计算机专业会开设诸如数学、电子电路、物理等等相关的课程，如果大家目的是找工作，那这些课程可以不用学习，把重点都放在我刚说的核心课程上。</b>

**2、明确自己的发展方向。**

打牢基础后，可以**开始选择自己后面的发展方向了**。

> 以后从事哪个方向，是硬件、嵌入式还是软件？
>
> 若是软件细分的话是前端？后端？客户端？或算法？

**3、进阶学习。**

在基础入门后，就可以开始进阶学习了。

- **学习优秀的开源框架**。计算机专业的好处是非常多的优秀资料都是开源的，我们只需一台电脑编可以免费获得。从优秀的框架中，学习其设计思路，代码风格，是提升自身编码水平的非常好的方式。

- **数据结构和算法**。平常要多刷题，推荐好的刷题平台 LeetCode。

**4、实战非常重要。**

第二部分说到要多实战，那么大学大概有哪几种实战类型呢？

**一是各种比赛**，有含金量的比赛大概有 ACM、天池、kaggle、阿里中间件性能挑战赛等等，这里不全部列出来，下次我专门出个文章来跟大家讲清楚。

**另外是做项目**，跟着老师做项目是首选。因为有人带着，可以跟着老师和学长学到很多东西。

如果没有这样的机会，自己**参与 github 开源社区**也是非常不错的，社区有对应的邮件组和群聊，有非常热心的小伙伴。

## 计算机编程基础

2024 编程语言榜单

<img src="https://cdn.golangcode.cn/images/202502041509110.png" alt="alt text" style="zoom:50%;" />

### （一）学习一门语言

**初学建议选 C 语言，为什么呢？**

因为 C 语言是一门偏底层的语言，能够让你了解到程序的底层机制，而且很多高校的课程也是 C 语言，很多比赛如 ACM 也是推荐 C/C++语言。**C 语言学好后再学其他语言则是比较容易的事情了**。

学了 C 语言后一般还会学习一门其他语言，特别是面向对象语言，常见的有 C++、Java。另外还有 python，用来做数据处理非常方便。

**C 语言经典的书籍这里推荐 3 本：**

- 第一本是<b>《C Primer Plus》</b>，**比较适合入门**。内容循序渐进，书中的每一个知识点都有很多生动简单的示例，并给出了相应的运行结果。而且每章末设计了大量复习题和编程练习，帮助巩固所学知识和提高实际编程能力。
- 第二本是<b>《C 程序设计语言》</b>，豆瓣评分 9.4 分，**适合有一点基础后再来看**。
- **进阶推荐《C 和指针》**。全书共 18 章，覆盖了数据、语句、操作符和表达式、指针、函数、数组、字符串、结构和联合等几乎所有重要的 C 编程话题。书中给出了很多编程技巧和提示，每章后面有针对性很强的练习。

**书籍配合着视频课程一起学**，效果会更好。如果只是抱着书啃，会比较枯燥。

国内浙大**翁凯老师**的课，看过的都说好~：

浙大 C 语言-翁凯，分为两门：

> C 语言程序设计 CAP（大学先修课）：https://www.icourse163.org/course/ZJU-1001614008
>
> C 语言程序设计进阶：https://www.icourse163.org/course/ZJU-200001

**关于编辑器/IDE，推荐如下几个适合初学者的**：
- Visual Studio（Windows 平台）
- Dev C++
- Code::Blocks

### （二）计算机基础内容有哪些？

计算机核心基础内容可以总结为 6 个部分：
- 1、操作系统（Linux）
- 2、计算机组成原理/系统结构
- 3、计算机网络
- 4、数据结构
- 5、算法
- 6、编程语言（推荐 C++/Java）


### （三）如何进行基础内容的学习

推荐**视频课程+书籍结合**的方式，千万不要抱着大块头的书从头啃到尾。看视频课程会比纯看书更生动，更容易理解。

课程平台推荐：

- 中国大学 MOCC
- 网易公开课

## **方向选择以及进阶学习**

**怎么选？选什么？**

计算机专业毕业后可以从事的方向很多，对于在校学生来讲可能非常迷茫。那么怎么确定自己以后的方向呢？可以按照这个思路：

- 观察行业需求
- 待遇情况
- 结合自己的个人兴趣
- 要从事对应岗位需要满足什么要求，需要做哪些准备

**1、收集信息**

**（1）和学长学姐交流，这是最直接最有效的方式。**

对于已经工作的学长学姐，经历了校园招聘以及职场的体验，亲身经历后感受会更深，视野也会更开阔。多和他们沟通，了解常见的方向有哪些，各有什么利弊，可以提前做哪些准备？

**（2）提前关注以下这些网站：**

- 招聘网站：BOSS 直聘、拉勾，观察各个公司的招聘岗位和要求以及大致待遇；
- 高效 BBS：观察各个公司的招聘启事，以及学长学姐的讨论和经验总结；

**2、了解对应行业的要求以及发展前景**

确定自己的目标，**找到对应技术栈的学习路径**，然后学起来吧。

## **实战练兵**

对于计算机专业来讲，**实践是非常非常重要的**。

那么有哪些实战的方式呢？

![alt text](https://cdn.golangcode.cn/images/202502041510418.png)

**1、参加比赛**

有含金量的比赛大概有如下几个：

- 经典算法比赛：ACM-ICPC 全球竞赛、topcoder
- 数据挖掘/AI 比赛：Kaggle 比赛、天池比赛、KDD-CUP、腾讯广告算法大赛
- 中间件-阿里巴巴中间件性能挑战赛

**2、实习**

实习是非常好的实践方式，一方面可以**提前了解和感受互联网公司的环境、氛围**，另一方面**向各路大牛学习**，获取一手的经验，还可以参与实际的项目。

**3、github 开源项目**

如果没有实习，也可以通过参与实验室项目或者 github 上的开源项目。github 上开源社区有完善的文档、协作机制，可以和很多人一起交流，开阔视野。

推荐的值得参与的 github 开源项目：

- https://github.com/macrozheng/mall
- https://github.com/qiurunze123/miaosha

**4、刷题**

主要是锻炼数据结构和算法能力、提高思维能力。推荐用 LeetCode 或牛客。

- https://github.com/CyC2018/CS-Notes/blob/master/notes/Leetcode%20%E9%A2%98%E8%A7%A3%20-%20%E7%9B%AE%E5%BD%95.md
- https://github.com/azl397985856/leetcodehttps://leetcode-solution-leetcode-pp.gitbook.io/leetcode-solution/
- https://github.com/youngyangyang04/leetcode-master

**5、顶尖公开课的 project**

通常都非常有深度，有一步一步实现一个麻雀虽小五脏俱全能 run 起来的系统，如数据库或操作系统，或者一个 tcp 协议栈。

---

## **其他 tips**

**（1）自学能力的培养。**

现在已经是互联网时代了，我们能从网络上很方便的获取大量、免费的资料，如公开课、电子书、github 上的开源代码。我们需要有从大量信息中筛选有价值信息的能力，人的精力是有限的，我们不可能面面俱到，学习一定要学经典资料。二是提升自己的自学能力，掌握学习新知识的方法论。

**（2）养成写博客的习惯。**

写博客对我们好处很多：

- 如果能讲出来让别人懂，那说明自己是真的懂了。勤总结可以不断加深我们对知识的理解；
- 博客也是自己的一个标签，能够让自己在以后的面试中获得加分；

**（3）多和高年级的师兄师姐沟通**

可以少走很多弯路。

> 注：本文来自https://github.com/summerjava/awosome-cs。