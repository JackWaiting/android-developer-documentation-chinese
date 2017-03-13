#从其他应用接收简单数据

如同你的应用可以向其他应用发送数据一样，你的应用也可以很容易的从其他应用接收数据。思考下用户如何和你的应用进行交互，以及你想要从其他应用获取数据的类型。例如，社交网络类的应用更倾向于接收文本内容，从另外一个应用获取诸如网页 URL 的数据。[Google+ Android 应用](https://play.google.com/store/apps/details?id=com.google.android.apps.plus)既接收文本信息，又可以接收单张或多张图片。通过这个应用，用户可以很容易的在 Google+ 中发布从 Android 图片应用中获取的图片。

##更新你的 Manifest 文件

意图过滤器提醒系统一个应用组件想要接收什么意图。和在[向其他应用发送简单数据](https://developer.android.com/training/sharing/send.html)课程里，你如何通过 [ACTION_SEND](https://developer.android.com/reference/android/content/Intent.html#ACTION_SEND) 的 Action 来构建一个意图类似，你创建意图过滤器是为了通过这个 Action 来接收意图。你在 Manifest 文件中定义一个意图过滤器，使用 `<intent-filter> `元素。例如，如果你的应用处理文本内容接收，一张或者多张任何类型的图片，你的 Manifest 文件就会如下所示：

```Java
<activity android:name=".ui.MyActivity" >
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND_MULTIPLE" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```

>**备注：** 更多关于意图过滤器和意图解决方案，请参阅[意图和意图过滤器](https://developer.android.com/guide/components/intents-filters.html#ifs)。

当其他应用通过构建一个意图并将它传给 [startActivity()](https://developer.android.com/reference/android/content/Context.html#startActivity(android.content.Intent)) 方法来尝试分享此种内容的时候，你的应用将会在意图选择器中作为一个可选项被列出。相符合 Activity（以上样例中的 `.ui.MyActivity`） 将会被启动。然后就是取决于你在你的代码和 UI 中如何正确的处理这个内容了。

##处理接收的内容

要处理 [Intent](https://developer.android.com/reference/android/content/Intent.html) 传送的内容，需要先通过调用 [getIntent()](https://developer.android.com/reference/android/content/Intent.html#getIntent(java.lang.String)) 方法来获取 [Intent](https://developer.android.com/reference/android/content/Intent.html) 对象。一旦你拥有了这个对象，你就可以判断它的内容来决定下一步做什么。要注意的就是如果这个 Activity 可以被系统的其他部分启动的话，例如一个 launcher，需要在检查 intent 的时候考虑这种情况。
Keep in mind that if this activity can be started from other parts of the system, such as the launcher, then you will need to take this into consideration when examining the intent.
```Java
void onCreate (Bundle savedInstanceState) {
    ...
    // Get intent, action and MIME type
    Intent intent = getIntent();
    String action = intent.getAction();
    String type = intent.getType();

    if (Intent.ACTION_SEND.equals(action) && type != null) {
        if ("text/plain".equals(type)) {
            handleSendText(intent); // Handle text being sent
        } else if (type.startsWith("image/")) {
            handleSendImage(intent); // Handle single image being sent
        }
    } else if (Intent.ACTION_SEND_MULTIPLE.equals(action) && type != null) {
        if (type.startsWith("image/")) {
            handleSendMultipleImages(intent); // Handle multiple images being sent
        }
    } else {
        // Handle other intents, such as being started from the home screen
    }
    ...
}

void handleSendText(Intent intent) {
    String sharedText = intent.getStringExtra(Intent.EXTRA_TEXT);
    if (sharedText != null) {
        // Update UI to reflect text being shared
    }
}

void handleSendImage(Intent intent) {
    Uri imageUri = (Uri) intent.getParcelableExtra(Intent.EXTRA_STREAM);
    if (imageUri != null) {
        // Update UI to reflect image being shared
    }
}

void handleSendMultipleImages(Intent intent) {
    ArrayList<Uri> imageUris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
    if (imageUris != null) {
        // Update UI to reflect multiple images being shared
    }
}
```

>**注意：**一定要格外检查接收的数据，你无法知道其他应用可能给你发送的数据类型。比如，可能会设置错误的 MIME 类型，或者过大的图片。因此我们应避免在 UI 线程里面去处理那些获取到的数据。


更新 UI 可以像填充 [EditText](https://developer.android.com/reference/android/widget/EditText.html) 一样简单，也可以是更加复杂一点的操作，例如过滤出感兴趣的图片。这个完全取决于应用接下来要做些什么。




