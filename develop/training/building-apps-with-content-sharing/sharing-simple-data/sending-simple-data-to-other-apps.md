#向其他应用发送简单的数据
当构建一个 Intent 时，必须指定这个 Intent 需要触发的 Action。Android 定义了一些 Action，比如[ACTION_SEND](https://developer.android.google.cn/reference/android/content/Intent.html#ACTION_SEND)，该 Action 会指示 Intent 用于从一个 Activity 发送数据到另外一个 Activity，甚至可以是跨进程之间的数据发送。为了发送数据到另外一个 Activity，我们只需要说明数据及其类型，系统会自动识别出能够兼容接受这些数据的 Activity。如果这些选择有多个，则把这些 Activity 显示给用户进行选择；如果只有一个，则立即启动该 Activity。同样的，我们可以在 manifest 文件的 Activity 描述中添加接受的数据类型。

在不同的程序之间使用 Intent 收发数据是在社交分享内容时最常用的方法。Intent 使用户能够通过最喜欢的程序进行快速简单的信息分享。

**注意:**为 [ActionBar](https://developer.android.com/reference/android/app/ActionBar.html) 添加分享功能的最佳方法是使用 [ShareActionProvider](https://developer.android.com/reference/android/widget/ShareActionProvider.html)，API level 14 以上的系统才支持。[ShareActionProvider](https://developer.android.com/reference/android/widget/ShareActionProvider.html) 将在关于[添加简易分享 Action](https://developer.android.com/training/sharing/shareaction.html)的课程中进行详细介绍。

##分享文本内容

[ACTION_SEND](https://developer.android.com/reference/android/content/Intent.html#ACTION_SEND) 最直接常用的地方是从一个 Activity 发送文本内容到另外一个 Activity。例如，Android 内置的浏览器可以将当前显示页面的 URL 作为文本内容分享到其他程序。这一功能对于通过邮件或者社交网络来分享文章或者网址给好友而言是非常有用的。下面是一段展示分享功能的代码:

```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
sendIntent.setType("text/plain");
startActivity(sendIntent);
```    

如果设备上安装有某个能够匹配 [ACTION_SEND](https://developer.android.com/reference/android/content/Intent.html#ACTION_SEND) 且 MIME 类型为 text/plain 的程序，则 Android 系统会立即运行它。若有多个匹配的程序，则系统会把他们都给筛选出来，并呈现一个选择提示框给用户进行选择。

然而，如果你调用了 [Intent.createChooser()](https://developer.android.com/reference/android/content/Intent.html#createChooser(android.content.Intent, java.lang.CharSequence))，传入 Intent 对象，那么 Android **总是会显示选择器**。这样做有一些好处：

+ 即使用户之前为这个 Intent 设置了默认的 Action，选择界面还是会显示。
+ 如果没有匹配的程序，Android会 显示系统信息。
+ 我们可以指定选择器界面的标题。

下面是更新后的代码：

```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
sendIntent.setType("text/plain");
startActivity(Intent.createChooser(sendIntent, getResources().getText(R.string.send_to));
```  

![image](share-text-screenshot.png)  
**图1：** 手机端 [ACTION_SEND](https://developer.android.com/reference/android/content/Intent.html#ACTION_SEND) 意图选择器屏幕截图。

图 1 显示的弹出框结果。

另外，我们可以为 Intent 设置一些标准的附加值，例如：[EXTRA_EMAIL](https://developer.android.com/reference/android/content/Intent.html#EXTRA_EMAIL), [EXTRA_CC](https://developer.android.com/reference/android/content/Intent.html#EXTRA_CC), [EXTRA_BCC](https://developer.android.com/reference/android/content/Intent.html#EXTRA_BCC), [EXTRA_SUBJECT](https://developer.android.com/reference/android/content/Intent.html#EXTRA_SUBJECT) 等。如果接收程序没有针对这些做特殊的处理，则会忽略不处理。
>**注：** 一些 e-mail 程序，例如 Gmail,对应接收的是 [EXTRA_EMAIL](https://developer.android.com/reference/android/content/Intent.html#EXTRA_EMAIL), [EXTRA_CC](https://developer.android.com/reference/android/content/Intent.html#EXTRA_CC)，他们都是 [String[]](https://developer.android.com/reference/java/lang/String.html) 类型的，可以使用 [putExtra(string,string[])](https://developer.android.com/reference/android/content/Intent.html#putExtra(java.lang.String, java.lang.String[])) 方法来添加至 Intent 中。

##分享二进制内容
分享二进制的数据需要使用 [ACTION_SEND](https://developer.android.com/reference/android/content/Intent.html#ACTION_SEND) 设置特定的 MIME 类型，需要在 [EXTRA_STREAM](https://developer.android.com/reference/android/content/Intent.html#EXTRA_STREAM) 里面放置数据的 URI。 下面有个分享图片的例子，该例子也可以修改用于分享任何类型的二进制数据：

```java
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_STREAM, uriToImage);
shareIntent.setType("image/jpeg");
startActivity(Intent.createChooser(shareIntent, getResources().getText(R.string.send_to)));
```
请注意以下内容：

* 我们可以使用  \*/*  这样的方式来指定 MIME 类型，但是这仅仅会匹配到那些能够处理一般数据类型的 Activity (即一般的 Activity 无法详尽所有的 MIME 类型)。
 
* 接收的程序需要有访问 [Uri](https://developer.android.com/reference/android/net/Uri.html) 资源的权限。推荐以下方法来处理这个问题：
 
  * 将数据存储在 [ContentProvider](https://developer.android.com/reference/android/content/ContentProvider.html) 中，确保其他程序有访问 ContentProvider 的权限。较好的提供访问权限的方法是使用 [per-URI permissions](https://developer.android.com/guide/topics/security/permissions.html#uri)，其对接收程序而言是只是暂时拥有该许可权限。类似于这样创建 [ContentProvider](https://developer.android.com/reference/android/content/ContentProvider.html) 的一种简单的方法是使用 [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html) 帮助类。
  
  * 使用 [MediaStore](https://developer.android.com/reference/android/provider/MediaStore.html) 系统。[MediaStore](https://developer.android.com/reference/android/provider/MediaStore.html) 系统主要用于音视频及图片的 MIME 类型。但在 Android 3.0 之后，它也可以用于存储非多媒体类型的数据（更多信息请参阅 [MediaStore.Files](https://developer.android.com/reference/android/provider/MediaStore.Files.html)）。


Files can be inserted into the MediaStore using scanFile() after which a content:// style Uri suitable for sharing is passed to the provided onScanCompleted() callback. 

在有适合分享的 `content://` 类型的 [Uri](https://developer.android.com/reference/android/net/Uri.html) 传递到 [onScanCompleted()](https://developer.android.com/reference/android/media/MediaScannerConnection.OnScanCompletedListener.html#onScanCompleted(java.lang.String, android.net.Uri)) 回调方法之后，可以通过调用 [scanFile()](https://developer.android.com/reference/android/media/MediaScannerConnection.html#scanFile(android.content.Context, java.lang.String[], java.lang.String[], android.media.MediaScannerConnection.OnScanCompletedListener)) 方法来向 MediaStore 插入文件。
  
##发送多个内容

为了同时分享多种不同类型的内容，需要 [ACTION\_SEND\_MULTIPLE](https://developer.android.google.cn/reference/android/content/Intent.html#ACTION_SEND_MULTIPLE) Action 与指定内容 URI 列表一起配合使用。MIME 类型也会根据分享的内容结构的不同而有差异。例如，如果你分享 3 张 JPEG 格式的图片，那么 MIME 类型仍然是 "image/jpeg"。如果是不同图片格式的话，应该是用 "image/\*" 来匹配那些可以接收任何图片类型的 Activity。如果需要分享多种不同类型的数据，可以使用 \*/* 来表示 MIME。像前面描述的那样，这取决于接收你数据的应用如何解析和处理你的数据。下面是一个例子：

```java
ArrayList<Uri> imageUris = new ArrayList<Uri>();
imageUris.add(imageUri1); // Add your image URIs here
imageUris.add(imageUri2);
I
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, imageUris);
shareIntent.setType("image/*");
startActivity(Intent.createChooser(shareIntent, "Share images to.."));
```
当然，请确保指定到数据的 [URI](https://developer.android.com/reference/android/net/Uri.html) 能够被接收程序所访问。


>翻译：[@misparking](https://github.com/misparking)    
原始文档：<https://developer.android.google.cn/training/sharing/send.html>