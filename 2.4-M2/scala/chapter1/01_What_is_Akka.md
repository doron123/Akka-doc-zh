# Akka是什么?

##### 可扩展的实时事务处理

我们相信编写出正确的、具有容错性和可扩展的并发程序太困难了。多数时候是因为我们使用了错误的工具和错误的抽象级别。Akka为此而生。使用Actor模型我们提升了抽象级别和为构建可扩展的、有弹性的响应式并发应用提供了一个更好的平台 —— 详见[《响应式宣言》](http://reactivemanifesto.org/) 。在容错性方面我们采用了“let it crash（让它崩溃）”模型，该模型已经在电信行业构建出“自愈合”的应用和永不停机的系统，取得了巨大成功。Actor还提供了透明分布式系统抽象化以及真正的可扩展和高容错应用的基础。

Akka是开源的，可以通过Apache 2许可获得。

可以从 [http://akka.io/downloads](http://akka.io/downloads)下载。

请注意所有的代码示例都是编译的，所以如果你想直接获得源代码，可以查看github上面的"Akka Docs"子项目——[java](http://github.com/akka/akka/tree/v2.4-M2/akka-docs/rst/java/code/docs)和[scala](http://github.com/akka/akka/tree/v2.4-M2/akka-docs/rst/scala/code/docs)

### Akka实现了独特的混合

##### Actors

Actors 给予你:

* 并发/并行的简单的和高级别抽象。
* 异步、非阻塞、高性能的事件驱动编程模型。
* 非常轻量的事件驱动处理（1G内存可容纳数百万个actors）。

参阅章节 [Actors (Scala)](../chapter3/01_Actors.md) 和 [Actors (Java)](#TODO)


##### 容错性

* 使用“let-it-crash”语义的监管层次结构。
* 监管层次结构可以跨越多个JVM，从而提供真正的容错系统。
* 擅于编写永不停机、自愈合的高容错系统。

参阅 [容错性 (Scala)](../chapter3/03_Fault_Tolerance.md) 和 [容错性 (Java)](#TODO)

##### 位置透明性
Akka一切都为分布式环境而设计：actor的所有交互只通过纯消息发送进行，所有的东西都是异步的。

集群支持的概览请参阅文档的[Java](#TODO)和[Scala](../chapter5/02_Cluster_Usage.md)相关章节。


##### 持久性

actor接收到的消息可以选择性的被持久化，并在actor启动或重启的时候回放。这使得actor能够恢复其状态，即使是在JVM崩溃或正在迁移到另外节点的情况下。

详情请参阅[Java](#TODO)和[Scala](../chapter3/08_Persistence.md)相关章节.

###Scala 和 Java APIs
Akka同时提供 [Scala API](../../README.md) 和 [Java API](#TODO)。

###Akka能以两种不同的方式使用

Akka能以二个不同的方式使用和部署：

* 以库的形式：作为一个普通的Jar包放进classpath 和/或在web应用中使用，放到 WEB-INF/lib中。
* 使用 [sbt-native-packager](https://github.com/sbt/sbt-native-packager) 打包。
* 使用[Typesafe ConductR](http://typesafe.com/products/conductr)打包和部署.


###商业支持
Akka由Typesafe公司提供，商业许可下提供开发和产品支持，详见[这里](http://www.typesafe.com/how/subscription)






