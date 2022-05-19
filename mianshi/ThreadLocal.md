# ThreadLocal.md
## ThreadLocal 简介
通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。**如果想实现每一个线程都有自己的本地变量该如何解决了？**
JDK中提供了 ThreadLocal类正是为了解决这个问题。

ThreadLoca **主要是让每个线程绑定自己的值，可以将
