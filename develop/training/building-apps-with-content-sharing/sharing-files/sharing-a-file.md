#共享文件
***

将应用设置为使用内容 URI 共享文件后，你可以响应其他应用对这些文件的请求。 响应这些请求的一种方式是从服务器应用提供其他应用可以调用的文件选择接口。 这种方法允许客户端应用程序让用户从服务器应用程序中选择一个文件，然后接收所选文件的内容 URI。

本课将向你展示如何在应用程序中创建响应文件请求的文件选择的 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html)

##接收文件请求

要从客户端应用程序接收文件请求并使用内容 URI 进行响应，你的应用程序应提供文件选择 Activity。 客户端应用程序通过使用包含操作 [ACTION_PICK](https://developer.android.google.cn/reference/android/content/Intent.html#ACTION_PICK) 的 Intent 调用 [startActivityForResult（）](https://developer.android.google.cn/reference/android/app/Activity.html#startActivityForResult)来启动此 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html)。 当客户端应用调用 [startActivityForResult（）](https://developer.android.google.cn/reference/android/app/Activity.html#startActivityForResult) 时，你的应用会返回用户选择文件的 URI 地址。

要了解如何在客户端应用程序中实现文件请求，请参阅请求共享文件中的课程。

##创建文件选择的 Activity

要创建文件选择的 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html)，请首先在清单中指定 Activity，以及与操作 [ACTION_PICK](https://developer.android.google.cn/reference/android/content/Intent.html#ACTION_PICK) 和类别 [CATEGORY_DEFAULT](https://developer.android.google.cn/reference/android/content/Intent.html#CATEGORY_DEFAULT) 和 [CATEGORY_OPENABLE](https://developer.android.google.cn/reference/android/content/Intent.html#CATEGORY_OPENABLE) 匹配的意图过滤器。 还要为你的应用为其他应用提供的文件添加 MIME 类型过滤器。 以下代码段显示如何指定新的 Activity 和意向过滤器：

	<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    ...
        <application>
        ...
            <activity
                android:name=".FileSelectActivity"
                android:label="@File Selector" >
                <intent-filter>
                    <action
                        android:name="android.intent.action.PICK"/>
                    <category
                        android:name="android.intent.category.DEFAULT"/>
                    <category
                        android:name="android.intent.category.OPENABLE"/>
                    <data android:mimeType="text/plain"/>
                    <data android:mimeType="image/*"/>
                </intent-filter>
            </activity>

#在代码中定义文件选择的 Activity

接下来，定义一个 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html) 子类，显示从应用程序的 files/images/ 目录在内部存储中可用的文件，并允许用户选择所需的文件。 以下代码段演示了如何定义此 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html) 并响应用户的选择：

	public class MainActivity extends Activity {
	    // The path to the root of this app's internal storage
	    private File mPrivateRootDir;
	    // The path to the "images" subdirectory
	    private File mImagesDir;
	    // Array of files in the images subdirectory
	    File[] mImageFiles;
	    // Array of filenames corresponding to mImageFiles
	    String[] mImageFilenames;
	    // Initialize the Activity
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Set up an Intent to send back to apps that request a file
        mResultIntent =
                new Intent("com.example.myapp.ACTION_RETURN_FILE");
        // Get the files/ subdirectory of internal storage
        mPrivateRootDir = getFilesDir();
        // Get the files/images subdirectory;
        mImagesDir = new File(mPrivateRootDir, "images");
        // Get the files in the images subdirectory
        mImageFiles = mImagesDir.listFiles();
        // Set the Activity's result to null to begin with
        setResult(Activity.RESULT_CANCELED, null);
        /*
         * Display the file names in the ListView mFileListView.
         * Back the ListView with the array mImageFilenames, which
         * you can create by iterating through mImageFiles and
         * calling File.getAbsolutePath() for each File
         */
         ...
   		}
    ...
	}

##响应文件选择

