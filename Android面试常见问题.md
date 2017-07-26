导读：最近有点时间，所以整理了一下Android面试中经常遇到的一些问题，大部分都是互联网上收集的，很多原作者的链接没有了，非常抱歉，如有问题可直接与我联
系，thanks.

1、Service 和Thread 的区别
a)、Thread: thread是程序执行的最小单元，他是CPU的基本单位。可以用thread来执行一些异步操作。
b)、Service: service是android的一种机制，当它运行的时候如果是LocalService,那么对应的Service运行在主线程的main线程上。如果是RemoteService,那么对
应的service则运行在独立进程的main线程上。
既然这样，那我们为什么还要用Service呢？其实这跟android的系统机制有关，我们先拿Thread来说。Thread的运行是独立于Activity的，也就是说当Activity被
finish之后，如果你没有主动停止Thread或者Thread里的run方法没有执行完毕的话，Thread也会一直执行(PS:当activity被切换到后台时如果系统紧张，系统会随时
结束掉你的process).因此这里会出现一个问题，当activity被finish之后，你不在持有该Thread的引用。另外一方面，你没有办法在不同的activity中对同一个
Thread进行控制。
举个例子：如果你的Thread需要不停的隔一段时间就要连接服务器做某种同步的话，该Thread需要在Activity没有start的时候也在运行。这个时候当你start一个
activity就没办法在该activity里控制之前创建的Thread。因此，你便需要创建并启动一个Service,在Service里面创建、运行并控制该Thread，这样便解决了该问题
（因为任何Activity都可以控制同一Service，而系统也只会创建一个相应的Service实例）。
因此你可以把Service想象成一种消息服务，而你可以在任何有Context的地方调用Context.startService、Context.stopService、Context.bindService、
Context.unbindService来控制它，你也可以在Service里注册BroadcastReceiver，在其他地方通过发送broadcast来控制它，当然，这些都是Thread做不到的。
根据进程的优先级，Thread在后台运行(Activity stop)的优先级低于后台运行的Service，如果执行系统资源紧张，会优先杀死前一种，后台运行的service一般不会
被杀死，如果杀死，系统空闲时会重新启动service。
service运行在主线程中，如果在Service中做耗时任务要开启子线程。

2、Handler是什么？
Handler是android给我们提供用来更新UI的一套机制，也是一套消息处理机制，我们可以发送消息，也可以通过它来处理消息。
a)、为什么要用Handler？
android在设计的时候就封装了一套消息创建、传递、处理机制。如果不遵循这样的机制就没办法更新UI信息，就会抛出异常
（android.view.ViewRootImpl$CalledFromWrongThreadException:
Only the original thread that created a view hierarchy can touch its views.）
b)、android为什么要设计通过Handler机制更新UI呢？
最根本的问题是解决多线程并发的问题。
假设如果在一个activity中，有多个线程去更新UI，并且都没有加锁机制，那么会产生什么样的问题？  更新界面错乱
如果对更新的UI操作都进行加锁处理的话又会产生什么样的问题？    1）、上锁会让UI控件变得复杂和低效；2）、上锁后会阻塞某些进程的执行
出于对以上问题的考虑，android给我们提供了一套更新UI的机制，我们只需要遵循这样的机制就可以了，根本不用关心多线程的问题，所有更新UI操作都是在主线程的
消息队列当中去轮流处理的。
c)、handler的原理
Handler
封装了消息的发送，解决多线程并发引发的问题，地址就是Messagetarget（默认情况发送给自己）
Looper
1）、内部包含一个消息队列也就是MessageQueue，所有的handler发送的消息都走向这个消息队列
2）、Looper.Looper方法，就是一个死循环，不断的从MessageQueue中取消息，如有消息就处理，没有消息就阻塞
总结：handler负责发送消息，Looper负责接收Handler发送的消息，并直接发消息回传给Handler自己，MessageQueue就是一个消息存储容器
![Image text](https://github.com/Don-Lee/Notes/blob/master/Images/handler.png)




3、如何保证Service不被杀死
a)、提高进程的优先级，降低进程被杀死的概率
    1)、利用Activity提升权限
    例：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的Activity，在用户解锁后将Activity销毁。
    适用场景：本方案主要解决第三方应用及系统管理工具在检测到锁屏事件后一段时间（一般为5分钟以内）内会杀死后台进程，已达到省电的目的问题
    2)、利用Notification提升权限
    Android中的Service的优先级为4，通过setForeground接口可以将后台Service设置为前台Service，是进程的优先级由4提升为2，从而使进程的优先级仅仅低于
    用户当前正在交互的进程，与可见进程的优先级一致。
    3)、AndroidManifest设置
    在AndroidManifest.xml文件中对于intent-filter可以通过android:priority="1000"这个属性设置最高优先级，1000是最高值，数字越小，优先级越低，同时
    也适用于广播。
