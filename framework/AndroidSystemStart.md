# Android系统的启动流程
>Android有哪些主要的系统进程
>这些系统进程怎么启动
>进程启动之后主要做了什么

## 系统进程
init.rc
zygote，servicemanager，media...
SystemServer
### Zygote启动
1.init fork
2.启动虚拟机，注册JNI
3.预加载资源
4.启动SystemServer
5.进入Socket Loop
### SystemServer怎么启动
Zygote fork
初始化--binder--SystemServer.main()
>可以认为是一个应用
主线程进入Loop循环
## 系统服务是怎么启动的
系统服务怎么发布，让应用可见
publishBinderServece注册到ServiceManager
系统服务跑在什么线程
## 问题
1、为什么系统服务不都跑在Binder线程里呢？
binder线程是大家一起共享的，如果系统负载很重，binder线程池忙碌，可能影响系统服务响应的实时性，另外如果任务太耗时，长时间占用binder线程也不妥。
2、为什么系统服务不都跑在自己私有的工作线程里呢？
不可能每个服务都启动一个工作线程，一共上百个系统服务，线程开太多了会内存溢出的。而且太多线程之间切换对性能不利。
3、跑在Binder线程和跑在工作线程，如何取舍？
总的来说，对于实时性要求不那么高，并且处理起来不太耗时的任务可以就放在binder线程里。另外启动工作线程也可以避免同步的问题，因为应用程序跨进程调用过来是在binder线程池，通过切到工作线程可以让binder调用序列化，不用到处上锁。
## 怎么解决系统服务的相互依赖
1.分批启动 AMS，PMS，PKMS
2.分阶段启动 阶段1，阶段2，阶段3
## 桌面的启动
看成一个单独的系统级应用
## 技巧点拨
条理清晰--what，how，why
结论+细节
有技巧的引导，掌握主动权
