# 面试题
> 为什么说Objective-C是一门动态的语言？
*  动态类型：
   如id类型。实际上静态类型因为其
   固定性和可预知性而使用的特别广泛。静态类型是强类型，动态类型是弱类型，运行时决定接收者。
*  动态绑定：
   让代码在运行时判断需要
   调用什么方法，而不是在编译时。与其他面向对象语言一样，方法调用和代码并没有在编译时连接在一起，而是在消息发送时才进行连接。运行时决定调用哪个方法。
*  动态载入：
   让程序在运行时添加代码模块以及其他资源。用户可以根据需要执行一些可执行代码和资源，而不是在启动时就加载所有组件。可执行代码中可以含有和程序运行时整合的新类。

> 讲一下MVC和MVVM，MVP？

[讲解](%20http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)

* MVC
  1. View 传送指令到 Controller
  2. Controller 完成业务逻辑后，要求 Model 改变状态
  3. Model 将新的数据发送到 View，用户得到反馈
* MVP
  1. 各部分之间的通信，都是双向的。
  2. View 与 Model 不发生联系，都通过 Presenter 传递。
  3. View 非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署在那里。
* MVVM
   MVVM 模式将 Presenter 改名为 ViewModel，基本上与 MVP 模式完全一致。唯一的区别是，它采用双向绑定（data-binding）：View的变动，自动反映在 ViewModel，反之亦然。Angular 和 Ember 都采用这种模式。
> 为什么代理要用weak？代理的delegate和dataSource有什么区别？block和代理的区别?

为了避免循环引用。weak指明该对象并不负责保持delegate这个对象，delegate这个对象的销毁由外部控制。strong该对象强引用delegate，外界不能销毁delegate对象，会导致循环引用。

DataSource是关于View的内容的东西包括属性，数据等等，而Delegate则是一些我们可以调用的方法，全是操作。
block和代理都能解决对象间交互的问题，block更轻型，更简单，能够直接访问上下文，代码通常在同一个地方，这样读代码也连贯。缺点是容易引起循环引用。delegate更重一些，需要实现接口，它的方法分开来，很多时候需要存储一些临时数据，另外相关的代码需要分离到各处没有block好读，其优点就是它是用weak关键字修饰的，不会引起循环引用。

> 属性的实质是什么？包括哪几个部分？属性默认的关键字都有哪些？@dynamic关键字和@synthesize关键字是用来做什么的？

属性的本质是@property = ivar+getter+setter,也就是说@property系统会自动生成getter和setter方法。属性默认的关键字包括atomic，nonatomic，@synthesize，@dynamic，getter=getterName，setter=setterName，readwrite，readonly，assign，retain，copy。
@dynamic：表示变量对应的属性访问器方法，是动态实现的，你需要在 NSObject 中继承而来的 +(BOOL) resolveInstanceMethod:(SEL) sel 方法中指定 动态实现的方法或者函数。
@synthesize：如果没有实现setter和getter，编译器能够自动实现getter和setter方法

> 如何令自己所写的对象具有拷贝功能?

若想让自己写的对象具有拷贝功能，则需要实现NSCopying协议。如果自定义的对象分为可变版本和非可变版本，那么就要同时实现NSCopying和NSMutableCopying协议，不过一般没什么必要，实现NSCopying协议就够了。

> 可变集合类 和 不可变集合类的 copy 和 mutablecopy有什么区别？如果是集合是内容复制的话，集合里面的元素也是内容复制么？

