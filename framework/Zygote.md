# Zygote进程
>谈谈你对Zygote的理解

## what：
Zygote的作用
1.启动SystemServer
2.孵化其它进程
## how：
Zygote的启动流程
启动三段式->进程启动--准备工作--LOOP（开启循环，接收消息）
1.怎么启动
init进程加载配置文件，启动系统服务（其中就包含zygote），fork
2.启动之后做了什么
2.1 Native
启动虚拟机--注册Android的JNI函数--进入Java世界
2.2 Java
加载系统资源--SystemServer--LOOP（socket消息）
zygote fork进程，执行ActivityThread的main函数
> 注意细节
Zygote fork 要单线程
Zygote的IPC没有使用binder
## why
Zygote工作原理
怎么启动进程，怎么通信
## 其它问题
孵化应用进程这种事为什么不交给SystemServer来做，而专门设计一个Zygote？
我们知道，应用在启动的时候需要做很多准备工作，包括启动虚拟机，加载各类系统资源等等，这些都是非常耗时的，如果能在zygote里就给这些必要的初始化工作做好，子进程在fork的时候就能直接共享，那么这样的话效率就会非常高。这个就是zygote存在的价值，这一点呢systemServer是替代不了的，主要是因为systemServer里跑了一堆系统服务，这些是不能继承到应用进程的。而且我们应用进程在启动的时候，内存空间除了必要的资源外，最好是干干净净的，不要继承一堆乱七八糟的东西。所以呢，不如给systemServer和应用进程里都要用到的资源抽出来单独放在一个进程里，也就是这的zygote进程，然后zygote进程再分别孵化出systemServer进程和应用进程。孵化出来之后，systemServer进程和应用进程就可以各干各的事了。
Zygote的IPC通信机制为什么不采用binder？如果采用binder的话会有什么问题么？
关于为什么不采用binder，有两个原因，
先看第一个原因，我们可以设想一下采用binder调用的话该怎么做，首先zygote要启用binder机制，需要打开binder驱动，获得一个描述符，再通过mmap进行内存映射，还要注册binder线程，这还不够，还要创建一个binder对象注册到serviceManager，另外AMS要向zygote发起创建应用进程请求的话，要先从serviceManager查询zygote的binder对象，然后再发起binder调用，这来来回回好几趟非常繁琐，相比之下，zygote和systemServer进程本来就是父子关系，对于简单的消息通信，用管道或者socket非常方便省事，如果对管道和socket不了解的话，可以参考APUE和UNP这两本书。
再看第二个原因，如果zygote启用binder机制，再fork出systemServer，那么systemServer就会继承了zygote的描述符以及映射的内存，这两个进程在binder驱动层就会共用一套数据结构，这显然是不行的，所以还得先给原来的旧的描述符关掉，再重新启用一遍binder机制，这个就是自找麻烦了。
总的来说呢，对于轻量级的跨进程消息通信，没必要杀鸡用牛刀，普通的管道或者socket足矣。

