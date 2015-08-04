# 入门

###准备工作
Akka要求你机器上安装了 [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或更高版本。

###入门指南和模板项目
学习Akka的最好方法是下载[Typesafe Activator](http://www.typesafe.com/platform/getstarted)并且尝试一下其中的Akka模板项目。

###下载
下载Akka有几种方法。你可以下载Typesafe平台来下载Akka（如前所述）。你可以下载包含所有模块的完整发布包。或者也可以使用构建工具如Maven或SBT从Akka Maven仓库下载依赖。


###模块
Akka是非常模块化的，它由几个包含不同功能的Jar包组成。

* `akka-actor` – 标准Actor, 类型Actor，IO Actor等。
* `akka-agent` – Agent，与 Scala STM 集成
* `akka-camel` – Apache Camel 集成
* `akka-cluster` – 集群成员管理，弹性路由器。
* `akka-osgi` – 在OSGi容器中使用Akka的基本工具
* `akka-osgi-aries` –  配置actor系统的Aries蓝图
* `akka-remote` – 远程Actor
* `akka-slf4j` – SLF4J日志(事件总线监听器)
* `akka-testkit` – Actor系统的测试工具包

除了这些稳定的模块之外，还有一些虽然趋于稳定但仍然被标记为“实验”的模块。这并不是说他们的功能不符合预期，而主要的意思是他们的API还没有足够稳定到被认为已经固定了。你可以通过在我们的邮件组里进行反馈，来加速试验模块发布的进程。

* `akka-contrib` – 一系列的Akka贡献，他们有可能被加入核心模块中，详情见[外部贡献](../chapter8/05_External_Contributions.md)。

实际的JAR包文件名会加上版本号，如 `akka-actor_2.11-2.4-M2.jar` （对其他模块也是类似）。

查看Akka模块之间的JAR依赖的详情在[依赖](../chapter9/01_Building_Akka.md#dependencies)这一节中。


###使用发布版本
从[http://akka.io/downloads](http://akka.io/downloads)下载发布包并解压。

###使用快照版本
Akka的每日快照发布在 [http://repo.akka.io/snapshots/](http://repo.akka.io/snapshots/)， 版本号中包含 `SNAPSHOT` 和时间戳。你可以选择一个时间戳的版本，可以决定何时升级到一个新的版本。

> 警告

> 不鼓励直接使用Akka快照版（SNAPSHOT）、每日构建版（nightly）和里程碑版（milestone），除非你知道自己在做什么。


###使用构建工具

Akka可使用支持Maven仓库的构建工具。

###Maven仓库

对Akka 2.1-M2 及以后的版本：

[Maven Central](https://repo1.maven.org/maven2/)

对以前的Akka 版本：

[Akka Repo](http://repo.akka.io/releases/)

###通过Maven使用Akka

通过Maven使用Akka的最简单的入门方法，是检出[Typesafe Activator](http://www.typesafe.com/platform/getstarted)中的教程，叫[Akka Main in Java](http://www.typesafe.com/activator/template/akka-sample-main-java)。

由于Akka已经发布到Maven中心仓库了（自2.1-M2版本起），所以直接在POM文件中加入Akka依赖即可。例如。这是akka-actor的依赖：

```xml
	<dependency>
	  <groupId>com.typesafe.akka</groupId>
	  <artifactId>akka-actor_2.11</artifactId>
	  <version>2.4-M2</version>
	</dependency>
```

注意：对于快照版本，SNAPSHOT和时间戳都在版本号中。

```xml
<repositories>
  <repository>
    <id>akka-snapshots</id>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    <url>http://repo.akka.io/snapshots/</url>
  </repository>
</repositories>
```

###通过SBT使用Akka

通过SBT使用Akka的最简单的入门方法，是检出[Akka/SBT template](https://www.typesafe.com/resources/getting-started/typesafe-stack/downloading-installing.html#template-projects-for-scala-akka-and-play)项目。

通过SBT使用Akka的要点:

SBT安装指导在 [https://github.com/harrah/xsbt/wiki/Setup](https://github.com/harrah/xsbt/wiki/Setup)

`build.sbt` 文件:

```sbt
name := "My Project"

version := "1.0"

scalaVersion := "2.11.6"

libraryDependencies +=
  "com.typesafe.akka" %% "akka-actor" % "2.4-M2"
```

注意：以上的`libraryDependencies` 设置需要SBT 0.12.x或更高的版本。如果你使用更老版本的SBT，`libraryDependencies`需要这样设置：

```sbt
libraryDependencies +=
  "com.typesafe.akka" % "akka-actor_2.11" % "2.4-M2"
```

###通过Gradle使用Akka 

需要 [Gradle](https://gradle.org/) 1.4 及以上的版本来使用 [Scala 插件](http://www.gradle.org/docs/current/userguide/scala_plugin.html)

```gradle
apply plugin: 'scala'

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.scala-lang:scala-library:2.11.6'
}

tasks.withType(ScalaCompile) {
  scalaCompileOptions.useAnt = false
}

dependencies {
  compile group: 'com.typesafe.akka', name: 'akka-actor_2.11', version: '2.4-M2'
  compile group: 'org.scala-lang', name: 'scala-library', version: '2.11.6'
}
```
对于快照版本，快照仓库要添加：

```gradle
repositories {
  mavenCentral()
  maven {
    url "http://repo.akka.io/snapshots/"
  }
}
```

###通过Eclipse使用Akka
设置SBT项目然后使用 [sbteclipse](https://github.com/typesafehub/sbteclipse) 来生成Eclipse项目。

###通过IntelliJ IDEA使用Akka
设置SBT项目然后使用 [sbt-idea](https://github.com/mpeltonen/sbt-idea) 来生成IntelliJ IDEA 项目。

###通过NetBeans使用Akka
设置SBT项目然后使用 [nbsbt](https://github.com/dcaoyuan/nbsbt) 来生成NetBeans项目。

你应当也使用[nbscala](https://github.com/dcaoyuan/nbscala)来生成IDE的scala支持。

###不要使用scala编译器的 -optimize 标记

> 警告

> Akka并没有在scala编译器的`-optimize`标志下编译和测试过。尝试过这种方式的用户报告了Akka的奇怪行为。

###从源码编译

Akka使用Git并托管在 [Github](https://github.com/)。

* Akka: 从 [https://github.com/akka/akka](https://github.com/akka/akka) 克隆 Akka 仓库。

具体请参考 [构建Akka](../chapter9/01_Building_Akka.md)。

###需要帮助?

如果有问题，你可以在 [Akka Mailing List](https://groups.google.com/group/akka-user) 获得帮助。

也可以寻求 [商业支持](https://www.typesafe.com/)。

感谢你成为Akka社区的一部分。

