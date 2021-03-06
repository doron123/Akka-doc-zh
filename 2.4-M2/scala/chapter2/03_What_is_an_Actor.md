# 什么是Actor?

上一节[Actor系统](02_Actor_Systems.md) 解释了actor是应用创建中最小的单元，以及它们如何组成一个层次结构。本节单独来看看一个actor，解释在实现它时你会遇到的概念。更多细节请参阅 [Actors (Scala)](../chapter3/01_Actors.md)和 [Actors (Java)](#TODO).

一个Actor是一个容器，它包含了[状态](#state)，[行为](#behavior)，一个[邮箱](#mailbox)，[孩子](#children)和一个[监管策略](#supervisor-strategy)。所有这些封装在一个[Actor引用](#actor-reference)里。最终在Actor终止时，会有[这些](#when-an-actor-terminates)发生。

###<a name="actor-reference"></a>Actor引用

如下所述，一个actor对象需要与外界隔离开才能从actor模型中受益。因此actor是以actor引用的形式展现给外界的，actor引用作为对象，可以不受限制地自由传递。内部和外部对象的这种划分使得所有想要的操作都能够透明：重启actor而不需要更新别处的引用，将实际actor对象放置到远程主机上，向另外一个应用程序中的actor发送消息。但最重要的方面是从外界不可能到actor对象的内部获取其状态，除非这个actor非常不明智地将信息公布出去。

###<a name="state"></a>状态
Actor对象通常包含一些变量来反映其所处的可能状态。这可以是一个显式状态机（例如使用 [FSM 模块](../chapter3/07_FSM.md))，或是一个计数器，一组监听器，待处理的请求，等等。这些数据使得actor有价值，并且必须将这些数据保护起来不被其它actor所破坏。好消息是在概念上每个Akka actor都有自己的轻量线程，它与系统其它部分是完全隔离的。这意味着你不需要使用锁来进行资源同步，可以直接编写你的actor代码，完全不必担心并发问题。

在幕后，Akka会在一组真实线程上运行Actor组，通常是很多actor共享一个线程，对某一个actor的调用可能会在不同的线程上得到处理。Akka保证这个实现细节不影响处理actor状态的单线程性。

由于内部状态对于actor的操作是至关重要的，所以状态不一致是致命的。因此当actor失败并被其监管者重新启动时，状态会被重新创建，就象第一次创建这个actor一样。这是为了实现系统的“自愈合”。

可选地，通过持久化收到的消息并在重启后回放它们，一个actor的状态可自动恢复到重启前的状态（详见[持久化](../chapter3/08_Persistence.md)）

###<a name="behavior"></a>行为
每当一个消息被处理，它会与actor的当前行为进行匹配。行为是一个函数，它定义了在某个时间点处理当前消息所要采取的动作，例如如果客户已经授权，那么就对请求进行转发处理，否则拒绝。这个行为可能随着时间而改变，例如由于不同的客户在不同的时间获得授权，或是由于actor进入了“非服务状态”模式，之后又变回来。这些变化的实现，要么是通过将它们编码入状态变量中并由行为逻辑读取，要么是函数本身在运行时被交换出来，见`become` 和 `unbecome`操作。但是actor对象在创建时所定义的初始行为是特殊的，因为actor重启时会恢复这个初始行为。


###<a name="mailbox"></a>邮箱
Actor的目的是处理消息，这些消息是从其它actor（或者从actor系统外部）发送来的。连接发送者与接收者的纽带是actor的邮箱：每个actor有且仅有一个邮箱，所有的发来的消息都在邮箱里排队。排队按照发送操作的时间顺序来进行，这意味着由于actor分布在不同的线程中，所以从不同的actor发来的消息在运行时没有一个固定的顺序。从另一个角度讲，从同一个actor发送到相同目标actor的多个消息，会按发送的顺序排队。

可以有不同的邮箱实现供选择，缺省的是FIFO：actor处理消息的顺序与消息入队列的顺序一致。这通常是一个好的默认选择，但是应用有可能需要对某些消息进行优先处理。在这种情况下，可以使用优先邮箱来根据消息优先级将消息放在非队尾的某个指定位置，甚至可能是队列头。如果使用这样的队列，消息的处理顺序是由队列的算法决定的，而不是FIFO。

Akka与其它actor模型实现的一个重要区别在于：当前的行为总是必须处理下一个从队列中取出的消息，Akka不会扫描邮箱队列来获取下一个匹配的消息。无法处理某个消息通常被认为是失败情况，除非这个行为被重写。

###<a name="children"></a>孩子
每个actor都是一个潜在的监管者：如果它创建子actor来委派处理子任务，它会自动地监管它们。子actor列表维护在actor的上下文中，actor可以访问它。对列表的更改是通过创建(`context.actorOf(...)`)或者停止(`context.stop(child)`)子actor来完成，并且这些更改会立刻生效。实际的创建和停止操作是在幕后以异步方式完成的，这样它们就不会“阻塞”其监管者。

###<a name="supervisor-strategy"></a>监管策略
Actor的最后一部分是它用来处理其子actor错误状况的机制。错误处理是由Akka透明完成的，针对每个出现的失败，将应用[监管与监控](04_Supervision_and_Monitoring.md)中所描述的一个策略。由于策略是actor系统组织结构的基础，所以一旦actor被创建，它就不能被修改。

考虑到对每个actor只有唯一的策略，这意味着：如果一个actor的子actor们应用了不同的策略，则这些子actor应该按照相同的策略来进行分组，并放在一个中间的监管者下，又一次转向了根据任务到子任务的划分来组织actor系统的结构的设计方法。

###<a name="when-an-actor-terminates"></a>当Actor终止时
当一个actor终止，例如失败了且不能用重启来解决、停止它自己或者被它的监管者停止，它会释放其资源，将其邮箱中所有未处理的消息放进系统的“死信邮箱(dead letter mailbox)”，即将所有消息作为死信重定向到事件流中。而actor引用中的邮箱将会被一个系统邮箱所替代，将所有的新消息作为死信重定向到事件流中。 但是这些操作只是尽力而为，所以不能依赖它来实现“投递保证”。

不是简单地把消息扔掉的想法来源于我们的测试：我们在事件总线上注册了`TestEventListener`来接收死信，然后将每个收到的死信在日志中生成一条警告——这对于更快地解析测试失败非常有帮助。可以想象这个特性也可以用于其它的目的。



