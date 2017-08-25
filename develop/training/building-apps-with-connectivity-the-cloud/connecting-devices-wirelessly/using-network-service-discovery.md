# 使用网络服务发现

网络服务发现（NSD）提供了应用可以在本地网络上访问其他设备的服务。设备支持的 NSD 包括打印机、网络摄像头、HTTPS 服务器和其他移动设备。


NSD 实现了基于 DNS 服务的发现机制（DNS-SD），允许应用程序通过指定类型的服务和指定名称的设备实例去请求服务，后者提供了必要的服务类型。DNS-SD 支持 Android 平台和其他移动平台。

在应用中添加 NSD，可以让用户在同一个网络中辨别出支持应用请求服务的其他设备。这对于各种点对点的应用非常有用，例如文件共享或者多人联机游戏。Android NSD 的 API 可以简化实现这些功能的难度。

这节课主要介绍如何构建一个应用，使其可以在局域网内广播名字和连接信息，并且扫描其它正在做同样事情的应用信息。最后，将介绍如果连接运行在另一台设备上的相同应用。



## 在网络中注册 NSD 服务
>**注:** 这一步骤是可选的。如果我们并不关心在本地网络上广播 app 服务，那么我们可以跳过这一节内容，直接进入 [Discover Services on the Network](https://developer.android.google.cn/training/connect-devices-wirelessly/nsd.html#discover)。

在本地网络注册服务，首先需要创建一个 [NsdServiceInfo](https://developer.android.google.cn/reference/android/net/nsd/NsdServiceInfo.html)  对象。该对象提供了网络上其他设备在决定是否连接到我们的服务时所使用的信息。

```java
public void registerService(int port) {
    // Create the NsdServiceInfo object, and populate it.
    NsdServiceInfo serviceInfo  = new NsdServiceInfo();

    // The name is subject to change based on conflicts
    // with other services advertised on the same network.
    serviceInfo.setServiceName("NsdChat");
    serviceInfo.setServiceType("nsdchat._tcp");
    serviceInfo.setPort(port);
    ....
}
```

这段代码设置服务名称为"NsdChat"。这个服务名称是一个实例名，对网络中的其他设备是可见的。同时该名称将对所有局域网络中使用 NSD 查找本地服务的设备可见。需要注意的是，在网络中该名称是唯一的且 Android 会自动处理名称冲突。在网络中，如果有二个设备同时拥有名称为 “NsdChat” 的应用，其中一个会自动改变服务名称，比如"NsdChat (1)"。

第二个参数设置了服务类型，指定应用使用的协议和传输层。语法是“_< protocol >._< transportlayer >”。在这段代码中，服务使用了基于 TCP 的 HTTP 协议。如果应用需要提供打印服务（比如网络打印机），那么应该设置服务类型为"_ipp._tcp"。


>**注:** 互联网数字分配机构（IANA）提供了用于服务发现协议（如 NSD 和 Bonjour）的官网服务类型列表。我们可以从 [the IANA list of service names and port numbers](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xml) 下载此列表。如果想要使用新的服务类型，我们应该在 [IANA Ports and Service registration form](https://www.iana.org/form/ports-services) 中填写申请表。

当设置服务端口时，需要避免硬编码，以防止与其他应用的冲突。例如，如果应用一直使用 1337 端口，就会与其他使用相同端口的应用存在潜在的冲突。因为信息是通过广播提供给其他应用的，在编译过程中是不需要让其他应用知道你的应用端口的。其他应用可以通过服务广播获取到信息，然后连接到我们的服务。

如果使用的是 socket，那么我们可以简单地将端口设置为 0，来初始化 socket 到任意可用的端口。

```java
public void initializeServerSocket() {  
    // Initialize a server socket on the next available port.  
    mServerSocket = new ServerSocket(0);  
  
    // Store the chosen port.  
    mLocalPort =  mServerSocket.getLocalPort();  
    ...  
}  
```

现在我们已经定义了 [NsdServiceInfo](https://developer.android.google.cn/reference/android/net/nsd/NsdServiceInfo.html) 对象，我们还需要实现 [RegistrationListener](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.RegistrationListener.html) 接口。该接口包含了在 Android 中注册的回调函数，用于服务注册和注销的成功或者失败的提醒回调。

```java
public void initializeRegistrationListener() {
    mRegistrationListener = new NsdManager.RegistrationListener() {

        @Override
        public void onServiceRegistered(NsdServiceInfo NsdServiceInfo) {
            // Save the service name.  Android may have changed it in order to
            // resolve a conflict, so update the name you initially requested
            // with the name Android actually used.
            mServiceName = NsdServiceInfo.getServiceName();
        }

        @Override
        public void onRegistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Registration failed!  Put debugging code here to determine why.
        }

        @Override
        public void onServiceUnregistered(NsdServiceInfo arg0) {
            // Service has been unregistered.  This only happens when you call
            // NsdManager.unregisterService() and pass in this listener.
        }

        @Override
        public void onUnregistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Unregistration failed.  Put debugging code here to determine why.
        }
    };
}
```
现在已经拥有注册服务的所有信息了。接下来调用方法 [registerService()](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.html#registerService(android.net.nsd.NsdServiceInfo, int, android.net.nsd.NsdManager.RegistrationListener))。

需要注意这个方法是异步的，所以后续的代码都需要在服务注册后的 [onServiceRegistered()](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.RegistrationListener.html#onServiceRegistered(android.net.nsd.NsdServiceInfo)) 回调方法中去运行。

```java
public void registerService(int port) {
    NsdServiceInfo serviceInfo  = new NsdServiceInfo();
    serviceInfo.setServiceName("NsdChat");
    serviceInfo.setServiceType("_http._tcp.");
    serviceInfo.setPort(port);

    mNsdManager = Context.getSystemService(Context.NSD_SERVICE);

    mNsdManager.registerService(
            serviceInfo, NsdManager.PROTOCOL_DNS_SD, mRegistrationListener);
}
```
##在网络中发现服务

生活中网络无处不在，从嘈杂的网络打印机到安静的网络摄影机，以及附近激烈的井字棋游戏。网络服务发现是让我们的应用可以融入这些充满活力的生态系统功能的关键。应用需要在网络中监听服务广播以便获知哪些服务是可用的，并且过滤无效的信息。

服务发现，就像服务注册一样，有二个步骤：设置发现监听的相关回调，并且调用 [discoverServices()](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.html#discoverServices(java.lang.String, int, android.net.nsd.NsdManager.DiscoveryListener)) 这个异步 API 。

首先，实例化一个实现 [NsdManager.DiscoveryListener](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.DiscoveryListener.html) 接口的匿名类。下面的代码段是一个简单的例子：

```java
public void initializeDiscoveryListener() {

    // Instantiate a new DiscoveryListener
    mDiscoveryListener = new NsdManager.DiscoveryListener() {

        //  Called as soon as service discovery begins.
        @Override
        public void onDiscoveryStarted(String regType) {
            Log.d(TAG, "Service discovery started");
        }

        @Override
        public void onServiceFound(NsdServiceInfo service) {
            // A service was found!  Do something with it.
            Log.d(TAG, "Service discovery success" + service);
            if (!service.getServiceType().equals(SERVICE_TYPE)) {
                // Service type is the string containing the protocol and
                // transport layer for this service.
                Log.d(TAG, "Unknown Service Type: " + service.getServiceType());
            } else if (service.getServiceName().equals(mServiceName)) {
                // The name of the service tells the user what they'd be
                // connecting to. It could be "Bob's Chat App".
                Log.d(TAG, "Same machine: " + mServiceName);
            } else if (service.getServiceName().contains("NsdChat")){
                mNsdManager.resolveService(service, mResolveListener);
            }
        }

        @Override
        public void onServiceLost(NsdServiceInfo service) {
            // When the network service is no longer available.
            // Internal bookkeeping code goes here.
            Log.e(TAG, "service lost" + service);
        }

        @Override
        public void onDiscoveryStopped(String serviceType) {
            Log.i(TAG, "Discovery stopped: " + serviceType);
        }

        @Override
        public void onStartDiscoveryFailed(String serviceType, int errorCode) {
            Log.e(TAG, "Discovery failed: Error code:" + errorCode);
            mNsdManager.stopServiceDiscovery(this);
        }

        @Override
        public void onStopDiscoveryFailed(String serviceType, int errorCode) {
            Log.e(TAG, "Discovery failed: Error code:" + errorCode);
            mNsdManager.stopServiceDiscovery(this);
        }
    };
}
```

NSD API 通过使用该接口中的方法通知应用程序搜索何时开始、何时失败以及何时找到可用服务和何时服务丢失（丢失意味着“不再可用”）。请注意当发现了可用服务时，上述代码程序做了数次校对。    

1.对比搜索到的服务名称和本地服务的名称，验证设备是否获得自己的广播（是否合法）。    
2.核对设备类型，确保服务类型是应用可以连接的。    
3.核对服务名称，确保连接正确的应用。    

有时候我们不必去核对服务名称，只需要在连接的具体应用进行检查。例如，应用也许只是想与运行在其他设备上的相同应用进行连接。然而，如果应用想要连接网络打印机，只需要确认服务类型为”_ipp._tcp"就可以了。

设置监听后，调用 [discoverServices()](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.html#discoverServices(java.lang.String, int, android.net.nsd.NsdManager.DiscoveryListener))，并传递相应的参数：应用需要的服务类型、使用的发现协议以及已经创建的监听器。


```java
 mNsdManager.discoverServices(
        SERVICE_TYPE, NsdManager.PROTOCOL_DNS_SD, mDiscoveryListener);
```
## 在网络中连接服务
当应用在网络中发现可以连接的服务时，首先需要调用 [resolveService()](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.html#resolveService(android.net.nsd.NsdServiceInfo, android.net.nsd.NsdManager.ResolveListener)) 来确认服务的连接信息，实现[NsdManager.ResolveListener](https://developer.android.google.cn/reference/android/net/nsd/NsdManager.ResolveListener.html) 接口以传递这个方法，继而获取包含连接信息的 [NsdServiceInfo](https://developer.android.google.cn/reference/android/net/nsd/NsdServiceInfo.html)。

```java
public void initializeResolveListener() {
    mResolveListener = new NsdManager.ResolveListener() {

        @Override
        public void onResolveFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Called when the resolve fails.  Use the error code to debug.
            Log.e(TAG, "Resolve failed" + errorCode);
        }

        @Override
        public void onServiceResolved(NsdServiceInfo serviceInfo) {
            Log.e(TAG, "Resolve Succeeded. " + serviceInfo);

            if (serviceInfo.getServiceName().equals(mServiceName)) {
                Log.d(TAG, "Same IP.");
                return;
            }
            mService = serviceInfo;
            int port = mService.getPort();
            InetAddress host = mService.getHost();
        }
    };
}
```
当服务解析完成后，应用会接收到包含 IP 地址和端口号的具体服务信息。这就是创建网络连接其他服务的所有信息了。
## 应用退出时注销服务
在应用的生命周期中适时的开启和关闭 NSD 功能是很重要的。在应用关闭时注销 NSD，可以防止其他应用认为服务还是一直存在而去尝试连接它。另外，服务发现是一个比较消耗性能的操作，当 Activity 处理暂停时应该停止，而当 Activity 恢复时再重新打开。重写主 Activity 生命周期的方法，然后在适当的时机添加开启、停止服务广播和发现的代码。


```java

//In your application's Activity
@Override
    protected void onPause() {
        if (mNsdHelper != null) {
            mNsdHelper.tearDown();
        }
        super.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (mNsdHelper != null) {
            mNsdHelper.registerService(mConnection.getLocalPort());
            mNsdHelper.discoverServices();
        }
    }

    @Override
    protected void onDestroy() {
        mNsdHelper.tearDown();
        mConnection.tearDown();
        super.onDestroy();
    }

    // NsdHelper's tearDown method
        public void tearDown() {
        mNsdManager.unregisterService(mRegistrationListener);
        mNsdManager.stopServiceDiscovery(mDiscoveryListener);
    }

```    

>**备注**  
翻译：[@misparking](https://github.com/misparking)    
原始文档:  [https://developer.android.google.cn/training/connect-devices-wirelessly/nsd.html)
