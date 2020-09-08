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
### ContentProvider的启动过程
#### query方法到AMS的调用过程
![-w661](media/15991134504761.jpg)
AMS启动ContentProvider的过程
![-w640](media/15991135083785.jpg)
## 理解上下文Context
### Context的关联类
Context是一个抽象类，它的具体实现类为ContextImpl
![-w427](media/15991138683766.jpg)
Context的关联类采用了装饰模式，优点如下：
- 使用者（比如Service）能够更方便地使用Context
- 如果ContextImpl发生了变化，它的装饰类ContextWrapper不需要做任何修改
- 通过组合而非继承的方式，拓展ContextImpl的功能，在运行时选择不同的装饰类，实现不同的功能
### Application Context的创建过程
![-w600](media/15991143567388.jpg)
### Application Context的获取过程
ContextImpl的getApplicationContext()
### Activity的Context创建过程
![-w667](media/15991151925648.jpg)
在启动Activity的过程中创建ContextImpl，并赋值给ContextWrapper的成员变量mBase。Activity继承自ContextWrapper的子类ContextThemeWrapper，这样在Activity中就可以使用Context中定义的方法了。
### Service的Context创建过程
和Activity的类似
## 理解ActivityManagerService
### AMS家族
#### Android7.0的AMS家族
ActivityManager是关联类
通过ActivityManagerNative（AMN）得到AMP
ActivityManagerProxy（AMP）是AMS的代理类，也是AMN的内部类，实现了IActivityManager接口
AMP和AMN运行在两个进程，AMP是Client端，AMN是Server端
AMN实现了Binder类
![-w712](media/15991235169727.jpg)
![-w713](media/15991235631437.jpg)
#### Android8.0的AMS家族
Activity的启动过程会调用Instrumentation的execStartActivity方法
![-w676](media/15991237594984.jpg)
ActivityManager的getService方法
![-w679](media/15991238864926.jpg)
![-w582](media/15991241818171.jpg)
### AMS的启动过程
SystemServer中通过SystemServiceManager启动
### AMS与应用程序进程
启动应用程序时AMS会检查这个应用程序需要的进程是否存在
如果不存在，AMS就会请求Zygote进程创建需要的应用程序的进程
### AMS重要的数据结构
#### 解析ActivityRecord
内部记录了Activity的所有信息
![-w710](media/15991248210883.jpg)
#### 解析TaskRecord
描述一个Activity任务栈
![-w717](media/15991251295607.jpg)
![-w711](media/15991251464106.jpg)
#### 解析ActivityStack
ActivityStack是一个管理类，用来管理系统所有Activity
### Activity栈管理
#### Activity任务栈模型
![-w315](media/15991254538678.jpg)
#### Launch Mode
standerd
singleTop  onNewIntent
singleTask
singleInstance  创建一个新栈
#### Intent的FLAG
如果和Launch Mode冲突，则以FLAG为准
#### taskAffinity
设置栈
## 理解WindowManager
### Window、WindowManager和WMS
Window的实现类是PhoneWindow，对View进行管理
WindowManager是一个接口类，继承自接口ViewManager，实现类是WindowManagerImpl
WindowManager和WMS通过Binder进行跨进程通信
![-w352](media/15991822643936.jpg)
### WindowManager的关联类
![-w634](media/15991823384441.jpg)
Window是以View的形式存在
WindowManagerGlobal 桥接模式
![-w677](media/15991826829495.jpg)
### Window的属性
Type
Flag
SoftInputMode
#### Window的类型和显示次序
1.应用程序窗口
Activity就是典型
![-w569](media/15991828694246.jpg)
Type值1~99，窗口层级
2.子窗口
不能独立存在，需要附着在其他窗口，比如PopupWindow
![-w684](media/15991829507516.jpg)
Type值1000~1999
3.系统窗口
Toast、输入法窗口、系统音量条窗口、系统错误窗口
![-w680](media/15991830072313.jpg)
Type值2000~2999
4.窗口显示次序
Type值越大，在越上面
#### Window的标志
![-w714](media/15991831067962.jpg)
#### 软键盘相关模式
![-w712](media/15991832217160.jpg)
### Window的操作
![-w337](media/15991839528068.jpg)
#### 系统窗口的添加过程
addView >>
WindowManagerGlobal 维护3个列表
ViewRootImpl功能：
- View树的根并管理View树
- 触发View的测量、布局和绘制
- 输入事件的中转站
- 管理Surface
- 负责与WMS进行进程间通信
![-w641](media/15991857126771.jpg)
![-w709](media/15991858014804.jpg)
#### Activity的添加过程
ActivityThread的handleResumeActivity方法
    wm.addView(decor, l);
