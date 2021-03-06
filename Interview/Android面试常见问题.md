导读：最近有点时间，所以整理了一下Android面试中经常遇到的一些问题，大部分都是互联网上收集的，很多原作者的链接没有了，非常抱歉，如有问题可直接与我联
系，thanks.

1、Service 和Thread 的区别 
-----
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
----
Handler是android给我们提供用来更新UI的一套机制，也是一套消息处理机制，我们可以发送消息，也可以通过它来处理消息。 

#### a)、为什么要用Handler？       
android在设计的时候就封装了一套消息创建、传递、处理机制。如果不遵循这样的机制就没办法更新UI信息，就会抛出异常
（android.view.ViewRootImpl$CalledFromWrongThreadException:
Only the original thread that created a view hierarchy can touch its views.）   

#### b)、android为什么要设计通过Handler机制更新UI呢？   
最根本的问题是解决多线程并发的问题。   
假设如果在一个activity中，有多个线程去更新UI，并且都没有加锁机制，那么会产生什么样的问题？     
更新界面错乱     
如果对更新的UI操作都进行加锁处理的话又会产生什么样的问题？    
1）、上锁会让UI控件变得复杂和低效；       
2）、上锁后会阻塞某些进程的执行   
出于对以上问题的考虑，android给我们提供了一套更新UI的机制，我们只需要遵循这样的机制就可以了，根本不用关心多线程的问题，所有更新UI操作都是在主线程的
消息队列当中去轮流处理的       

#### c)、handler的原理      
Handler       
封装了消息的发送，解决多线程并发引发的问题，地址就是Messagetarget（默认情况发送给自己）   

Looper  
1）、内部包含一个消息队列也就是MessageQueue，所有的handler发送的消息都走向这个消息队列      
2）、Looper.Looper方法，就是一个死循环，不断的从MessageQueue中取消息，如有消息就处理，没有消息就阻塞     
总结：handler负责发送消息，Looper负责接收Handler发送的消息，并直接发消息回传给Handler自己，MessageQueue就是一个消息存储容器  
![Image text](https://github.com/Don-Lee/Notes/blob/master/Images/handler.png)




3、如何保证Service不被杀死 
----
#### a)、提高进程的优先级，降低进程被杀死的概率     
    1)、利用Activity提升权限       
    例：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的Activity，在用户解锁后将Activity销毁。        
    适用场景：本方案主要解决第三方应用及系统管理工具在检测到锁屏事件后一段时间（一般为5分钟以内）内会杀死后台进程，
    已达到省电的目的问题   
    
    2)、利用Notification提升权限                                                                
    Android中的Service的优先级为4，通过setForeground接口可以将后台Service设置为前台Service，是进程的优先级由4提升为2，
    从而使进程的优先级仅仅低于用户当前正在交互的进程，与可见进程的优先级一致。  
    
    3)、AndroidManifest设置                                     
    在AndroidManifest.xml文件中对于intent-filter可以通过android:priority="1000"这个属性设置最高优先级，1000是最高值，
    数字越小，优先级越低，同时也适用于广播。  
    
#### b)、进程死后，进行拉活     
    1)、利用系统广播拉活              
    例：在发生特定系统事件时（如：屏幕亮灭、锁屏解锁、网络变化，开机等），系统会发出响应的广播，通过在AndroidManifest中“静态”注册对应的广播监听器，
    即可在发生响应的事件拉活。   
    
    2)、利用系统Service机制拉活                  
    例：将Service设置为START_STICKY，利用系统机制在 Service 挂掉后自动拉活  
    
    3)、利用Native进程进行拉活                    
    例：利用 Linux 中的 fork 机制创建 Native 进程，在 Native 进程中监控主进程的存活，当主进程挂掉后，
    在Native进程中立即对主进程进行拉活。   
    
    4)、自定义广播拉活                  
    例：service +broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service   
    
#### c)、通过推送拉活     
    根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。
    国外版应用可考虑接入 Google 的 GCM。       

