# 使用案例和部署场景

###我该如何使用和部署Akka？

Akka 可以有几种使用方式:

* 以库的形式：作为一个普通的Jar包放进classpath 和/或在web应用中使用，放到 WEB-INF/lib中。
* 使用 [sbt-native-packager](https://github.com/sbt/sbt-native-packager) 打包。
* 使用[Typesafe ConductR](http://typesafe.com/products/conductr)打包和部署.


#####原生包
[sbt原生包](https://github.com/sbt/sbt-native-packager) 是一个工具，用来创建任何应用程序的发布, 包括一个 Akka 应用程序。

定义 sbt 版本在 `project/build.properties` 文件:

```sbt
sbt.version=0.13.7
```

添加 sbt-native-packager 到 `project/plugins.sbt` 文件:

```sbt
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.0.0-RC1")
```

使用包设置和在 `build.sbt` 文件在选择指定 `mainClass` :

```sbt
import NativePackagerHelper._
 
name := "akka-sample-main-scala"
 
version := "2.4-M2"
 
scalaVersion := "2.11.5"
 
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % "2.4-M2"
)
 
enablePlugins(JavaServerAppPackaging)
 
mainClass in Compile := Some("sample.hello.Main")
 
mappings in Universal ++= {
  // optional example illustrating how to copy additional directory
  directory("scripts") ++
  // copy configuration files to config directory
  contentOf("src/main/resources").toMap.mapValues("config/" + _)
}
 
// add 'config' directory first in the classpath of the start script,
// an alternative is to set the config file locations via CLI parameters
// when starting the application
scriptClasspath := Seq("../config/") ++ scriptClasspath.value
```


> 注意

> 使用 JavaServerAppPackaging. 不要使用 AkkaAppPackaging (以前命名的 packageArchetype.akka_application, 因为它不具有JavaServerAppPackaging那样的灵活性和质量。

使用 sbt 任务 `dist` 打包应用程序。

要启动应用程序 (在基于unix的系统):

```shell
cd target/universal/
unzip akka-sample-main-scala-2.4-M2.zip
chmod u+x akka-sample-main-scala-2.4-M2/bin/akka-sample-main-scala
akka-sample-main-scala-2.4-M2/bin/akka-sample-main-scala sample.hello.Main
```

使用 Ctrl-C 结束和退出应用程序。
在 Windows 机器上你也可以使用 bin\akka-sample-main-scala.bat 脚本。




