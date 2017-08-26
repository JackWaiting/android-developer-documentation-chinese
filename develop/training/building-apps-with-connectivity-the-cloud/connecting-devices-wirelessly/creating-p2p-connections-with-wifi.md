# 使用Wi-Fi创建P2P连接

Wi-Fi P2P允许你的应用程序在远远超出蓝牙功能的范围内快速找到并与附近的设备进行交互。

Wi-Fi peer-to-peer（P2P）API 允许应用程序连接到附近的设备，而无需连接到网络或热点（Android 的 Wi-Fi P2P 框架符合 Wi-Fi Direct™ 认证计划）。 如果你的应用被设计为安全的近距离网络的一部分，Wi-Fi Direct 比传统的 Wi-Fi ad-hoc 网络更为合适，因为以下原因：

- Wi-Fi Direct 支持 WPA2 加密。 （某些 ad-hoc 网络仅支持WEP加密。）

- 设备可以广播他们提供的服务，这有助于其他设备更容易地发现合适的设备。

- 当确定哪个设备应该是网络的组所有者时，Wi-Fi Direct 将检查每个设备的电源管理，UI和服务功能，并使用此信息来选择最有效地处理服务器功能的设备。

- Android 不支持 Wi-Fi ad-hoc 模式。

本课程将向你展示如何使用 Wifi P2P 查找和连接附近的设备。

## 设置权限

为了使用 Wi-Fi P2P，请将 manifest 中的 ACCESS_COARSE_LOCATION ，CHANGE_WIFI_STATE，ACCESS_WIFI_STATE 和 INTERNET 权限添加到其中。 Wi-Fi P2P 不需要互联网连接，但它使用标准的 Java 套接字，这需要 INTERNET 权限。 所以你需要以下权限才能使用 Wi-Fi P2P：

	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	    package="com.example.android.nsdchat"
	    ...
	    <uses-permission
	        android:required="true"
	        android:name="android.permission.ACCESS_COARSE_LOCATION"/>
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

## 设置广播接收器和 P2P 管理器

要使用 Wi-Fi P2P 时，你需要监听某些事件并告知应用这个广播 Intent。 在你的应用中，实例化  IntentFilter 并将其设置为监听以下内容：

	WIFI_P2P_STATE_CHANGED_ACTION
	
		指示是否启用 Wi-Fi P2P

	WIFI_P2P_PEERS_CHANGED_ACTION

		表示可用的 P2P 列表已更改。

	WIFI_P2P_CONNECTION_CHANGED_ACTION

		表示 Wi-Fi P2P 的连接状态发生了变化。

	WIFI_P2P_THIS_DEVICE_CHANGED_ACTION

		表示此设备的配置详细信息已更改。

-

	private final IntentFilter intentFilter = new IntentFilter();
	...
	@Override
	public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.main);
	
	    //  Indicates a change in the Wi-Fi P2P status.
	    intentFilter.addAction(WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION);
	
	    // Indicates a change in the list of available peers.
	    intentFilter.addAction(WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION);
	
	    // Indicates the state of Wi-Fi P2P connectivity has changed.
	    intentFilter.addAction(WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION);
	
	    // Indicates this device's details have changed.
	    intentFilter.addAction(WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION);
	
	    ...
	}	

在 onCreate（） 方法的最后，获取一个 WifiP2pManager 的实例，并调用其 initialize（） 方法。 此方法返回一个 WifiP2pManager.Channel 对象，稍后你将使用该对象将应用连接到 Wi-Fi P2P 框架。

	Override
	
	Channel mChannel;
	
	public void onCreate(Bundle savedInstanceState) {
	    ....
	    mManager = (WifiP2pManager) getSystemService(Context.WIFI_P2P_SERVICE);
	    mChannel = mManager.initialize(this, getMainLooper(), null);
	}

现在创建一个新的 BroadcastReceiver 类，你将用于监听系统的 Wi-Fi P2P 状态的更改。 在onReceive（）方法中，添加一个条件来处理上面列出的每个 P2P 状态变化。

	@Override
	public void onReceive(Context context, Intent intent) {
		String action = intent.getAction();
	        if (WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION.equals(action)) {
	            // Determine if Wifi P2P mode is enabled or not, alert
	            // the Activity.
	            int state = intent.getIntExtra(WifiP2pManager.EXTRA_WIFI_STATE, -1);
	            if (state == WifiP2pManager.WIFI_P2P_STATE_ENABLED) {
	                activity.setIsWifiP2pEnabled(true);
	            } else {
	                activity.setIsWifiP2pEnabled(false);
	            }
	        } else if (WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION.equals(action)) {
	
	            // The peer list has changed!  We should probably do something about
	            // that.
	
	        } else if (WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION.equals(action)) {
	
	            // Connection state changed!  We should probably do something about
	            // that.
	
	        } else if (WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION.equals(action)) {
	            DeviceListFragment fragment = (DeviceListFragment) activity.getFragmentManager()
	                    .findFragmentById(R.id.frag_list);
	            fragment.updateThisDevice((WifiP2pDevice) intent.getParcelableExtra(
	                    WifiP2pManager.EXTRA_WIFI_P2P_DEVICE));
	
	        }        
	}

