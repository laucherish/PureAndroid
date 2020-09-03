# Android进阶解密
## Android系统启动
### init进程启动
启动电源和系统>>引导程序BootLoader>>Linux内核>>init进程
init启动Zygote fork（）函数
init进程启动工作总结
- 创建和挂载启动所需要的目录
- 初始化和启动属性服务
- 解析init.rc配置文件并启动Zygote进程
### Zygote进程启动（孵化器）
- 启动Zygote进程
- 创建Java虚拟机并注册JNI方法
- 通过JNI调用进入Zygote的Java框架层
- 创建服务端Socket，进入Loop循环，等待AMS的请求来创建新的应用程序进程
- 启动SystemServer进程
### SystemServer
主要用于创建系统服务
SystemServer进程被创建后，主要做了如下工作
- 启动Binder线程池，这样可以和其它进程通信
- 创建SystemServiceManager，用于对系统的服务进行创建、启动和生命周期管理
- 启动各种系统服务
### Launcher启动过程
Launcher就是Android系统的桌面，作用主要如下：
- Android系统的启动器，用于启动应用
- Android系统的桌面，用于显示和管理应用程序的快捷图标或其他桌面组件
## 应用程序进程启动过程
AMS在启动应用程序的时候会检查这个应用程序的进程是否存在，不存在就会请求Zygote进程启动需要的应用程序进程
### AMS发送启动应用程序请求
![-w557](media/15990289459430.jpg)
通过socket与Zygote进程建立连接
### Zygote接收请求并创建应用程序进程
![-w611](media/15990297623789.jpg)
建立socket连接之后
Zygote获取应用程序进程的启动参数
fork 创建应用程序进程
通过反射调用ActivityThread（主线程的管理类）的main方法
### Binder线程池启动过程
应用程序创建过程中会启动Binder线程池，创建线程池中的第一个线程（线程池的主线程），将当前线程注册到Binder驱动程序中，我们只需要创建当前进程的Binder对象，并将它注册到ServiceManager中就可以实现Binder进程间通信
### 消息循环创建过程
ActivityThread>>main>>Looper
## 四大组件的工作过程
时序图
### 根Activity的启动过程
#### Launcher请求AMS过程
![-w670](media/15990329055507.jpg)
Instrumentation主要用来监控应用程序和系统的交互
IActivityManager采用AIDL，和AMS通过Binder进行IPC
#### AMS到ApplicationThread的调用过程
![-w638](media/15990331537523.jpg)
#### ActivityThread启动Activity的过程
目前代码逻辑运行在应用程序进程中
![-w644](media/15990367318345.jpg)
#### 根Activity启动过程中涉及的进程
![-w480](media/15990378023475.jpg)
![-w519](media/15990380658741.jpg)
### Service的启动过程
#### ContextImpl到AMS的调用过程
![-w576](media/15990400669255.jpg)
#### ActivityThread启动Service
![-w582](media/15990401605300.jpg)
最终调用ActivityThread的handleCreateService
### Service的绑定过程
#### ContextImpl到AMS的调用过程
![-w502](media/15990972334241.jpg)
#### Service的绑定过程
![-w575](media/15990975160568.jpg)
![-w694](media/15990975358332.jpg)
### 广播的注册、发送和接收过程
#### 广播的注册过程
![-w564](media/15990976002346.jpg)
IIntentReceiver是一个Binder接口
AMS中取出粘性广播列表，遍历寻找匹配的粘性广播
将BroadcastFilter存入AMS的ReceiverList和mReceiverResolver
#### 广播的发送和接收过程
##### ContextImpl到AMS的调用过程
![-w527](media/15990986581140.jpg)
AMS首先验证广播是否合法
##### AMS到BroadcastReceiver的调用过程
![-w719](media/15990989237747.jpg)

