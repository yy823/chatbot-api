# Caffeine的缓存

若你已经加入“技术派”的语雀文档，你可以在基础篇中看到“技术派Caffeine整合本地缓存”的实现逻辑。这里就不具体描述如何实现。

你可以在SidebarServiceImpl类中看到更详细的实现。

**补充信息**：

-   你可以在这篇[博客](https://blog.csdn.net/youbl/article/details/113052502)看到更多Spring Cache的扩展使用。
-   Caffeine是存储的实例指针，对应的缓存内容仍然存储在Java进程内部中，所以称之为“本地缓存”。需要和Redis这种“序列化缓存”区分开来。你可以在“技术派Caffeine整合本地缓存踩坑实录”中看到它们更加详细的区别。

**吐槽一下**：Spring Cache也太垃圾了吧……



