4、Activity和Fragment的生命周期     
-----
![Image text](https://github.com/Don-Lee/Notes/blob/master/Images/activity_fragment.png)   




5、Activity的任务栈    
-----
在AndroidManifest.xml中使用android:launchMode="standard|singleInstance|singleTask|SingleTop"来控制activity的任务栈。     

任务栈是一种后进先出的结构。位于栈顶的Activity处于焦点状态，当按下back键时，栈内的activity会一个一个的出栈，并调用onDestory()方法。如果栈内没有
activity，那么系统就会回收这个栈，每个app默认只有一个栈，以app的包名来命名.    

#### standard:标准模式
每次启动activity都会创建一个新的的activity实例，并且将其压入任务栈栈顶，而不管这个activity是否已经存在。    

#### singleTop:顶单例模式（栈顶复用模式）
这种模式和standard模式基本相似，但有一点不同，当要被启动的目标activity已经位于栈顶时，系统不会重新创建目标
activity的实例，而是直接复用已有的activity实例。 

#### singleTask：内单例模式(栈内复用模式)
采用这种加载模式的Activity在同一个任务栈内只有一个实例，当采用此模式启动activity时，可分为如下三种情况：   
	a）、如果要启动的activity不存在，系统则会创建activity的实例，并将它加到栈顶。    
	b)、如果要启动的activity已经位于栈顶，此时与singleTop相同    
	c)、如果要启动的activity已经存在，但是没有位于栈顶，系统将会把位于该activity上面的所有的activity移出栈，从而使得目标acitivity转入栈顶。
	
#### singleInstance：全局单例模式
这种加载模式，系统保证无论从那个任务栈中启动目标activity，只会创建一个目标activity实例，并会使用一个全新的的任务栈来装载该activity实例。          
	采用这种方式启动activity，分为如下2种情况：     
	a)、如果将要启动的目标activity不存在，系统会先创建一个全新的栈，再创建目标activity的实例，并将他加入新的栈的栈顶。 
	b)、如果要启动的activity已经存在，无论它位于哪个应用程序中，无论位于那个栈中，系统都会把该activity所在的栈转到前台，从而使该activity显示出
	来。

6、android中有哪几种解析xml的类，官方推荐哪种？以及他们的原理和区别
----------
- DOM解析  
优点：   
a)、XML树在内存中完整存储，因此可以直接修改其数据和结构.   
b)、可以通过该解析器随时访问XML树中的任何一个节点.   
c)、DOM解析器的API在使用上也现对比较简单  <br/>
缺点：  <br/>
如果XML文档体积比较大时，将文档读入内存是非常消耗系统资源的。    <br/>
- SAX解析 <br/>
优点： <br/>
SAX对内存的要求比较低，因为它让开发人员自己来决定所要处理的标签。特别是当开发人员只需要处理文档中所包含的部分数据时，SAX这种扩展能力得到了更好的体现。  <br/> 
缺点：<br/>
用SAX进行XML解析时，需要顺序执行，所以很难访问到同一文档中的不同数据。此外，在基于该方式的解析编码过程也相对复杂。  <br/>            
- Xmlpull解析  <br/>
android SDK提供了xmlpull api，xmlpull和sax类似，是基于流(stream)操作文件，然后根据节点事件回调开发者编写的处理
程序，因为是基于流的处理，因此xmlpull和sax都比较节约内存资源，不会像dom那样要把所有节点以橡树的形式展开在内存中，xmlpull
比sax更简明，而且不需要扫描完整个流。                  

7、Jar和Aar的区别
----------
jar里只有代码，aar里面除了代码还包括资源文件，比如drawable文件，xml资源文件。对于一些不常变动的Android Library，我们可以直接引用aar，
加快编译速度。         

8、Fragment特点
---------- 
Fragment可以作为Activity界面的一部分出现；<br/>
可以在一个Activity中同时出现多个Fragment，并且一个Fragment也可以在多个Activity中使用；    <br/>
在Activity运行过程中，可以添加、移除或者替换Fragment；        <br/>
Fragment可以响应自己的输入事件，并且有自己的生命周期，它的生命周期会受宿主Activity的生命周期影响。                

9、怎么考虑数据传输的安全性
-----------------
如果如果应用对传输的数据没有任何安全措施，攻击者设置的钓鱼网络中更改DNS服务器。这台服务器可以获取用户信息，
或者充当中间人与源服务器交换数据。在SSL/TLS通信中，客户端可以通过数字证书判断服务器是否可信，并采用证书的公钥与服务器进行加密通讯。                 

10、移动端获取网络数据优化的几个点
----------------
连接复用：节省连接建立时间，如开启keep-alive.                                
对于android来说默认情况下HttpURLConnetctionh和HttpClient都开启了keep-alive。只是2.2之前HttpURLConnection存在影响连接池的bug.              

请求合并：即将多个请求合并为一个请求，比较常见的就是网页中的css image sprites。              