对于不可变对象，copy操作是浅复制，mutableCopy是深复制。对于不可变对象，mutableCopy不仅仅是深复制，返回的对象类型还是不可变对象类型相应的可变对象的类型。内容复制也就是深拷贝，集合的深复制有两个方法，可以用initWithArray:copyItems:将第二个参数设置为YES即可进行深复制，如：NSDictionary shallowCopyDict = [NSDictionary alloc](#)initWithDictionary:someDictionary copyItems:YES];如果用这个方法深复制，集合里的每个元素都会收到copyWithZone：消息。如果集合里的对象遵循NSCopying协议，那么对象就会深复制到新的集合。如果对象没有遵循NSCopying协议，而尝试用这种方法进行深复制则会出错。copyWithZone：这种拷贝方式只能提供一层内存拷贝，而非真正的深拷贝。第二种方法是将集合进行归档解档，如：`NSArray trueDeepCopyArray = NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithRootObject:oldArray];`

> nonatomic和atomic的区别？atomic是绝对的线程安全么？为什么？如果不是，那应该如何实现？

nonatomic和atomic的区别在于两者自动生成getter和setter的方法不一样，如果你自己写getter和setter方法，那么（getter，setter，retain，copy，assign）只起提示作用，写不写都一样。
对于atomic的属性，系统生成的getter和setter会保证get，set的操作完整性，不受其他线程影响。比如线程A的getter方法运行到一半，线程B调用了setter，那么线程A的getter还是能得到一个完整的对象。
而nonatomic就没有这个保证了，所以速度要比atomic快。
不过atomic可不能保证线程安全，如果线程A调用了getter，与此同时线程B和线程C都调了setter，那最后线程Aget到的值，三种都有可能：可能是B，C set之前原始的值，也可能是B set的值，也可能是C set的值。同时这个最终的值，也可能是B set的值，也可能是C set的值。要保证安全，可以使用线程锁。

> 用StoryBoard开发界面有什么弊端？如何避免？

如果需要使用Storyboard，但又想最大化地避免冲突呢？最好的方法就是将UI划分的更小的、不同的Storyboard文件中。在过去如果想要做到这一点，意味跨Storyboard的跳转方法

在Xcode7 和 iOS 9中，只需要用Storyboard Reference就可以用Segue轻松实现跨Storyboard的跳转了。Storyboard Reference的出现，保留了单个Storyboard文件跳转的优点的同时，提供了多Storyboard文件时利于合并的便利。

> 进程和线程的区别？同步异步的区别？并行和并发的区别？

进程是一个内存中运行的应用程序，比如在Windows系统中，一个运行的exe就是一个进程。
线程是指进程中的一个执行流程。
同步是顺序执行，执行完一个再执行下一个。需要等待，协调运行。
异步就是彼此独立，在等待某事件的过程中继续做自己的事，不需要等待这些事件完成后再工作。
并行和并发 是前者相当于三个人同时吃一个馒头，后者相当于一个人同时吃三个馒头。
并发性（Concurrence）：指两个或两个以上的事件或活动在同一时间间隔内发生。并发的实质是一个物理CPU(也可以多个物理CPU) 在若干道程序之间多路复用，并发性是对有限物理资源强制行使多用户共享以提高效率。
并行性（parallelism）指两个或两个以上事件或活动在同一时刻发生。在多道程序环境下，并行性使多个程序同一时刻可在不同CPU上同时执行。
区别：(并发)一个处理器同时处理多个任务和（并行）多个处理器或者是多核的处理器同时处理多个不同的任务。

> GCD的一些常用的函数？（group，barrier，信号量，线程同步）

1.延迟执行任务函数：dispatch_after(.....)。
2.一次性执行dispatch_once(...)。
3.栅栏函数dispatch_barrier_async/dispatch_barrier_sync。
4.队列组的使用dispatch_group_t。
5.GCD定时器
如何使用队列来避免资源抢夺？
dispatch_barrior_async 作用是在并行队列中，等待前面两个操作并行操作完成。

> NSCache优于NSDictionary的几点？
>  NSCache 是一个容器类，类似于NSDIctionary,通过key-value 形式存储和查询值，用于临时存储对象。

1. NSCache 中的key不必实现copy，NSDictionary中的key必须实现copy。
2. NSCache中存储的对象也不必实现NSCoding协议，因为毕竟是临时存储，类似于内存缓存，程序退出后就被释放了。
> 知不知道Designated Initializer？使用它的时候有什么需要注意的问题？
> 指定初始化函数，继承父类后变成便利初始化函数，

> 实现description方法能取到什么效果？

1. NSLog(@"%@", objectA);这会自动调用objectA的description方法来输出ObjectA的描述信息.
2. description方法默认返回对象的描述信息(默认实现是返回类名和对象的内存地址)
3. description方法是基类NSObject 所带的方法,因为其默认实现是返回类名和对象的内存地址, 这样的话,使用NSLog输出OC对象,意义就不是很大,因为我们并不关心对象的内存地址,比较关心的是对象内部的一些成变量的值。因此,会经常重写description方法,覆盖description方法的默认实现
> objc使用什么机制管理对象内存？

通过 retainCount 的机制来决定对象是否需要释放。 每次 runloop 的时候，都会检查对象的 retainCount，如果retainCount 为 0，说明该对象没有地方需要继续使用了，可以释放掉了。

> block的实质是什么？一共有几种block？都是什么情况下生成的？

block对象就是一个结构体，里面有isa指针指向自己的类（global malloc stack），有desc结构体描述block的信息，__forwarding指向自己或堆上自己的地址，如果block对象截获变量，这些变量也会出现在block结构体中。最重要的block结构体有一个函数指针，指向block代码块。block结构体的构造函数的参数，包括函数指针，描述block的结构体，自动截获的变量（全局变量不用截获），引用到的__block变量。(__block对象也会转变成结构体)
block代码块在编译的时候会生成一个函数，函数第一个参数是前面说到的block对象结构体指针。执行block，相当于执行block里面__forwarding里面的函数指针__

> 为什么在默认情况下无法修改被block捕获的变量？ __block都做了什么？

[讲解](http://www.jianshu.com/p/93041852ff21)

> objc在向一个对象发送消息时，发生了什么？

[讲解](http://www.jianshu.com/p/b7cda433e4f5)

objc在向一个对象发送消息时，runtime库会根据对象的isa指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，然后在发送消息的时候，objc_msgSend方法不会返回值，所谓的返回内容都是具体调用时执行的

> 什么时候会报unrecognized selector错误？iOS有哪些机制来避免走到这一步？

[讲解](http://www.jianshu.com/p/4d40ff407d0e)

> 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

不能向编译后得到的类中增加实例变量；
能向运行时创建的类中添加实例变量；

原因如下：

因为编译后的类已经注册在 runtime 中，类结构体中的 objc_ivar_list 实例变量的链表 和 instance_size 实例变量的内存大小已经确定，同时runtime 会调用 class_setIvarLayout 或 class_setWeakIvarLayout 来处理 strong weak 引用。所以不能向存在的类中添加实例变量；

运行时创建的类是可以添加实例变量，调用 class_addIvar 函数。但是得在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前，原因同上。

> runtime如何实现weak变量的自动置nil？

runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

> 给类添加一个属性后，在类结构体里哪些元素会发生变化？

????

> runloop是来做什么的？runloop和线程有什么关系？主线程默认开启了runloop么？子线程呢？

总的说来，Run loop，正如其名，loop表示某种循环，和run放在一起就表示一直在运行着的循环。
实际上，run loop和线程是紧密相连的，可以这样说run loop是为了线程而生，没有线程，它就没有存在的必要。
Run loops是线程的基础架构部分，Cocoa和CoreFundation都提供了run loop对象方便配置和管理线程的run loop。
每个线程，包括程序的主线程（main thread）都有与之相应的run loop对象。
runloop和线程的关系：主线程的run loop默认是启动的, 子线程的runloop默认是不开启的,需要我们自己手动开启循环; 。



> runloop的mode是用来做什么的？有几种mode？



* 一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer
* 每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode
* 如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入
* 这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响
* NSDefaultRunLoopMode：App的默认Mode，通常主线程是在这个Mode下运行

**系统默认注册了5个Mode:**

1. NSDefaultRunLoopMode：App的默认Mode，通常主线程是在这个Mode下运行
2. UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
3. UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
4. GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
5. NSRunLoopCommonModes: 这是一个占位用的Mode，不是一种真正的Mode

> 为什么把NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环以后，滑动scrollview的时候NSTimer却不动了？

`如果我们把一个NSTimer对象以NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不再被调度。当我们滚动的时候，也希望不调度，那就应该使用默认模式。但是，如果希望在滚动时，定时器也要回调，那就应该使用common mode。`

```objective-c
 //将timer添加到NSDefaultRunLoopMode中
 [NSTimer scheduledTimerWithTimeInterval: target: selector:@selector(timerTick:) userInfo: repeats:];
  //然后再添加到NSRunLoopCommonModes里
   NSTimer *timer = [NSTimer timerWithTimeInterval: target: selector:@selector(timerTick:) userInfo: repeats:];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```



> 苹果是如何实现Autorelease Pool的？

[讲解](http://www.jianshu.com/p/cc3ee2909457)
