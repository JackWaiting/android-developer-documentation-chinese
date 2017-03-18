# 从其他的设备接收文件
Android Beam 文件传输文件到接收设备的特殊目录下。它并且使用 Android Media Scanner 扫描复制的文件和添加媒体文件到 MediaStore 提供者。本片课程告诉你如何在文件复制完成后做出响应，并且在接收的设备上找到复制的文件。
## 响应请求去显示数据
当 Android Beam 文件传输到接收设备完成后， 它会发送一个通知，包含一个 ACTION_VIEW 动作，传输的第一个文件的 MIME 类型和指向都一个文件的 URI 的 Intent。当用户点击这个通知，这个意图将被发送到系统。要想你的应用响应这个意图，为 应该响应的Activity 的<activity> 元素添加一个 <intent-filter> 元素。这个 <intent-filter> 元素， 添加下面的子元素:

<action android:name="android.intent.action.VIEW" />

匹配发送 ACTION_VIEW 意图的通知。

<category android:name="android.intent.category.CATEGORY_DEFAULT" />

匹配一个没有明确种类的意图。

<data android:mimeType="mime-type" />

匹配 MIME 类型。仅指定您的应用程序可以处理的 MIME 类型。

```
    <activity
        android:name="com.example.android.nfctransfer.ViewActivity"
        android:label="Android Beam Viewer" >
        ...
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            ...
        </intent-filter>
    </activity>
```
>**Note:** Android Beam 文件传输不是 ACTION_VIEW 意图的唯一来源。接收设备上的其他应用也可能发送具有此动作的意图。处理这种情况将从  Get the directory from a content URI 章节讨论。
## 请求文件权限
读取 Android Beam 文件传输复制到设备的文件,请求权限 READ_EXTERNAL_STORAGE。如例子

```
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

```
如果你想复制传输文件到你应用独有的存储区域，替换成请求权限 WRITE_EXTERNAL_STORAGE。 WRITE_EXTERNAL_STORAGE 包含 READ_EXTERNAL_STORAGE. 
>**Note:** 在 Android 4.2.2(API level 17)，这个权限 READ_EXTERNAL_STORAGE 仅当用户选择使才施行。未来版本的平台可能所有的权限都需要这样。为了确保向前兼容，请在需求之前请求许可。

因为你的应用可以控制内部存储区域，当你复制传输的文件到内部存储区域时你不一定需要请求写入权限

## 获取已复制文件的目录
在一次传输中Android Beam 文件传输复制的所有文件在接收设备的一个文件夹中。Android Beam文件传输通知发送的内容意图的 URI 指向第一个传输的文件。但是，您的应用程序也可能从 Android Beam 文件传输之外的来源接收 ACTION_VIEW 意图。要确定如何处理传入Intent，您需要检查其方案和权限。要获取 URI 的方案，请调用Uri.getScheme（）。 以下代码段显示了如何确定方案并相应地处理URI：
```
public class MainActivity extends Activity {
    ...
    // A File object containing the path to the transferred files
    private File mParentPath;
    // Incoming Intent
    private Intent mIntent;
    ...
    /*
     * Called from onNewIntent() for a SINGLE_TOP Activity
     * or onCreate() for a new Activity. For onNewIntent(),
     * remember to call setIntent() to store the most
     * current Intent
     *
     */
    private void handleViewIntent() {
        ...
        // Get the Intent action
        mIntent = getIntent();
        String action = mIntent.getAction();
        /*
         * For ACTION_VIEW, the Activity is being asked to display data.
         * Get the URI.
         */
        if (TextUtils.equals(action, Intent.ACTION_VIEW)) {
            // Get the URI from the Intent
            Uri beamUri = mIntent.getData();
            /*
             * Test for the type of URI, by getting its scheme value
             */
            if (TextUtils.equals(beamUri.getScheme(), "file")) {
                mParentPath = handleFileUri(beamUri);
            } else if (TextUtils.equals(
                    beamUri.getScheme(), "content")) {
                mParentPath = handleContentUri(beamUri);
            }
        }
        ...
    }
    ...
}
```
### 从文件的 URI 获取目录
如果传入的 Intent 包含文件 URI ，则 URI 包含文件的绝对文件名，包括完整的目录路径和文件名。对于 Android Beam 文件传输，目录路径指向其他传输文件的位置（如果有）。 要获取目录路径，请获取 URI 的路径部分，其中包含除文件：前缀之外的所有 URI。 从路径部分创建一个文件，然后获取文件的父路径：
```
...
    public String handleFileUri(Uri beamUri) {
        // Get the path part of the URI
        String fileName = beamUri.getPath();
        // Create a File object for this filename
        File copiedFile = new File(fileName);
        // Get a string containing the file's parent directory
        return copiedFile.getParent();
    }
    ...
```
### 从内容的 URI 获取目录
如果传入 Intent 包含内容 URI ，则 URI 可以指向存储在 MediaStore 内容提供者中的目录和文件名。 您可以通过测试 URI 的权限值来检测 MediaStore的内容URI。 MediaStore 的内容 URI 可能来自 Android Beam 文件传输或来自其他应用程序，但在这两种情况下，您都可以检索内容URI的目录和文件名。

