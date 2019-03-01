Service是Android四大组件中与Activity最相似的组件，它们都代表可执行的程序，Service与Activity的区别在于：Service一直在后台运行，
它没有用户界面，所以绝对不会到前台来。一旦Serivce被启动，它就与Activity一样。它完全具有自己的生命周期。

关于程序中Activity和Service的选择标准是：如果某个程序组件需要在运行时向用户呈现某种界面或者需要与用户交互，就需要使用Activity，
否则就应该考虑使用Service。

#### 1、Service启动方法与区别
  Service有2种启动方式：                
    a、通过Context的startService()方法：通过该方法启动的Service，访问者与Service之间没有关联，因此Service和访问者之间无法
    进行通讯和数据交换，而且即使访问者退出了，Service仍然运行。               
    b、通过Context的bindService()方法：使用该方法启动Service，访问者与Service绑定在了一起，访问者一旦退出，Service也就终止了，
    如果Service和访问者之间需要进行方法调用或数据交换，应选用此方式启动Service。
    
#### 2、Service的生命周期               
    a：startService 
    Service的生命周期：onCreate() --> onStart() -> onDestroy()
    停止服务：stopService()
    
    b：bindService
    Service的生命周期 onCreate() --> onBind()  --> onUnBind() --> onDestroy()
    停止服务：UnbindService()