减少请求数据的大小： 对于post请求，body可以做gzip压缩的，header也可以做数据压缩(不过只支持Http2.0).                             
返回数据的body也可以做gzip压缩，body数据体积可以缩小到原来的30%左右。（也可以考虑压缩返回的json数据的key数据体积，尤其是针对
返回数据格式变化不大的情况，支付宝聊天返回的数据用到了）<br/>
根据用户的当前的网络质量来判断下载什么质量的图片（电商用的较多）                              

11、如何避免内存溢出
-------
1)、尽量在不显示的时候释放内存                       
2)、对不需要强引用的对象采用弱引用                  
3)、对质量要求不大的图片采用 RBG_565 格式                  
4)、对可重用对象进行重用                 
5)、谨慎使用依赖注入                 
6)、减少布局的嵌套层次            
7)、尽量少用services             
8)、对webview退出时进行回收               
9)、能使用application Context 的时候尽量不使用 Activity Context              
10)、使用弱引用handler
11）、移除掉所有的静态引用。
12)、考虑用 EventBus 来解耦 Listener。
13)、记着在不需要的时候，解除 Listener 的绑定。
14）、尽量用静态内部类。

12、Retrofit + okHttp + RxJava
------------
Retrofit:
	Retrofit是简化HTTP请求的库，retrofit 2背后的Http client只支持okHttp，详情请参考：https://academy.realm.io/cn/posts/droidcon-jake-wharton-simple-http-retrofit-2/
	
RxJava:
	rxJava为android 提供了一种全新的异步操作机制，详情请参考：https://academy.realm.io/cn/posts/360andev-christina-lee-intro-rxjava-java-android/
	
okHttp:


12、面向对象
--------
面向对象是一种对现实世界理解和抽象的方法，可以把一切事物都看为对象，开发时对其的属性及行为进行抽象和封装，而且其具有继承和多态性，可以理解为面向类开发（类的实例即对象）


13、Android 两种序列化的区别与作用
------------
Serializable: 使用简单，但是需要做大量的I/O操作，开销很大，效率低，适用于序列化到本地或网络传输. <br/> 
Parcelable: 直接在内存上读写，开销小，效率高，适用于运行时数据传递，比如使用Intent，Bundle 等传递数据; 但是Parcelable无法将数据进行持久化,因此在将数据保存在磁盘的时候,仍然需要使用Serializable 
	
14、Android几种数据存储方式的应用场景
---------
- SharedPreferences:<br/>		
适用于保存少量数据，且这些数据的格式非常简单：字符串型、基本类型的值
- File 存储<br/>		
适用于存储多媒体文件, 文件缓存
- Sqlite数据库<br/>		
适用于对保存的数据进行一些复杂的操作，或者数据量很大，超出了文本文件和Preference的性能的范围的数据
- ContentProvider<br/>		
适用于进程(应用程序)间数据共享的数据
- 网络存储<br/>
由于手机内存的限制，和数据实时性的要求，手机APP大部分的数据来源还是来自于服务器，通过调动接口获取，本地只是作为缓存，辅助存储用。比如天气数据等

15、Merge、ViewStub的作用
---------
### Merge:  
merge用于消除视图层次结构中的冗余视图,主要用于辅助include标签,例如根布局是Linearlayout,那么我们又include一个LinerLayout布局就没意义了,反而会减慢UI加载速度
##### 使用场景
1. 根布局是FrameLayout且不需要设置background或padding等属性,可以用merge代替,因为Activity的ContentView父元素就是FrameLayout,所以可以用merge消除只剩一个.<br/>
2. 某布局作为子布局被其他布局include时,使用merge当作该布局的顶节点,这样在被引入时顶结点会自动被忽略,而将其子节点全部合并到主布局中.<br/>
3. 自定义View如果继承LinearLayout(ViewGroup),建议让自定义View的布局文件根布局设置成merge,这样能少一层结点.
##### 注意事项
1. 因为merge标签并不是View,所以在通过LayoutInflate.inflate()方法渲染的时候,第二个参数必须指定一个父容器,且第三个参数必须为true,也就是必须为merge下的视图指定一个父亲节点.<br/>
2. 因为merge不是View,所以对merge标签设置的所有属性都是无效的.<br/>
3. merge标签必须使用在根布局<br/>
4. ViewStub标签中的layout布局不能使用merge标签
### ViewStub
ViewStub 标签最大的优点是当你需要时才会加载,使用它并不会影响UI初始化时的性能.各种不常用的布局像进度条、显示错误消息等可以使用ViewStub标签,以减少内存使用量,加快渲染速度.ViewStub是一个不可见的,实际上是把宽高设置为0的View.效果有点类似普通的view.setVisible(),但性能体验提高不少  

