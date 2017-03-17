# 打印照片
拍照和分享照片是移动设备最常用的功能之一。如果在我们的应用中需要拍照并且展示照片，或者允许用户共享图片，那么就需要考虑在应用中可以打印这些图片。[The Android Support Library](https://developer.android.google.cn/topic/libraries/support-library/index.html) 提供了一个方便的功能，只需要使用少量的代码和简单的打印布局选择设置就可以进行图片打印。    

这节课就是来讲述如何使用 V4 支持库中的 [PrintHelper](https://developer.android.google.cn/reference/android/support/v4/print/PrintHelper.html) 类来打印一张图片。

##打印图片

[PrintHelper](https://developer.android.google.cn/reference/android/support/v4/print/PrintHelper.html) 是 Android 支持的库类，它提供了一种简单的打印图片的方式。这个类有一个单一的布局选项：[setScaleMode](https://developer.android.google.cn/reference/android/support/v4/print/PrintHelper.html#setScaleMode(int))，它支持下面二种打印选项的其中一个：

* [SCALE_MODE_FIT](https://developer.android.google.cn/reference/android/support/v4/print/PrintHelper.html#SCALE_MODE_FIT)-这个选项会调整图片大小，以便整个图片可以显示在打印区域内。       
* [SCALE_MODE_FILL](https://developer.android.google.cn/reference/android/support/v4/print/PrintHelper.html#SCALE_MODE_FILL)-这个选项会调整图片比例，可以全部填充打印范围。选择这个选项意味着顶部和底部，或者左侧和右侧的边缘是不能打印的。如果不设置图片的布局选项，这个选项就是默认的。    

setScaleMode() 的二个选项都可以保证图片原始比例的完整性。接下来的代码样例将展示如何创建 PrintHelper 类的实例，设置布局选项并开始打印进程：

```java
private void doPhotoPrint() {
    PrintHelper photoPrinter = new PrintHelper(getActivity());
    photoPrinter.setScaleMode(PrintHelper.SCALE_MODE_FIT);
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(),
            R.drawable.droids);
    photoPrinter.printBitmap("droids.jpg - test print", bitmap);
}
```

该方法可以被看作一个菜单项的操作。注意那些设备不一定支持的菜单项（如打印）应该放在菜单中的“更多”中。有关更新多信息，请参阅 [Action Bar](https://developer.android.google.cn/design/patterns/actionbar.html) 设计指南。  
  
在调用 [printBitmap()](https://developer.android.google.cn/reference/android/support/v4/print/PrintHelper.html#printBitmap(java.lang.String, android.graphics.Bitmap)) 后，应用不需要再进行其他操作了。Android 打印用户界面会随后出现，允许用户选择打印机和打印选项。用户可以选择打印图片或者取消打印。如果用户选择了打印图片，则会创建打印任务并在通知栏中显示打印通知。    

如果你想要打印的不仅仅是图片，你必须构建一个打印文档。有关打印文档的信息，请参阅 [Printing an HTML Document](https://developer.android.google.cn/training/printing/html-docs.html) 或 [Printing Custom Documents](https://developer.android.google.cn/training/printing/custom-docs.html)。

>翻译：[@misparking](https://github.com/misparking)   
>原始文档：<https://developer.android.google.cn/training/printing/photos.html#image>  