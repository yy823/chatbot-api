# Guava的缓存

若你已经加入“技术派”的语雀文档，你可以在基础篇中看到“技术派Guava整合本地缓存”的实现逻辑。这里就不具体描述如何实现。

**补充信息**：

-   技术派中，只使用了Guava的LoadingCache类。你可以把LoadingCache看成一个完全的HashMap，只是在get()时缓存了结果，假如没有则执行对应的load()方法得到结果。
-   和Caffeine比较，Caffeine更加优秀，你可以在这篇[知乎文章](https://zhuanlan.zhihu.com/p/345175951)查看具体的信息。

**吐槽一下**：个人不建议将Guava称为一种缓存，而是一种工具，内部包含了缓存。在初始技术派时，我以为常常需要Guava和Caffeine二选一来用。