第一次初始化时,初始化的是ViewStub View,当我们调用inflate()或setVisibility()后会被remove掉,然后在将其中的layout加到当前view hierarchy中

##### 注意事项
1. ViewStub标签不支持merge标签<br/>
2. 使用ViewStub时应先判断ViewStub(做单例)是否已经加载过,因为ViewStub的inflate只能被调用一次,第二次调用会抛出异常,setVisibility可以被调用多次,但不建议这么做(ViewStub 调用过后,可能被GC掉,再调用setVisibility()会报异常)<br/>
3. 为ViewStub赋值的android:layout_XX属性会替换待加载布局文件的根节点对应的属性

16、Android App的启动过程？怎么加速启动App?
-----------
- app启动方式  
启动方式分为：冷启动、热启动、温启动。
1. 冷启动  
启动app时，后台没有app的进程，或者进程被killed，这叫冷启动。冷启动因为系统会重新创建一个新的进程分配给它，所以会先创建和初始化Application类，再创建和初始化MainActivity类（包括一系列的测量、布局、绘制），最后显示在界面上。
2. 热启动  
启动app时，后台已有app的进程（例：按back键、home键，应用虽然会退出，但是该应用的进程是依然会保留在后台，可进入任务列表查看），所以在已有进程的情况下，这种启动会从已有的进程中来启动应用，这个方式叫热启动。  
热启动因为会从已有的进程中来启动，所以热启动就不会走Application这步了，而是直接走MainActivity（包括一系列的测量、布局、绘制），所以热启动的过程只需要创建和初始化一个MainActivity就行了，而不必创建和初始化Application，因为一个应用从新进程的创建到进程的销毁，Application只会初始化一次。
3. 温启动  
介于冷启动和热启动之间, 一般来说在以下两种情况下发生:
用户back退出了App, 然后又启动. App进程可能还在运行, 但是activity需要重建。用户退出App后, 系统可能由于内存原因将App杀死, 进程和activity都需要重启, 但是可以在onCreate中将被动杀死锁保存的状态(saved instance state)恢复。
通过三种启动状态的相关描述, 可以看出我们要做的启动优化其实就是针对冷启动. 热启动和温启动都相对较快.
- App启动过程  
App启动进程大致可以分为3步：
1. 创建进程  
当用户点击app的启动图标后，AMS(ActivityManagerService)会创建一个新的进程分配给应用  
2. 绑定Application  
接下来要做的就是将进程和指定的Application绑定起来，这个是通过ActivityThread对象中调用bindApplication()方法完成的  
3. 启动Activity
经过前两个步骤之后, 系统已经拥有了该application的进程. 后面的调用顺序就是普通的从一个已经存在的进程中启动一个新进程的activity了.  

更多详情可参考[[译]Android Application启动流程分析](https://www.jianshu.com/p/a5532ecc8377)

- 扩展
1. AMS（ActivityManagerService）
Activity的管理者。其实除了Activity，AMS也管Service等组件信息，另外AMS还管理Process信息。  
2. ActivityThread  
每一个App进程有一个主线程，它由ActivityThread描述。它负责这个App进程中各个Activity的调度和执行，以及响应AMS的操作请求等。  
3. ApplicationThread  
AMS和Activity通过它进行通信。对于AMS而言，ApplicationThread代表了App的主线程。简而言之，它是AMS与ActivityThread进行交互的接口。注意ActivityThread和ApplicationThread之间的关系并不像Activity与Application。后者的关系是Application中包含了多个Activity，而前者ActivityThread和ApplicationThread是同一个东西的两种"View"，ApplicationThread是在AMS眼中的ActivityThread。

- App加速启动优化方案  
1. 利用提前展示出来的Window，快速展示出来一个界面，给用户快速反馈的体验(闪屏页面)；
2. 避免在启动时做密集沉重的初始化（Heavy app initialization）；
3. 定位问题：避免I/O操作、反序列化、网络操作、布局嵌套等。  

更多详情可参考[Android性能优化（一）之启动加速35%](https://juejin.im/post/5874bff0128fe1006b443fa0)


