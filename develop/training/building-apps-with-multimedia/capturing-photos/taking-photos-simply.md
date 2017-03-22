## 拍照

这一课将讲解如何使用应用相机拿取照片。

假设你正在做一个全球天气地图的服务，需要整合从客户端应用拿取到的天空照片。整合照片仅仅是应用的一个小功能。你想最方便的掉用拍照，而不是重新写一个相机模块。幸运的是大多数 Android 系统设备都最少安装有一个相机应用。这一课，将学习如何调用它去拍照。

## 请求相机权限

如果你的应用需要拍照功能，在 Google Play 上显示需要相机，去通知应用需要相机。添加 [<uses-feature>](https://developer.android.google.cn/guide/topics/manifest/uses-feature-element.html) 标签到清单文件中：

	<manifest ... >
	    <uses-feature android:name="android.hardware.camera"
	                  android:required="true" />
	    ...
	</manifest>

如果你的应用有使用，但是没有请求相机对应的功能，替换设置 android:required 为 false。Google Play 将允许设备没有相机去下载应用。应用有必要在运行时检查相机是否可用，通过调用 [hasSystemFeature(PackageManager.FEATURE_CAMERA)](https://developer.android.google.cn/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)) 方法，当不可用时应该让相机功能不可用。

## 获取照片通过相机应用

在 Android 系统上，通过调用 [Intent](https://developer.android.google.cn/reference/android/content/Intent.html) 描述所要做的事，将动作委托到另外一个应用。设置过程包括三点：Intent本身，调用开启一个 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html)，和当 Activity 返回时去处理图片和数据的代码。

下面的代码功能是调用 Intent 开启拍照。


    static final int REQUEST_IMAGE_CAPTURE = 1;
    
    private void dispatchTakePictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
        }
    }
    
注意这里的 startActivityForResult() 方法是受到 IF 条件保护的，[resolveActivity()](https://developer.android.google.cn/reference/android/content/Intent.html#resolveActivity(android.content.pm.PackageManager)) 方法会返回能够处理 Intent 的第一个 Activity 组件。做这个检查是非常必要的，因为如果调用 startActivityForResult() 执行的 Intent 没有一个应用能处理，应用将会闪退。所以只有当结果不为空时才执行才是安全的方式。


## 获取缩略图

打开相机拍照或许并不是你的目的，你可能想拿到从相机中拍到的照片并做一些处理。

系统相机应用会将照片编译成一个小的 [Bitmap](https://developer.android.google.cn/reference/android/graphics/Bitmap.html)，放在 [Intent](https://developer.android.google.cn/reference/android/content/Intent.html) 中最后通过 onActivityResult() 返回，通过键 "data" 获取。下面的恢复照片并显示到 [ImageView](https://developer.android.google.cn/reference/android/widget/ImageView.html) 上。

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
            Bundle extras = data.getExtras();
            Bitmap imageBitmap = (Bitmap) extras.get("data");
            mImageView.setImageBitmap(imageBitmap);
        }
    }
    
注意：从 *"data"* 中取到的缩略图可能适合用来做图标，并不适合做其它用途，其它用途请采用全尺寸图片。

## 保存全尺寸照片

当你设置要保存到一个文件时，系统相机会设置一个全尺寸相片。当相机保存图片时，你设置的必须是有效的文件名。

通常，用户拍摄的所有照片都应该是保存在公共的外部存储卡，因为它可以被所有应用访问。适当的分享照片目录通过 [getExternalStoragePublicDirectory()](https://developer.android.google.cn/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)) 方法的 [DIRECTORY_PICTURES](https://developer.android.google.cn/reference/android/os/Environment.html#DIRECTORY_PICTURES) 属性获取，因为方法提供的这个目录是所有应用共享的，读写这个目录需要分别请求 [READ_EXTERNAL_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 和 [WRITE_EXTERNAL_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 权限。写入权限中是包含读取权限的，当写存储卡时需要去请求这一个权限：

    <manifest ...>
        <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
        ...
    </manifest>
    
当然如果仅想对自己的应用有效，可用替换使用 [getExternalFilesDir()](https://developer.android.google.cn/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 提供的目录。在 Android 4.3 以及更低的版本上，写这个目录也是请求 WRITE_EXTERNAL_STORAGE 权限。从 Android 4.4 开始，不用在请求这个权限，因为这个目录其它应用不可访问。所以请求这个权限只需要在比较底的版本上，通过添加 [maxSdkVersion](https://developer.android.google.cn/guide/topics/manifest/uses-permission-element.html#maxSdk) 属性设置：

    <manifest ...>
        <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                         android:maxSdkVersion="18" />
        ...
    </manifest>
    
注：通过 [getExternalFilesDir()](https://developer.android.google.cn/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 和 [getFilesDir()](https://developer.android.google.cn/reference/android/content/Context.html#getFilesDir()) 获取的文件目录里保存的文件，在应用卸载的时候会被删除。

当确定了文件目录后，你需要创建独特的文件名，避免产生同名等冲突。你也可以保存上次使用的文件路径到一个成员变量中。如下例子是通过时间戳为一张新照片返回唯一名字的方法。


    String mCurrentPhotoPath;
    
    private File createImageFile() throws IOException {
        // Create an image file name
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String imageFileName = "JPEG_" + timeStamp + "_";
        File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
        File image = File.createTempFile(
            imageFileName,  /* prefix */
            ".jpg",         /* suffix */
            storageDir      /* directory */
        );
    
        // Save a file: path for use with ACTION_VIEW intents
        mCurrentPhotoPath = image.getAbsolutePath();
        return image;
    }
    
使用这个方法可以为照片创建一个有效的文件，如下方式可以创建和调用 Intent：


    static final int REQUEST_TAKE_PHOTO = 1;
    
    private void dispatchTakePictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        // Ensure that there's a camera activity to handle the intent
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            // Create the File where the photo should go
            File photoFile = null;
            try {
                photoFile = createImageFile();
            } catch (IOException ex) {
                // Error occurred while creating the File
                ...
            }
            // Continue only if the File was successfully created
            if (photoFile != null) {
                Uri photoURI = FileProvider.getUriForFile(this,
                                                      "com.example.android.fileprovider",
                                                      photoFile);
                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
                startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
            }
        }
    }

注：我们使用 [getUriForFile(Context, String, File)](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context, java.lang.String, java.io.File)) 会返回 content:// URI。在 Android 7.0 （API 24）以及更高的版本上，返回的 file:// URI 导致产生 [FileUriExposedException](https://developer.android.google.cn/reference/android/os/FileUriExposedException.html)。因此，现在更多的通用方式是使用 [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html) 来储存。

现在，你需要去配置 [FileProvider](https://developer.android.google.cn/reference/android/support/v4/content/FileProvider.html)，在应用的清单文件中，添加一个文件提供者到应用中：

    <application>
       ...
       <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.android.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"></meta-data>
        </provider>
        ...
    </application>
    
确定认证字符串是否匹配第二个参数使用 getUriForFile(Context, String, File)。在文件提供者定义的 meta-data 块中，你能看到在专用的资源文件 res/xml/file_paths.xml 中配置的合法路径。下面是请求内容代表的例子：

    <?xml version="1.0" encoding="utf-8"?>
    <paths xmlns:android="http://schemas.android.com/apk/res/android">
        <external-path name="my_images" path="Android/data/com.example.package.name/files/Pictures" />
    </paths>

路径组件与通过 [getExternalFilesDir()](https://developer.android.google.cn/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 方法的 [Environment.DIRECTORY_PICTURES](https://developer.android.google.cn/reference/android/os/Environment.html#DIRECTORY_PICTURES) 拿取的路径是一致的。确定替换的 com.example.package.name 是你应用的真实包名。另外，检查和 external-path 有关的文件与对应的 FileProvider 扩展描述。

## 添加照片到相册：

通过意图创建一张照片，你可以知道图片是本地的，因为你在第一个地方去保存它。至于其它，推测最简单的方式是从系统媒体提供者中拿取照片。

注：如果你保存照片的路径是通过 getExternalFilesDir() 获取的，那么系统媒体检查器不能访问到这些文件，因为这个目录是私有的。

下面的例子是展示如何去调用系统媒体扫描器，并添加照片到媒体提供者数据库，使它在系统相册和其它应用生效。

    private void galleryAddPic() {
        Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
        File f = new File(mCurrentPhotoPath);
        Uri contentUri = Uri.fromFile(f);
        mediaScanIntent.setData(contentUri);
        this.sendBroadcast(mediaScanIntent);
    }
    
## 解析扫描到的图片

管理全尺寸的图片要当心受限的内存。你会发现显示仅仅一点的图片就会出现内存泄露，你可以通过扩展的 JPEG 导入到内存中，可以显著的减少动态堆的使用，它会缩放图片得到一个与显示一致的尺寸。下面的例子展示这个技术。


    private void setPic() {
        // Get the dimensions of the View
        int targetW = mImageView.getWidth();
        int targetH = mImageView.getHeight();
    
        // Get the dimensions of the bitmap
        BitmapFactory.Options bmOptions = new BitmapFactory.Options();
        bmOptions.inJustDecodeBounds = true;
        BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
        int photoW = bmOptions.outWidth;
        int photoH = bmOptions.outHeight;
    
        // Determine how much to scale down the image
        int scaleFactor = Math.min(photoW/targetW, photoH/targetH);
    
        // Decode the image file into a Bitmap sized to fill the View
        bmOptions.inJustDecodeBounds = false;
        bmOptions.inSampleSize = scaleFactor;
        bmOptions.inPurgeable = true;
    
        Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
        mImageView.setImageBitmap(bitmap);
    }


>翻译：[@iOnesmile](https://github.com/iOnesmile)    
原始文档：<https://developer.android.google.cn/training/camera/photobasics.html>