一旦用户选择了共享文件，你的应用程序必须确定选择了哪个文件，然后为该文件生成内容 URI。 由于  [Activity](https://developer.android.google.cn/reference/android/app/Activity.html) 显示 [ListView](https://developer.android.google.cn/reference/android/widget/ListView.html) 中的可用文件列表，当用户单击文件名时，系统调用方法 [onItemClick（）](https://developer.android.google.cn/reference/android/widget/AdapterView.OnItemClickListener.html#onItemClick)，在其中可以获取所选文件。

当使用意图将文件的 URI 从一个应用程序发送到另一个应用程序时，你必须小心获取其他应用程序可以读取的 URI。 在运行 Android 6.0（API级别23）及更高版本的设备上执行此操作时需要特别注意，因为该版本 Android 中的权限模型发生了更改，特别是 [READ_EXTERNAL_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 已成为危险权限，接收应用可能缺少此权限。

考虑到这些因素，我们建议你避免使用 [Uri.fromFile（）](https://developer.android.google.cn/reference/android/net/Uri.html#fromFile(java.io.File))，这存在一些缺点。 这种方法：

* 不允许在配置文件之间共享文件。
* 要求你的应用拥有 [WRITE_EXTERNAL_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 权限，当执行在 Android 4.4（API等级19）或更低版本的系统上。
* 要求接收应用具有 [READ_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 权限，在没有这项权限的重要应用（例如 Gmail ）上，请求将会失败。

替代 [Uri.fromFile（）](https://developer.android.google.cn/reference/android/net/Uri.html#fromFile(java.io.File))，你可以使用 URI 权限授予其他应用程序访问特定的 URI。尽管 URI 权限对由 [Uri.fromFile（）](https://developer.android.google.cn/reference/android/net/Uri.html#fromFile(java.io.File)) 生成的 file：// URI 不起作用，但它们在与内容提供者相关联的 URI 上工作。 FileProvider API 可以帮助你创建此类 URI。此方法也适用于不在外部存储中的文件，但在发送意图的应用程序的本地存储中。

在 onItemClick（）中，为所选文件的文件名获取 File 对象，并将其作为参数传递给 getUriForFile（），以及在 FileProvider 的 [&lt;provider&gt;](https://developer.android.google.cn/guide/topics/manifest/provider-element.html) 元素中指定的权限。结果内容 URI 包含权限，对应于文件目录（如 XML 元数据中指定的）的路径段以及包括其扩展名的文件的名称。 [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html) 如何将目录映射到基于 XML 元数据的路径段，请参阅指定可分享目录一节。

以下代码段显示了如何检测所选文件并获取其内容 URI：

	    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            /*
             * When a filename in the ListView is clicked, get its
             * content URI and send it to the requesting app
             */
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                /*
                 * Get a File for the selected file name.
                 * Assume that the file names are in the
                 * mImageFilename array.
                 */
                File requestFile = new File(mImageFilename[position]);
                /*
                 * Most file-related method calls need to be in
                 * try-catch blocks.
                 */
                // Use the FileProvider to get a content URI
                try {
                    fileUri = FileProvider.getUriForFile(
                            MainActivity.this,
                            "com.example.myapp.fileprovider",
                            requestFile);
                } catch (IllegalArgumentException e) {
                    Log.e("File Selector",
                          "The selected file can't be shared: " +
                          clickedFilename);
                }
                ...
            }
        });
        ...
    }

请记住，您只能为驻留在包含 < path > 元素的元数据文件中指定的目录中的文件生成内容 URI，如指定可分享目录一节中所述。 如果对未指定的路径中的文件调用 getUriForFile（），则会收到 [IllegalArgumentException](https://developer.android.google.cn/reference/java/lang/IllegalArgumentException.html)。

##授予文件权限
现在你有要分享到另外一个应用的文件 URI 地址，你需要允许客户端应用能访问这个文件。 要允许访问，通过将内容 URI 添加到 [Intent](https://developer.android.google.cn/reference/android/content/Intent.html)，然后在 Intent 上设置权限标志，向客户端应用程序授予权限。 您授予的权限是临时的，并在接收应用程序的任务堆栈完成时自动过期。

以下代码段显示如何为文件设置读取权限：

    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    // Grant temporary read permission to the content URI
                    mResultIntent.addFlags(
                        Intent.FLAG_GRANT_READ_URI_PERMISSION);
                }
                ...
             }
             ...
        });
    ...
    }

>**警告**：调用 [setFlags（）](https://developer.android.google.cn/reference/android/content/Intent.html#setFlags(int))是使用临时访问权限安全授予对文件的访问权限的唯一方法。 避免为文件的内容 URI 调用 Context.grantUriPermission（）方法，因为此方法授予你只能通过调用 Context.revokeUriPermission（）撤消的访问。

不要使用 Uri.fromFile（）。 它强制接收应用程序具有 [READ_EXTERNAL_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 权限，如果你尝试在所有用户之间共享，则不会工作;在低于 4.4（API级别19）的 Android 版本中，你的应用程序需要拥有 [WRITE_EXTERNAL_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)。 而且真正重要的共享目标（例如 Gmail 应用）没有 READ_EXTERNAL_STORAGE，导致此调用失败。 相反，你可以使用 URI 权限授予其他应用程序对特定 URI 的访问权限。 虽然 URI 权限对由 Uri.fromFile（）生成的 file：// URIs 不起作用，但它们在与内容提供者相关联的 Uris 上工作。 而不是实现自己的，为此，你可以和应该使用 [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html) ，如文件共享中所述。

##与请求应用程序共享文件

要与请求它的应用程序共享文件，请将包含内容 URI 和权限的 Intent 传递给 [setResult（）](https://developer.android.google.cn/reference/android/app/Activity.html#setResult(int))。 当刚刚定义的 Activity 完成后，系统将包含内容 URI 的 Intent 发送到客户端应用程序。 以下代码段显示了如何执行此操作：

	    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    ...
                    // Put the Uri and MIME type in the result Intent
                    mResultIntent.setDataAndType(
                            fileUri,
                            getContentResolver().getType(fileUri));
                    // Set the result
                    MainActivity.this.setResult(Activity.RESULT_OK,
                            mResultIntent);
                    } else {
                        mResultIntent.setDataAndType(null, "");
                        MainActivity.this.setResult(RESULT_CANCELED,
                                mResultIntent);
                    }
                }
        });


为用户提供一种在选择文件后立即返回到客户端应用程序的方法。 一种方法是提供复选标记或完成按钮。 使用按钮的 [android：onClick](https://developer.android.google.cn/reference/android/view/View.html#attr_android:onClick) 属性将方法与按钮相关联。 在方法中，调用 [finish（）](https://developer.android.google.cn/reference/android/app/Activity.html#finish())。 例如：

	 public void onDoneClick(View v) {
        // Associate a method with the Done button
        finish();
    }



>**备注**  
翻译：[@jarylan](https://github.com/jarylan)    
校对：[@iOnesmile](https://github.com/iOnesmile)    
原始文档:  [https://developer.android.com/training/secure-file-sharing/share-file.html](https://developer.android.com/training/secure-file-sharing/share-file.html)
