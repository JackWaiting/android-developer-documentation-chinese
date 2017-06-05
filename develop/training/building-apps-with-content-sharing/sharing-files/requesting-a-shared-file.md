#请求共享文件

当应用程序想要访问由其他应用程序共享的文件时，请求应用程序（客户端）通常会向共享文件的应用程序（服务器）发送请求。在大多数情况下，请求在服务器应用程序中启动一个 [Activity](https://developer.android.com/reference/android/app/Activity.html) ，显示它可以共享的文件。 用户选择一个文件，之后服务器应用程序将文件的内容URI返回给客户端应用程序。 

本课将向您展示客户端应用程序如何从服务器应用程序请求文件，从服务器应用程序接收文件的内容URI，以及如何使用内容URI打开文件。

##发送文件请求

要从服务器应用程序请求文件，客户端应用程序使用包含操作（如 [ACTION_PICK](https://developer.android.com/reference/android/content/Intent.html#ACTION_PICK) ）和客户端应用程序可以处理的 MIME 类型的 [Intent](https://developer.android.com/reference/android/content/Intent.html) 调用 [startActivityForResult](https://developer.android.com/reference/android/app/Activity.html#startActivityForResult)。

例如，以下代码段演示了如何向服务器应用程序发送 [Intent](https://developer.android.com/reference/android/content/Intent.html) 以启动共享文件中描述的[Activity](https://developer.android.com/reference/android/app/Activity.html) ：

		public class MainActivity extends Activity {
	    private Intent mRequestFileIntent;
	    private ParcelFileDescriptor mInputPFD;
	    ...
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        mRequestFileIntent = new Intent(Intent.ACTION_PICK);
	        mRequestFileIntent.setType("image/jpg");
	        ...
	    }
	    ...
	    protected void requestFile() {
	        /**
	         * When the user requests a file, send an Intent to the
	         * server app.
	         * files.
	         */
	            startActivityForResult(mRequestFileIntent, 0);
	        ...
	    }
	    ...
	}


##访问请求的文件

服务器应用程序在 [Intent](https://developer.android.com/reference/android/content/Intent.html) 中将文件的内容URI发送回客户端应用程序。 这个  [Intent](https://developer.android.com/reference/android/content/Intent.html) 传递给客户端应用程序的覆盖 [startActivityForResult](https://developer.android.com/reference/android/app/Activity.html#startActivityForResult) 。 一旦客户端应用程序具有文件的内容URI，它就可以通过获取其 [FileDescriptor](https://developer.android.com/reference/java/io/FileDescriptor.html) 来访问文件。

在此过程中保留文件安全性，因为内容URI是客户端应用程序接收的唯一数据。 由于此URI不包含目录路径，因此客户端应用程序无法发现和打开服务器应用程序中的任何其他文件。 只有客户端应用程序可以访问该文件，并且只有服务器应用程序授予的权限。 权限是临时的，因此一旦客户端应用程序的任务堆栈完成，该文件就不再在服务器应用程序之外访问。

下一个片段演示了客户端应用程序如何处理从服务器应用程序发送的 [Intent](https://developer.android.com/reference/android/content/Intent.html) ，以及客户端应用程序如何使用内容URI获取 [FileDescriptor](https://developer.android.com/reference/java/io/FileDescriptor.html) ：

	/*
     * When the Activity of the app that hosts files sets a result and calls
     * finish(), this method is invoked. The returned Intent contains the
     * content URI of a selected file. The result code indicates if the
     * selection worked or not.
     */
    @Override
    public void onActivityResult(int requestCode, int resultCode,
            Intent returnIntent) {
        // If the selection didn't work
        if (resultCode != RESULT_OK) {
            // Exit without doing anything else
            return;
        } else {
            // Get the file's content URI from the incoming Intent
            Uri returnUri = returnIntent.getData();
            /*
             * Try to open the file for "read" access using the
             * returned URI. If the file isn't found, write to the
             * error log and return.
             */
            try {
                /*
                 * Get the content resolver instance for this context, and use it
                 * to get a ParcelFileDescriptor for the file.
                 */
                mInputPFD = getContentResolver().openFileDescriptor(returnUri, "r");
            } catch (FileNotFoundException e) {
                e.printStackTrace();
                Log.e("MainActivity", "File not found.");
                return;
            }
            // Get a regular file descriptor for the file
            FileDescriptor fd = mInputPFD.getFileDescriptor();
            ...
        }
    }


方法 [openFileDescriptor（）](https://developer.android.com/reference/android/content/ContentResolver.html#openFileDescriptor)为文件返回一个 [ParcelFileDescriptor](https://developer.android.com/reference/android/os/ParcelFileDescriptor.html) 。 从此对象，客户端应用程序获取一个 [FileDescriptor](https://developer.android.com/reference/java/io/FileDescriptor.html) 对象，然后可以使用它来读取文件。

翻译人：[JackWaiting](https://github.com/JackWaiting)

原文地址：[https://developer.android.com/training/secure-file-sharing/request-file.html#OpenFile](https://developer.android.com/training/secure-file-sharing/request-file.html#OpenFile)