#Akka 类型

> 警告

> 这个是目前正在积极研究的主题的实验模块. API或语义在更改时，不会有警告或者废弃的提示，建议不要使用这个模块到生产环境！.

前面关于讲到的[Actor 系统](../chapter2/02_Actor_Systems.md) (和下面的章节) Actors是在独立计算的单位之前发送消息, 但看起来如何? 假如我们导入下面这些:

```scala
import akka.typed._
import akka.typed.ScalaDSL._
import akka.typed.AskPattern._
import scala.concurrent.Future
import scala.concurrent.duration._
import scala.concurrent.Await
```
有了这些我们可以定义我们的首个Actor, 当然也它也可以 say hello!

```scala
object HelloWorld {
  final case class Greet(whom: String, replyTo: ActorRef[Greeted])
  final case class Greeted(whom: String)
 
  val greeter = Static[Greet] { msg =>
    println(s"Hello ${msg.whom}!")
    msg.replyTo ! Greeted(msg.whom)
  }
}
```

这一小块代码定义了两种消息类型, 一个是命令Actor 给某人打招呼和一个让 Actor 确认已完成. ``Greet`` 类型不仅仅包含要打招呼的信息 whom, 还持有一个``ActorRef`` 引用，提供了消息的发送者，这样``HelloWorld`` Actor 能回复一个回执信息.

在``Static``行为构造函数的帮助下，Actor的行为被定义为值``greeter`` --正如我们在下面将看到的，有很多不同的方式制定行为。 “static” 行为在响应一个消息时不能改变, 在Actor被其父亲终止之前它一直不会改变。

通过行为处理的消息类型是声明的``Greet``类, 函数中的隐式提供的``msg`` 参数就是属于这个类。这是我们能访问``whom`` 和 ``replyTo`` 成员的一种方式，无须使用模式匹配。

在最后一行我们看到``HelloWorld`` Actor 发送一个消息到另一个Actor, 使用 ! 操作符 (很明显的 “tell”方式). 因为replyTo 地址被声明为``ActorRef[Greeted]`` 类型，编译器仅允许我们发送这个类型的消息, 其它方式都不接受。

Actor一起接受的消息类型和所有回复类型通过这个Actor所说的协议来定义; 在这种情况下它是一个简单的“请求-回复”协议，但Actors当需要时，可以模拟任意复杂的协议. 协议和行为捆绑在一起，实现在一个精巧的包装作用域``HelloWorld`` 对象。

现在我们想要考验这个 Actor, 所以我们得启动一个 Actor系统来试试它:

```scala
import HelloWorld._
// using global pool since we want to run tasks after system shutdown
import scala.concurrent.ExecutionContext.Implicits.global
 
val system: ActorSystem[Greet] = ActorSystem("hello", Props(greeter))
 
val future: Future[Greeted] = system ? (Greet("world", _))
 
for {
  greeting <- future.recover { case ex => ex.getMessage }
  done <- { println(s"result: $greeting"); system.terminate() }
} println("system terminated")
```

在导入Actor的协议定义之后，我们从定义的行为启动一个 Actor系统, 包装它在``Props``中，就像actor就在前台一样. ``props`` 只是我们给定的默认值， 我们也可以在这一点上配置Actor如何部署到一个集群系统.

正如Carl Hewitt所说, 一个Actor不是Actor---这会很孤独，没有人说话. 在示例场景确实有点残忍，因为我们只给了``HelloWorld`` Actor 一个假人来对话--- “ask” 模式(用 ? 操作符表示) 可以用来发送一条信息，这个可以得到回执，取回相应的 Future.

注意这个通过“ask”操作返回的``Future`` 已经是正确的类型, 不需要再检查类型. 这个是因为类型信息是消息协议的一部分:  ? 操作带有一个函数参数，它接受 ``ActorRef[U]`` (这就解释了在上面6行的表达式的空位的 _ ) 和``replyTo`` 参数，我们会填入像``ActorRef[Greeted]``类型, 这就意味着值得到``保证``-只能是``Greeted``类型。

