# 概述

* [术语，概念](01_Terminology_Concepts.md)
  * 并发 vs. 并行
  * 异步 vs. 同步
  * 非阻塞 vs. 阻塞
  * 死锁 vs. 饥饿 vs. 活锁
  * 竞态条件
  * 非阻塞担保（进展条件）
  * 推荐文献
* [Actor系统](02_Actor_Systems.md)
  * 层次结构
  * 配置容器
  * Actor最佳实践
  * 阻塞需要小心管理
  * 你不应该担心的事
* [什么是Actor?](03_What_is_an_Actor.md)
  * Actor引用
  * 状态
  * 行为
  * 邮箱
  * 孩子
  * 监管策略
  * 当Actor终止时
* [监管与监控](04_Supervision_and_Monitoring.md)
  * 监管的意思
  * 顶级监管者
  * 重启的含义
  * 生命周期监控的含义
  * 一对一策略 vs. 多对一策略
* [Actor引用, 路径与地址](05_Actor_References_Paths_and_Addresses.md)
  * 什么是Actor引用？
  * 什么是Actor路径?
  * 如何获得Actor引用？
  * Actor引用和路径相等性
  * 重用Actor路径
  * 与远程部署之间的互操作
  * 路径中的地址部分用来做什么？
  * Actor路径的顶层作用域
* [位置透明性](06_Location_Transparency.md)
  * 默认分布式
  * 破坏透明性的方式
  * 远程调用如何使用？
  * Peer-to-Peer vs. Client-Server
  * 使用路由来进行垂直扩展的标记点
* [Akka与Java内存模型](07_Akka_and_the_Java_Memory_Model.md)
  * Java内存模型
  * Actors与Java内存模型
  * Future与Java内存模型
  * STM与Java内存模型
  * Actor与共享的可变状态
* [消息传递可靠性](08_Message_Delivery_Reliability.md)
  * 一般规则
  * JVM内（本地）消息发送规则
  * 更高层次的抽象
  * 死信
* [配置](09_Configuration.md)
  * 配置读取的地方
  * 当使用JarJar,，OneJar，Assembly或任何jar打包命令（jar-bundler）
  * 自定义application.conf
  * 包含文件
  * 配置日志
  * 讲讲“类加载器”
  * 应用特定设置
  * 配置多个ActorSystem
  * 从自定义位置读取配置
  * Actor部署配置
  * 参考配置清单 