最后，当你的主要 Activiry 处于活动状态时，添加注册 intent filter （意向过滤器） 和   broadcast (广播接收者)的代码，并在活动暂停时取消注册。 执行此操作的最佳方法是 onResume（）和 onPause（）方法。

		/** register the BroadcastReceiver with the intent values to be matched */
	    @Override
	    public void onResume() {
	        super.onResume();
	        receiver = new WiFiDirectBroadcastReceiver(mManager, mChannel, this);
	        registerReceiver(receiver, intentFilter);
	    }
	
	    @Override
	    public void onPause() {
	        super.onPause();
	        unregisterReceiver(receiver);
	    }

## 启动 P2P 搜索
要开始使用 Wi-Fi P2P 搜索附近的设备，请调用 discoverPeers（）。 此方法采用以下参数：

- 当你初始化 P2P 的 mManager 时，你收到的 WifiP2pManager.Channel
 
- 使用系统调用成功和不成功搜索的方法来实现 WifiP2pManager.ActionListener。

		mManager.discoverPeers(mChannel, new WifiP2pManager.ActionListener() {
	
	        @Override
	        public void onSuccess() {
	            // Code for when the discovery initiation is successful goes here.
	            // No services have actually been discovered yet, so this method
	            // can often be left blank.  Code for peer discovery goes in the
	            // onReceive method, detailed below.
	        }
	
	        @Override
	        public void onFailure(int reasonCode) {
	            // Code for when the discovery initiation fails goes here.
	            // Alert the user that something went wrong.
	        }
		});

请记住，这只会启动 P2P 的搜索功能。 discoverPeers（）方法启动搜索过程，搜索到然后立即返回。 如果回调了 onSuccess()，系统将通知你。 此外，搜索将保持活动状态，直到建立连接或形成P2P 组。

## 获取 P2P List集合
现在写代码来获取和处理 P2P 列表。 首先实现 WifiP2pManager.PeerListListener 接口，它提供有关 Wi-Fi P2P 监听到的 P2P 信息。 此信息还允许你的应用确定何时加入或离开网络。 以下代码片段说明了与 P2P 相关的这些操作：

	private List<WifiP2pDevice> peers = new ArrayList<WifiP2pDevice>();
	    ...
	
	    private PeerListListener peerListListener = new PeerListListener() {
	        @Override
	        public void onPeersAvailable(WifiP2pDeviceList peerList) {
	
	            List<WifiP2pDevice> refreshedPeers = peerList.getDeviceList();
	            if (!refreshedPeers.equals(peers)) {
	                peers.clear();
	                peers.addAll(refreshedPeers);
	
	                // If an AdapterView is backed by this data, notify it
	                // of the change.  For instance, if you have a ListView of
	                // available peers, trigger an update.
	                ((WiFiPeerListAdapter) getListAdapter()).notifyDataSetChanged();
	
	                // Perform any other updates needed based on the new list of
	                // peers connected to the Wi-Fi P2P network.
	            }
	
	            if (peers.size() == 0) {
	                Log.d(WiFiDirectActivity.TAG, "No devices found");
	                return;
	            }
	        }
	    }

当接收到 WIFI_P2P_PEERS_CHANGED_ACTION 的 Intent 时，现在修改广播接收者的onReceive（）方法来调用requestPeers（）。 你需要以这种方式把这个 requestPeers() 传给接收者。 一种方法是将其作为参数发送给广播接收者的构造函数。

	public void onReceive(Context context, Intent intent) {
	    ...
	    else if (WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION.equals(action)) {
	
	        // Request available peers from the wifi p2p manager. This is an
	        // asynchronous call and the calling activity is notified with a
	        // callback on PeerListListener.onPeersAvailable()
	        if (mManager != null) {
	            mManager.requestPeers(mChannel, peerListListener);
	        }
	        Log.d(WiFiDirectActivity.TAG, "P2P peers changed");
	    }...
	}


现在，使用 WIFI_P2P_PEERS_CHANGED_ACTION 意图将触发对 P2P 列表请求的更新。

