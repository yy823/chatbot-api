### 1 前言

相对于原本的1.0版本（即Spring Boot + MySQL版本），此版本添加了Redis + Caffeine + Guava来强化整个项目。其中Guava在1.0版本也遇到过，只是简单地提了一下。这三个都可以用于简单的缓存设计，下面稍微介绍一下，能够让各位对它们有一个基本的印象。**其中Redis是学习的重中之重，也是必学的内容。**

### 2 Redis

Redis是一个由C语言编写的、基于内存、单线程执行命令的中间件，是一个独立的程序。和原本的Java程序需要通过Socket连接通信（进程通信）。下面是它的特点：

-   因为基于内存，所以很快，可以用作缓存功能。
-   因为其执行命令是单线程的，所以能够实现并发计数的功能。
-   其内部有优质的数据结构，能够实现很多特别的功能，例如排行榜、抽奖系统。
-   因为是单独的进程且使用Socket连接，所以可以额外使用一个物理机运行Redis，再和Java进程连接，从而实现某种意义上的分布式，即不会和原本的Java程序竞争计算机资源。
-   拥有集群、主从复制的功能，能够很好的横向扩展，例如可以在两个物理机中分别运行Redis，并且能够同时服务于一个Java进程，其中一个用于读，另一个用于写，同时，这两个Redis进程也能够自动进行数据同步。

在技术派中，Redis使用场景：

-   缓存设计：Redis缓存
    -   ForumCoreAutoConfig：缓存相关的设置类
    -   SitemapServiceImpl：站点地图：站点的所有url内容、PV/UV计数统计
    -   TagSettingServiceImpl：标签相关的类
    -   UserStatisticEventListener：统计用户的粉丝、收藏、关注信息
    -   CountServiceImpl：统计文章的点赞、阅读、评论数量
-   排行榜：Redis的Zset数据结构，自动根据分数排名，快速获取前n名
    -   UserActivityRankServiceImpl：活跃度排行榜的实现类
-   白名单：Redis的Set数据结构，快速查看是否存在
    -   AuthorWhiteListServiceImpl：白名单管理的实现类
-   JWT的升级：Redis存储token，服务端掌握token的主动权
    -   UserSessionHelper：token相关的类

额外的，Redis的著名功能，但没有使用的：

-   分布式锁
-   延迟队列

>   **能够在Java内部维护一个类来实现排行榜、白名单这些功能，但为什么使用Redis？**
>
>   -   Redis本身应该算是一种基于内存的数据库，这些功能的实现可以看作是一种缓存。
>   -   Redis是线程安全的，默认与原进程异步的。如果使用Java内部维护则会涉及到更多的并发操作和线程切换。
>   -   Redis能够实现类分布式，也就是一个Redis服务器可以服务多个Java进程，方便分布式的扩展。
>   -   Redis本身由C语言编写，并且其优化的数据结构使得实现这些更加高效。
>   -   Redis有持久化功能，能够保证即使宕机后也能够实现数据的恢复，假如使用Java内部维护，则需要控制更多的内容。

### 3 Caffeine

Caffeine是一个由Java编写的缓存依赖包。内部有优质的缓存实现，会将数据存储在Java进程中。

在技术派中，Caffeine的使用场景：

-   SidebarServiceImpl：侧边栏实现类，配合@Cacheable注解实现Spring自动缓存

### 4 Guava

Guava一个由Java编写的、Google提供的工具依赖包。内部有优质的数据机构和API，包括简化某些缓存操作。

在技术派中，Guava的使用场景：

-   HttpRequestHelper：缓存请求模板
-   CategoryServiceImpl：缓存侧边栏分类信息
-   ImageServiceImpl：缓存已经转链后的图片的url
