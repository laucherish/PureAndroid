> 源码版本3.14.9，即Java版本的最新版，4.0版本以上使用Kotlin
> [源码地址](https://github.com/square/okhttp/tree/parent-3.14.9)
## 1.使用方法
引入依赖
`implementation 'com.squareup.okhttp3:okhttp:3.14.9'
`
异步网络请求（项目中通常使用异步请求方法，Android要求必须在子线程中执行网络请求）

```java
// 1.创建OkHttpClient，相当于一个管家
OkHttpClient client = new OkHttpClient.Builder().build();
// 2.创建Request
Request request = new Request.Builder().url("https://github.com/").build();
// 3.根据Request创建一个Call对象
Call call = client.newCall(request);
// 4.执行Call的enqueue，此为异步方法，不会阻塞线程，在回调onResponse方法里面得到响应结果
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        // 请求失败的回调
    }
    @Override
    public void onResponse(Call call, Response response) throws IOException {
        // 回调里面得到响应结果response，需要注意此时是在子线程，如果需要更新UI必须切回主线程
    }
});
```
这就是OkHttp的常见用法
* 一，创建OkHttpClient，这个是整个OkHttp的核心管理类，所有的内部逻辑和对象由OkHttpClient统一管理，它通过Builder构造器生成，构造参数和类成员很多，此时可以对OkHttp进行配置。
* 二，创建Request，Request是我们发送请求封装类，对应Http协议中的请求报文，内部包含请求方法method，请求地址method，请求头headers，请求体body，符合Http协议所定义的请求内容。
* 三，创建Call，源码里描述Call是准备好执行的一个request，它可以被取消，它可以表示一个request/response对，只能被执行一次。它的实现类是RealCall，虽然OkHttpClient是整个OkHttp的核心管理类，但是真正发出请求并且组织逻辑的是RealCall类，它同时肩负了调度和责任链组织的两大重任。
* 四，得到Response，Call调用同步execute或者异步enqueue方法得到请求响应结果Response，Response是请求响应封装类，内部包含响应码code，响应消息message，响应头header，响应体body，还有一些其它信息，符合Http协议所定义的响应内容。
## 2.整体流程分析
先给出一张OkHttp的整体流程图，下面我们具体分析：
![OkHttp流程图](media/16037840704075.jpg)
OkHttpClient和Request都可以使用Builder构建模式创建，从OkHttpClient的newCall方法开始分析，其调用了RealCall的newRealCall方法。Call是一个接口，
RealCall是其实现类。

```java
// OkHttpClient.java
Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false);
}

// RealCall.java
RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    // Transmitter翻译为传达者，主要用于更新状态信息
    call.transmitter = new Transmitter(client, call);
    return call;
}
```
接下来执行RealCall的enqueue方法

```java
// RealCall.java
void enqueue(Callback responseCallback) {
    //回调eventListener，实时汇报状态，先忽略
    transmitter.callStart();
    //用AsyncCall封装Callback，由调度中心dispatcher安排进入执行队列
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
}
```
内部调用了Dispatcher的enqueue方法，并创建了一个AsyncCall作为参数。

```java
//Dispatcher.java
int maxRequests = 64;  // 最多同时请求数为64
int maxRequestsPerHost = 5;  // 每个主机最多同时请求数为5

enqueue(AsyncCall call) {
    synchronized (this) {
        // 加入异步请求准备队列
        readyAsyncCalls.add(call);
        if (!call.get().forWebSocket) {
            //查找和call请求相同host的AsyncCall
            AsyncCall existingCall = findExistingCallWithHost(call.host());
            //复用此host的计数器callsPerHost，用于统计同一个host的请求数
            if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
        }
    }
    // 如果满足条件，就会执行请求
    promoteAndExecute();
}

boolean promoteAndExecute() {
    // 收集可以执行的AsyncCall
    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
        for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
            AsyncCall asyncCall = i.next();
            // 如果正在执行的请求数超过最大值，则跳出
            if (runningAsyncCalls.size() >= maxRequests) break;
            // 如果请求同一个主机的请求数超过允许的最大值，则跳出
            if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue;
            // 从准备队列里移除
            i.remove();
            // 同一个主机的请求数量+1
            asyncCall.callsPerHost().incrementAndGet();
            // 把AsyncCall存起来
            executableCalls.add(asyncCall);
            // 加入执行请求队列
            runningAsyncCalls.add(asyncCall);
        }
        isRunning = runningCallsCount() > 0;
    }
    // 遍历并执行可执行队列的AsyncCall
    for (int i = 0, size = executableCalls.size(); i < size; i++) {
        AsyncCall asyncCall = executableCalls.get(i);
        asyncCall.executeOn(executorService());
    }
    return isRunning;
}
```