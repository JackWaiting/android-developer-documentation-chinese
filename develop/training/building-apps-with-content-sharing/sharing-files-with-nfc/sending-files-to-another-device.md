# 向其他设备发送文件

这节课教你如何在你的应用中使用 Android Beam 文件传输来向另外一个设备发送大文件。发送文件之前，你需要请求使用 NFC 和内部存储的权限，测试确保你的设备支持 NFC，给 Android Beam 文件传输提供 URI。

<small>Android Beam 文件传输功能有以下要求：</small>

<small>1.Android Beam 大文件传输功能在 Android 4.1（API Level 16）以及更高版本上才能使用。</small>

<small>2.你要传输的文件必须位于外部存储。更多关于外部存储的使用，请参阅[外部存储的使用](https://developer.android.google.cn/guide/topics/data/data-storage.html#filesExternal)。</small>

<small>3.每一个你想要传输的文件必须是全局可读的。你可以通过调用 `File.setReadable(true,false)` 方法来设置这个权限。</small>

<small>4.你必须为你需要传输的文件提供一个文件 URI。Android Beam 文件传输无法处理通过 `FileProvider.getUriForFile` 生成的内容 URI。</small>

## Manifest 文件中声明特性

首先，编辑应用中的 Manifest 文件来声明应用需要的权限和特性。

### 请求权限

为了能够让你的应用通过 NFC 使用 Android Beam 文件传输来发送外部存储的文件，你应当在应用 Manifest 文件中请求以下权限：

[NFC](https://developer.android.google.cn/reference/android/Manifest.permission.html#NFC)

<small>允许你的应用通过 NFC 发送数据。要指定此权限，需要添加以下元素作为 `<manifest>` 的子元素：</small>

```
<uses-permission android:name="android.permission.NFC" />
```

[READ\_EXTERNAL\_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)

<small>允许你的应用读取外部存储。要指定此权限，需要添加以下元素作为 `<manifest>` 的子元素：</small>

```
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

>**备注：**从 Android 4.2.2 (API Level 17）开始，这个权限不再强制了。针对想要读取外部存储的应用，将来的平台版本可能会有此权限要求。为了确保向前兼容性，在强制之前，我们现在就请求此权限。


### 指定 NFC 特性

<small>可以通过追加一个 `<uses-feature>` 的 `<manifest>` 的子元素，来声明你的应用需要使用 NFC。将 `android:required` 属性设置为 `true` 的话，就表明只有当前 NFC 连接打开时应用才能使用。</small>

<small>以下代码片段教你如何指定 `<uses-feature>` 元素：</small>

```
<uses-feature
    android:name="android.hardware.nfc"
    android:required="true" />
```

备注：如果你的应用对 NFC 的使用只是一个可选项，但是如果当前 NFC 功能不能使用的话，仍然可以工作，你应当将 `android:required` 属性设置成 `false`，在代码中判断是否支持 NFC。

### 指定 Android Beam 文件传输

因为 Android Beam 文件传输在 Android 4.1 （API level 16）及以上版本才支持，如果你的应用依赖 Android Beam 文件传输作为功能的关键部分，那么你用 `android:minSdkVersion="16"` 属性来指定 `<uses-sdk>` 元素。

## 测试 Android Beam 文件传输的支持

如果在应用 manifest 文件中指定 NFC 是可选的，你需要使用以下元素：

```
<uses-feature android:name="android.hardware.nfc" android:required="false" />
```

如果你设置 `android:required="false"` 属性的话，你必须在代码中判断是否有 NFC 功能和支持 Android Beam 文件传输功能。

可以通过调用 `PackageManager.hasSystemFeature()` 方法传入 `FEATURE_NFC ` 参数来判断设备是否支持 NFC 功能。然后通过判断 `SDK_INT ` 的值来确定 Android 系统是否支持 Android Beam 文件传输功能。如果 Android Beam 文件传输功能支持的话，可以通过获取一个 NFC 控制器的实例，这个实例可以让你和 NFC 硬件进行通信。例如：

```
public class MainActivity extends Activity {
    ...
    NfcAdapter mNfcAdapter;
    // Flag to indicate that Android Beam is available
    boolean mAndroidBeamAvailable  = false;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // NFC isn't available on the device
        if (!PackageManager.hasSystemFeature(PackageManager.FEATURE_NFC)) {
            /*
             * Disable NFC features here.
             * For example, disable menu items or buttons that activate
             * NFC-related features
             */
            ...
        // Android Beam file transfer isn't supported
        } else if (Build.VERSION.SDK_INT <
                Build.VERSION_CODES.JELLY_BEAN_MR1) {
            // If Android Beam isn't available, don't continue.
            mAndroidBeamAvailable = false;
            /*
             * Disable Android Beam file transfer features here.
             */
            ...
        // Android Beam file transfer is available, continue
        } else {
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        ...
        }
    }
    ...
}
```

## 创建提供文件的回调方法

一旦你确认设备支持 Android Beam 文件传输，添加一个系统回调方法，这个方法会在 Android Beam 文件传输检测到用户想要向另外一个启动 NFC 设备发送文件的动作。在这个回调方法中，会返回一个 [Uri](https://developer.android.google.cn/reference/android/net/Uri.html) 对象的数组。Android Beam 文件传输会复制这些用 URI 表示的文件到接收的设备。

要想添加这个回调方法，需要实现 [`NfcAdapter.CreateBeamUrisCallback`](https://developer.android.google.cn/reference/android/nfc/NfcAdapter.CreateBeamUrisCallback.html) 接口以及它的 [`createBeamUris()`](https://developer.android.google.cn/reference/android/nfc/NfcAdapter.CreateBeamUrisCallback.html#createBeamUris(android.nfc.NfcEvent))方法。以下代码片段告诉你如何做：

```
public class MainActivity extends Activity {
    ...
    // List of URIs to provide to Android Beam
    private Uri[] mFileUris = new Uri[10];
    ...
    /**
     * Callback that Android Beam file transfer calls to get
     * files to share
     */
    private class FileUriCallback implements
            NfcAdapter.CreateBeamUrisCallback {
        public FileUriCallback() {
        }
        /**
         * Create content URIs as needed to share with another device
         */
        @Override
        public Uri[] createBeamUris(NfcEvent event) {
            return mFileUris;
        }
    }
    ...
}
```

一旦你实现了这个接口，你需要通过调用 [`setBeamPushUrisCallback()`](https://developer.android.google.cn/reference/android/nfc/NfcAdapter.html#setBeamPushUrisCallback(android.nfc.NfcAdapter.CreateBeamUrisCallback, android.app.Activity)) 方法来传递回调给 Android Beam 文件传输。以下代码片段告诉你如何做：

```
public class MainActivity extends Activity {
    ...
    // Instance that returns available files from this app
    private FileUriCallback mFileUriCallback;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Android Beam file transfer is available, continue
        ...
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        /*
         * Instantiate a new FileUriCallback to handle requests for
         * URIs
         */
        mFileUriCallback = new FileUriCallback();
        // Set the dynamic callback for URI requests.
        mNfcAdapter.setBeamPushUrisCallback(mFileUriCallback,this);
        ...
    }
    ...
}
```

>**备注：**你也可以通过你应用的 [`NfcAdapter`](https://developer.android.google.cn/reference/android/nfc/NfcAdapter.html) 的实例直接给 NFC 框架提供 Uri 对象数组。如果你可以在 NFC 触摸时间发生之前就可以定义你需要传输的 URI 的话，可以选择这种方法。更多关于此方法，参见 [`NfcAdapter.setBeamPushUris()`](https://developer.android.google.cn/reference/android/nfc/NfcAdapter.html#setBeamPushUris(android.net.Uri[], android.app.Activity))。

## 指定传输文件

要想向其他支持 NFC 功能的设备传输一个或多个文件的话，需要获取每一个文件的 URI （带有文件 scheme 的 URI），然后将这个 URI 添加到 Uri 对象数组中。传输一个文件，你需要拥有这个文件的完全读写权限。例如，以下代码片段会告知你如何从一个文件名中获取文件 URI，然后添加 URI 到数组中：

```
        /*
         * Create a list of URIs, get a File,
         * and set its permissions
         */
        private Uri[] mFileUris = new Uri[10];
        String transferFile = "transferimage.jpg";
        File extDir = getExternalFilesDir(null);
        File requestFile = new File(extDir, transferFile);
        requestFile.setReadable(true, false);
        // Get a URI for the File and add it to the list of URIs
        fileUri = Uri.fromFile(requestFile);
        if (fileUri != null) {
            mFileUris[0] = fileUri;
        } else {
            Log.e("My Activity", "No File URI available for file.");
        }
```

>翻译：[@ifeegoo](https://github.com/ifeegoo)   
>审核：[@misparking](https://github.com/misparking)    
>原始文档：<https://developer.android.google.cn/training/beam-files/send-files.html>  