b)、进程死后，进行拉活
    1)、利用系统广播拉活
    例：在发生特定系统事件时（如：屏幕亮灭、锁屏解锁、网络变化，开机等），系统会发出响应的广播，通过在AndroidManifest中“静态”注册对应的广播监听器，
    即可在发生响应的事件拉活。
    2)、利用系统Service机制拉活
    例：将Service设置为START_STICKY，利用系统机制在 Service 挂掉后自动拉活
    3）、利用Native进程进行拉活
    例：利用 Linux 中的 fork 机制创建 Native 进程，在 Native 进程中监控主进程的存活，当主进程挂掉后，在 Native 进程中立即对主进程进行拉活。
    4)、自定义广播拉活
    例：service +broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service
c)、通过推送拉活
    根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。国外版应用
    可考虑接入 Google 的 GCM。
参考博客：https://mp.weixin.qq.com/s?
__biz=MzA3NTYzODYzMg==&mid=2653577617&idx=1&sn=623256a2ff94641036a6c9eea17baab8&scene=0#wechat_redirect

4、Activity和Fragment的生命周期
![Image text](https://github.com/Don-Lee/Notes/blob/master/Images/activity_fragment.png)


5、Activity的任务栈
在AndroidManifest.xml中使用android:launchMode="standard|singleInstance|singleTask|SingleTop"来控制activity的任务栈。

任务栈是一种后进先出的结构。位于栈顶的Activity处于焦点状态，当按下back键时，栈内的activity会一个一个的出栈，并调用onDestory()方法。如果栈内没有
activity，那么系统就会回收这个栈，每个app默认只有一个栈，以app的包名来命名.
      standard:标准模式，每次启动activity都会创建一个新的activity实例，并且将其压入任务栈栈顶，而不管这个activity是否已经存在。
      singleTop:顶单例模式（栈顶复用模式），这种模式和standard模式基本相似，单有一点不同，当要被启动的目标activity已经位于栈顶时，系统不会重新创建
      目标activity的实例，而是直接复用已有的activity实例。
      singleTask：内单例模式(栈内复用模式)，采用这种加载模式的Activity在同一个任务栈内只有一个实例，当采用此模式启动activity时，可分为如下三种情
      况：
	a）、如果要启动的activity不存在，系统则会创建activity的实例，并将它加到栈顶。
	b)、如果要启动的activity已经位于栈顶，此时与singleTop相同
	c)、如果要启动的activity已经存在，但是没有位于栈顶，系统将会把位于该activity上面的所有的activity移出栈，从而使得目标acitivity转入栈顶。
      singleInstance：全局单例模式，这种加载模式，系统保证       无论从那个任务栈中启动目标activity，只会创建一个目标activity实例，并会使用一个全
      新的的任务栈来装载该activity实例。
	采用这种方式启动activity，分为如下2种情况：
	a)、如果将要启动的目标activity不存在，系统会先创建一个全新的栈，再创建目标activity的实例，并将他加入新的栈的栈顶。
	b)、如果要启动的activity已经存在，无论它位于哪个应用程序中，无论位于那个栈中，系统都会把该activity所在的栈转到前台，从而使该activity显示出
	来。

