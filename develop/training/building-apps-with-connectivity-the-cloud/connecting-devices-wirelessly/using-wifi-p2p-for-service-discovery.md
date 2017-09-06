#使用Wifi P2P进行服务搜索

本阶段的第一课[《使用网络服务搜索》](https://developer.android.google.cn/training/connect-devices-wirelessly/nsd.html)，展示如何搜索连接了本地网络的服务。然而使用 WiFi 对等网络（P2P）搜索服务可以直接搜索到附近设备的服务，而不需要连接网络。你可以让该服务运行在手机。这些功能可以帮助我们在两个应用间通讯，即使没有有效的网络或热点。

> 尽管这一套 API 与之前课程《网络服务搜索》 API 大纲目的类似，但在代码实现还是有很大不同。这节课解说使用 WiFi P2P 如何从其它设备中搜索有效服务。这节课假定你已经熟悉了 [WiFi P2P](https://developer.android.google.cn/guide/topics/connectivity/wifip2p.html) API。


### 设置清单文件

使用 WiFi P2P 时需要添加权限 [CHANGE_WIFI_STATE](https://developer.android.google.cn/reference/android/Manifest.permission.html#CHANGE_WIFI_STATE), [ACCESS_WIFI_STATE](https://developer.android.google.cn/reference/android/Manifest.permission.html#ACCESS_WIFI_STATE), 和 [INTERNET](https://developer.android.google.cn/reference/android/Manifest.permission.html#INTERNET) 到清单文件。虽然 WiFi P2P 使用标准 Java Socket，并不需要有互联网连接，但是在 Android 中使用这些时需要请求权限。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.nsdchat"
    ...

    <uses-permission
        android:required="true"
        android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission
        android:required="true"
        android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission
        android:required="true"
        android:name="android.permission.INTERNET"/>
    ...
```


### 添加本地服务

如果提供本地服务，需要为它注册服务搜索。一旦本地服务注册后，Framework 会自动响应服务搜索请求。

创建本地服务：

1. 创建 [WifiP2pServiceInfo](https://developer.android.google.cn/reference/android/net/wifi/p2p/nsd/WifiP2pServiceInfo.html) 对象
2. 配置关于服务的信息
3. 调用 [addLocalService()](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html#addLocalService(android.net.wifi.p2p.WifiP2pManager.Channel,android.net.wifi.p2p.nsd.WifiP2pServiceInfo,android.net.wifi.p2p.WifiP2pManager.ActionListener)) 方法注册本地服务

```java
private void startRegistration() {
    //  Create a string map containing information about your service.
    Map record = new HashMap();
    record.put("listenport", String.valueOf(SERVER_PORT));
    record.put("buddyname", "John Doe" + (int) (Math.random() * 1000));
    record.put("available", "visible");

    // Service information.  Pass it an instance name, service type
    // _protocol._transportlayer , and the map containing
    // information other devices will want once they connect to this one.
    WifiP2pDnsSdServiceInfo serviceInfo =
            WifiP2pDnsSdServiceInfo.newInstance("_test", "_presence._tcp", record);

    // Add the local service, sending the service info, network channel,
    // and listener that will be used to indicate success or failure of
    // the request.
    mManager.addLocalService(channel, serviceInfo, new ActionListener() {
        @Override
        public void onSuccess() {
            // Command successful! Code isn't necessarily needed here,
            // Unless you want to update the UI or add logging statements.
        }

        @Override
        public void onFailure(int arg0) {
            // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
        }
    });
}
```

### 搜索附近服务

第一步应创建回调方法来通知应用中的有效服务。创建 [WifiP2pManager.DnsSdTxtRecordListener](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdTxtRecordListener.html) 对象来监听回调的消息，该消息由其它设备随意的广播。当收到一条消息时，拿到设备地址或其它任何你需要的相关信息以便后续使用。下面的例子采用消息的 “buddyname” 字段作为它的标识。

```java
final HashMap<String, String> buddies = new HashMap<String, String>();
...
private void discoverService() {
    DnsSdTxtRecordListener txtListener = new DnsSdTxtRecordListener() {
        @Override
        /* Callback includes:
         * fullDomain: full domain name: e.g "printer._ipp._tcp.local."
         * record: TXT record dta as a map of key/value pairs.
         * device: The device running the advertised service.
         */

        public void onDnsSdTxtRecordAvailable(
                String fullDomain, Map record, WifiP2pDevice device) {
                Log.d(TAG, "DnsSdTxtRecord available -" + record.toString());
                buddies.put(device.deviceAddress, record.get("buddyname"));
            }
        };
    ...
}
```

获取服务信息时，创建 [WifiP2pManager.DnsSdServiceResponseListener](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdServiceResponseListener.html) 对象，它接收实际的描述和连接信息。之前的代码块实现的 [Map](https://developer.android.google.cn/reference/java/util/Map.html) 对象匹配设备地址和名称。服务返回 DNS 记录和相关的服务信息。当完成这两个监听的实现后，使用 [setDnsSdResponseListeners()](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html#setDnsSdResponseListeners(android.net.wifi.p2p.WifiP2pManager.Channel,android.net.wifi.p2p.WifiP2pManager.DnsSdServiceResponseListener,android.net.wifi.p2p.WifiP2pManager.DnsSdTxtRecordListener)) 方法添加它们到 [WifiP2pManager](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html) 中。

```java
private void discoverService() {
...

    DnsSdServiceResponseListener servListener = new DnsSdServiceResponseListener() {
        @Override
        public void onDnsSdServiceAvailable(String instanceName, String registrationType,
                WifiP2pDevice resourceType) {

                // Update the device name with the human-friendly version from
                // the DnsTxtRecord, assuming one arrived.
                resourceType.deviceName = buddies
                        .containsKey(resourceType.deviceAddress) ? buddies
                        .get(resourceType.deviceAddress) : resourceType.deviceName;

                // Add to the custom adapter defined specifically for showing
                // wifi devices.
                WiFiDirectServicesList fragment = (WiFiDirectServicesList) getFragmentManager()
                        .findFragmentById(R.id.frag_peerlist);
                WiFiDevicesAdapter adapter = ((WiFiDevicesAdapter) fragment
                        .getListAdapter());

                adapter.add(resourceType);
                adapter.notifyDataSetChanged();
                Log.d(TAG, "onBonjourServiceAvailable " + instanceName);
        }
    };

    mManager.setDnsSdResponseListeners(channel, servListener, txtListener);
    ...
}
```


现在调用 [addServiceRequest()](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html#addServiceRequest(android.net.wifi.p2p.WifiP2pManager.Channel,android.net.wifi.p2p.nsd.WifiP2pServiceRequest,android.net.wifi.p2p.WifiP2pManager.ActionListener)) 方法创建服务请求，该方法接收一个监听来报告成功或失败。

```java
serviceRequest = WifiP2pDnsSdServiceRequest.newInstance();
mManager.addServiceRequest(channel,
        serviceRequest,
        new ActionListener() {
            @Override
            public void onSuccess() {
                // Success!
            }

            @Override
            public void onFailure(int code) {
                // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
            }
        });
```

最后调用 [discoverServices()](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverServices(android.net.wifi.p2p.WifiP2pManager.Channel,android.net.wifi.p2p.WifiP2pManager.ActionListener)) 方法。

```java
mManager.discoverServices(channel, new ActionListener() {

    @Override
    public void onSuccess() {
        // Success!
    }

    @Override
    public void onFailure(int code) {
        // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
        if (code == WifiP2pManager.P2P_UNSUPPORTED) {
            Log.d(TAG, "P2P isn't supported on this device.");
        else if(...)
            ...
    }
});
```

如果一切顺利，很好，已经完成了。如果遇到问题，记得检查 [WifiP2pManager.ActionListener](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html) 的异步监听，它提供了成功和失败的回调。如果回调 [onFailure()](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html#onFailure(int)) 方法，则可能是代码有问题。下面是可能的错误码和它们的意思：

1. [`P2P_UNSUPPORTED`](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html#P2P_UNSUPPORTED)   
	当前设备不支持 Wi-Fi P2P

2. [`BUSY`](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html#BUSY)   
	系统忙

3. [`ERROR`](https://developer.android.google.cn/reference/android/net/wifi/p2p/WifiP2pManager.html#ERROR)   
	内部错误，操作失败



>翻译：[@iOnesmile](https://github.com/iOnesmile)  
>审核人：[@JackWaiting](https://github.com/JackWaiting)     
原始文档：<https://developer.android.google.cn/training/connect-devices-wirelessly/nsd-wifi-direct.html#manifest>