#### Window的更新过程
updateViewLayout
## 理解WindowManagerService
### WMS的职责
Android中的重要服务，WindowManager的管理者
1.窗口管理
2.窗口动画
3.输入系统的中转站
InputManagerService（IMS），对触摸事件进行处理，寻找最合适的窗口来处理
4.Surface管理
绘制
![-w647](media/15992079256553.jpg)
### WMS的创建过程
SystemServer >> startOtherServices
DisplayThread
![-w455](media/15992094761264.jpg)
### WMS的重要成员
![-w675](media/15992096187370.jpg)
### Window的添加过程（WMS处理部分）
WMS>>addWindow
### Window的删除过程
## JNI原理
### 系统源码中的JNI
![-w149](media/15992106181525.jpg)
## 10 Java虚拟机
![-w550](media/15994443766751.jpg)
![-w475](media/15994444014376.jpg)
## 11 Dalvik和ART
### Dalvik虚拟机
#### DVM和JVM的区别
1.基于的架构不同
2.执行的字节码不同
JVM执行顺序：.java>>.class>>.jar
DVM执行顺序：.java>>.class>>.dex
![-w428](media/15994451060388.jpg)
3.DVM允许在有限的内存中同时运行多个进程
Android中每一个应用程序都运行在一个DVM实例中，每一个DVM实例都运行在一个独立的进程空间中
4.DVM由Zygote创建和初始化
5.DVM有共享机制
不同应用之间在运行时可以共享相同的类，拥有更高的效率
6.DVM早期没有使用JIT编译器
### ART虚拟机
Android5.0以后默认采用ART
#### ART与DVM的区别
1.AOT（预编译），应用安装时就编译
缺点：一是安装时间变长，二是存储空间变多
2.DVM是32位，ART支持64位
3.ART对垃圾回收机制进行改进
4.运行时堆空间划分不同
## 12 理解ClassLoader
### Java中的ClassLoader
#### 双亲委托模式
![-w515](media/15994502066905.jpg)
### Android中的ClassLoader
加载dex文件
1.BootClassLoader
Java实现，系统预加载常用类
2.DexClassLoader
可以加载dex以及包含dex的压缩文件（apk和jar）
3.PathClassLoader
加载系统类和应用程序的类，通常用于安装的apk的dex文件
![-w586](media/15994603242795.jpg)
![-w569](media/15994614462384.jpg)
#### BootClassLoader的创建
Zygote进程>>ZygoteInit
#### PathClassLoader
Zygote进程启动SystemServer进程
## 13 热修复原理
![-w712](media/15994691081633.jpg)
### 资源修复
#### Instant Run概述
![-w544](media/15994691698226.jpg)
![-w674](media/15994692006786.jpg)
- Hot Swap：不需要重启应用和Activity，修改一个现有方法中的代码
- Warm Swap：不重启App，重启Activity，修改或删除一个现有资源文件
- Cold Swap：App需要重启，但不需要重新安装。
1.创建新的AssetManager，通过反射调用addAssetPath方法加载外部资源，这样新创建的AssetManager就含有了外部资源。
2.将AssetManager类型的mAssets字段的引用全部替换为新创建的AssetManager。
### 代码修复
#### 类加载方案
Dex分包
放在dexElements的第一个元素
![-w573](media/15994783164431.jpg)
需要重启App后让ClassLoader重新加载新的类
#### 底层替换方案
替换ArtMethod，不需重启
#### Instant Run 方案