您还可以接收传入的 ACTION_VIEW 意图，其中包含除 MediaStore 之外的内容提供者的内容 URI。 在这种情况下，内容 URI 不包含 MediaStore 权限值，并且内容 URI 通常不指向目录。
>**Note:** 对
于Android Beam
文件传输，如果第一个传入文件的MIME类型为“audio / *”，“image / *”或“video / *”，则会在
ACTION_VIEW
意图中接收到一个内容
URI， - 相关。Android Beam 文件传输通过在存储传输文件的目录上运行 Media Scanner 来对其传输的媒体文件进行索引。Media Scanner 将其结果写入 MediaStore 内容提供程序，然后将第一个文件的内容 URI 传递回 Android Beam 文件传输。 此内容URI是您在通知 Intent 中收到的 URI。 要获取第一个文件的目录，可以使用内容 URI 从 MediaStore 中检索它。

### 确定内容提供者
要确定是否可以从内容 URI 检索文件目录，请通过调用 Uri.getAuthority（）来确定与 URI 关联的内容提供者以获取 URI 的权限。 结果有两个可能的值：

MediaStore.AUTHORITY

    该URI用于由MediaStore跟踪的一个或多个文件。 从MediaStore检索完整的文件名，并从文件名获取目录。
    
    
Any other authority value

    来自另一内容提供商的内容URI。 显示与内容URI相关联的数据，但不获取文件目录。
    
要获取 MediaStore 内容 URI 的目录，请运行一个查询，指定 Uri 参数的传入内容 URI 和投影的列 MediaColumns.DATA。 返回的 Cursor 包含由 URI 表示的文件的完整路径和名称。 此路径还包含 Android Beam 文件传输刚刚复制到设备的所有其他文件。

以下代码段显示了如何测试内容 URI 的权限，并检索传输文件的路径和文件名：  
    
```
...
    public String handleContentUri(Uri beamUri) {
        // Position of the filename in the query Cursor
        int filenameIndex;
        // File object for the filename
        File copiedFile;
        // The filename stored in MediaStore
        String fileName;
        // Test the authority of the URI
        if (!TextUtils.equals(beamUri.getAuthority(), MediaStore.AUTHORITY)) {
            /*
             * Handle content URIs for other content providers
             */
        // For a MediaStore content URI
        } else {
            // Get the column that contains the file name
            String[] projection = { MediaStore.MediaColumns.DATA };
            Cursor pathCursor =
                    getContentResolver().query(beamUri, projection,
                    null, null, null);
            // Check for a valid cursor
            if (pathCursor != null &&
                    pathCursor.moveToFirst()) {
                // Get the column index in the Cursor
                filenameIndex = pathCursor.getColumnIndex(
                        MediaStore.MediaColumns.DATA);
                // Get the full file name including path
                fileName = pathCursor.getString(filenameIndex);
                // Create a File object for the filename
                copiedFile = new File(fileName);
                // Return the parent directory of the file
                return new File(copiedFile.getParent());
             } else {
                // The query didn't work; return null
                return null;
             }
        }
    }
    ...
```
要了解有关从内容提供程序检索数据的详细信息，请参阅从提供程序检索数据一节。

