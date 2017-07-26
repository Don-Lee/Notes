导读：最近有点时间，所以整理了一下Android面试中经常遇到的一些问题，大部分都是互联网上收集的，很多原作者的链接没有了，非常抱歉，
如有问题可直接与我联系，thanks.

1、Service 和Thread 的区别
a)、Thread: thread是程序执行的最小单元，他是CPU的基本单位。可以用thread来执行一些异步操作。
b)、Service: service是android的一种机制，当它运行的时候如果是LocalService,那么对应的Service运行在主线程的main线程上。
如果是RemoteService,那么对应的service则运行在独立进程的main线程上。