## 连接到 P2P

为了连接到 P2P，创建一个新的 WifiP2pConfig 对象，并从要连接的设备 WifiP2pDevice 复制数据。 然后调用connect（）方法。

	@Override
    public void connect() {
        // Picking the first device found on the network.
        WifiP2pDevice device = peers.get(0);

        WifiP2pConfig config = new WifiP2pConfig();
        config.deviceAddress = device.deviceAddress;
        config.wps.setup = WpsInfo.PBC;

        mManager.connect(mChannel, config, new ActionListener() {

            @Override
            public void onSuccess() {
                // WiFiDirectBroadcastReceiver will notify us. Ignore for now.
            }

            @Override
            public void onFailure(int reason) {
                Toast.makeText(WiFiDirectActivity.this, "Connect failed. Retry.",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }

如果你组中的每个设备都直接支持 Wi-Fi，则在连接时不需要明确要求组的密码。 但是，要允许不支持 Wi-Fi Direct 的设备加入组，你需要通过调用 requestGroupInfo（） 来检索此密码，如以下代码片段所示：
	
	mManager.requestGroupInfo(mChannel, new GroupInfoListener() {
	  @Override
	  public void onGroupInfoAvailable(WifiP2pGroup group) {
	      String groupPassword = group.getPassphrase();
	  }
	});

请注意，在 connect（） 方法中实现的 WifiP2pManager.ActionListener 仅在启动成功或失败时通知你。要监听连接状态的变化，请执行 WifiP2pManager.ConnectionInfoListener 接口。 它的 onConnectionInfoAvailable（） 回调将在连接状态更改时通知你。 如果多个设备要连接到单个设备（如具有3个或更多播放器的游戏或聊天应用程序），则会将一个设备指定为“组所有者”。 你可以按照“创建组”部分中的步骤将特定设备指定为网络组所有者。

	@Override
    public void onConnectionInfoAvailable(final WifiP2pInfo info) {

        // InetAddress from WifiP2pInfo struct.
        InetAddress groupOwnerAddress = info.groupOwnerAddress.getHostAddress());

        // After the group negotiation, we can determine the group owner
        // (server).
        if (info.groupFormed && info.isGroupOwner) {
            // Do whatever tasks are specific to the group owner.
            // One common case is creating a group owner thread and accepting
            // incoming connections.
        } else if (info.groupFormed) {
            // The other device acts as the peer (client). In this case,
            // you'll want to create a peer thread that connects
            // to the group owner.
        }
    }


现在回到广播接收者的 onReceive（） 方法，并修改监听 WIFI_P2P_CONNECTION_CHANGED_ACTION 意图的部分。 收到这个意图后，调用 requestConnectionInfo（）。 这是一个异步调用，所以结果将被你提供的连接信息监听器作为参数接收。

		...
        } else if (WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION.equals(action)) {

            if (mManager == null) {
                return;
            }

            NetworkInfo networkInfo = (NetworkInfo) intent
                    .getParcelableExtra(WifiP2pManager.EXTRA_NETWORK_INFO);

            if (networkInfo.isConnected()) {

                // We are connected with the other device, request connection
                // info to find group owner IP

                mManager.requestConnectionInfo(mChannel, connectionListener);
            }
            ...

## 创建一个组

如果你希望运行应用的设备作为包含旧设备的网络的组所有者（即不支持 Wi-Fi Direct 的设备），则遵循与“连接到 P2P ”相同的步骤部分，除了你使用 createGroup（） 而不是 connect（） 创建一个新的 WifiP2pManager.ActionListener 。 WifiP2pManager.ActionListener 中的回调处理是相同的，如下面的代码片段所示：

	mManager.createGroup(mChannel, new WifiP2pManager.ActionListener() {
	    @Override
	    public void onSuccess() {
	        // Device is ready to accept incoming connections from peers.
	    }
	
	    @Override
	    public void onFailure(int reason) {
	        Toast.makeText(WiFiDirectActivity.this, "P2P group creation failed. Retry.",
	                Toast.LENGTH_SHORT).show();
	    }
	});

> 注意：如果网络中的所有设备都支持 Wi-Fi Direct ，则可以在每个设备上使用 connect（） 方法，因为该方法会自动创建组并选择组所有者。

创建组后，可以调用 requestGroupInfo（） 来检索网络上对等体的详细信息，包括设备名称和连接状态

翻译者：[JackWaiting](https://github.com/JackWaiting)

翻译原址：[https://developer.android.com/training/connect-devices-wirelessly/wifi-direct.html#create-group](https://developer.android.com/training/connect-devices-wirelessly/wifi-direct.html#create-group)