我们使用这个来发送``Greet`` 命令给 Actor，当得到回复时我们会打印它，并告诉actor系统关闭。一旦这个完成以及我们打印``"system terminated"`` 消息，然后程序结束。``recovery`` 组合子需要原来的``Future`` ，为了确保系统即使在出了问题时还可以正确关闭; for 表达式 得到的``flatMap`` 和map连接符，变成只关心 “happy path”，如果``future`` 超时失败那么没有``greeting`` 会被提取和什么都没有发生。

这表明，Actor消息的这些方面可以通过编译器进行类型检查, 但这种能力不是无限的, 我们能静态表达的是有界限的。在我们继续一个更复杂的(和现实)的例子之前，我们加一个小插曲，强调这背后的一些理论。

###一点点理论

The Actor Model as defined by Hewitt, Bishop and Steiger in 1973 is a computational model that expresses exactly what it means for computation to be distributed. The processing units—Actors—can only communicate by exchanging messages and upon reception of a message an Actor can do the following three fundamental actions:

* send a finite number of messages to Actors it knows
* create a finite number of new Actors
* designate the behavior to be applied to the next message

The Akka Typed project expresses these actions using behaviors and addresses. Messages can be sent to an address and behind this façade there is a behavior that receives the message and acts upon it. The binding between address and behavior can change over time as per the third point above, but that is not visible on the outside.

With this preamble we can get to the unique property of this project, namely that it introduces static type checking to Actor interactions: addresses are parameterized and only messages that are of the specified type can be sent to them. The association between an address and its type parameter must be made when the address (and its Actor) is created. For this purpose each behavior is also parameterized with the type of messages it is able to process. Since the behavior can change behind the address façade, designating the next behavior is a constrained operation: the successor must handle the same type of messages as its predecessor. This is necessary in order to not invalidate the addresses that refer to this Actor.

What this enables is that whenever a message is sent to an Actor we can statically ensure that the type of the message is one that the Actor declares to handle—we can avoid the mistake of sending completely pointless messages. What we cannot statically ensure, though, is that the behavior behind the address will be in a given state when our message is received. The fundamental reason is that the association between address and behavior is a dynamic runtime property, the compiler cannot know it while it translates the source code.

This is the same as for normal Java objects with internal variables: when compiling the program we cannot know what their value will be, and if the result of a method call depends on those variables then the outcome is uncertain to a degree—we can only be certain that the returned value is of a given type.

We have seen above that the return type of an Actor command is described by the type of reply-to address that is contained within the message. This allows a conversation to be described in terms of its types: the reply will be of type A, but it might also contain an address of type B, which then allows the other Actor to continue the conversation by sending a message of type B to this new address. While we cannot statically express the “current” state of an Actor, we can express the current state of a protocol between two Actors, since that is just given by the last message type that was received or sent.

In the next section we demonstrate this on a more realistic example.

###A More Complex Example

Consider an Actor that runs a chat room: client Actors may connect by sending a message that contains their screen name and then they can post messages. The chat room Actor will disseminate all posted messages to all currently connected client Actors. The protocol definition could look like the following:

```scala
sealed trait Command
final case class GetSession(screenName: String, replyTo: ActorRef[SessionEvent])
  extends Command
 
sealed trait SessionEvent
final case class SessionGranted(handle: ActorRef[PostMessage]) extends SessionEvent
final case class SessionDenied(reason: String) extends SessionEvent
final case class MessagePosted(screenName: String, message: String) extends SessionEvent
 
final case class PostMessage(message: String)
```

Initially the client Actors only get access to an ActorRef[GetSession] which allows them to make the first step. Once a client’s session has been established it gets a SessionGranted message that contains a handle to unlock the next protocol step, posting messages. The PostMessage command will need to be sent to this particular address that represents the session that has been added to the chat room. The other aspect of a session is that the client has revealed its own address, via the replyTo argument, so that subsequent MessagePosted events can be sent to it.

This illustrates how Actors can express more than just the equivalent of method calls on Java objects. The declared message types and their contents describe a full protocol that can involve multiple Actors and that can evolve over multiple steps. The implementation of the chat room protocol would be as simple as the following:

