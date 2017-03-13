# 添加一个简单的分享动作
在 Android 4.0(API Level 14) 引入的 [ActionProvider](https://developer.android.google.cn/reference/android/view/ActionProvider.html) 是你更容易的在 [AcionBar](https://developer.android.google.cn/reference/android/app/ActionBar.html) 实现一个有效并且友好的分享动作。[ActionProvider](https://developer.android.google.cn/reference/android/view/ActionProvider.html) 一但附加到 action bar 的菜单选项中，就会去处理该选项的外观和行为。在使用 [ShareActionProvider](https://developer.android.google.cn/reference/android/widget/ShareActionProvider.html) 的情况下，你只需要提供意图他会做其他的。 
>**Note:** ShareActionProvider 是在API Level 14 以及更高的版本提供的。

## 更新菜单声明
开始使用 [ShareActionProvider](https://developer.android.google.cn/reference/android/widget/ShareActionProvider.html)，在你的 [menu resource](https://developer.android.google.cn/guide/topics/resources/menu-resource.html) 文件中为相应的 `<item>` 定义 `android:actionProviderClass` 属性:
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
            android:id="@+id/menu_item_share"
            android:showAsAction="ifRoom"
            android:title="Share"
            android:actionProviderClass=
                "android.widget.ShareActionProvider" />
    ...
</menu>
```
项目的外观和行为将委托给 [ShareActionProvider](https://developer.android.google.cn/reference/android/widget/ShareActionProvider.html)。但是你需要告诉提供者你想要分享。

## 设置分享的意图
在 [ShareActionProvider](https://developer.android.google.cn/reference/android/widget/ShareActionProvider.html) 的顺序方法中，你必须提供一个分享意图。这个分享意图的描述应该是相同的与 [Sending Simple Data to Other Apps](https://developer.android.google.cn/training/sharing/send.html) 课程中，有 [ACTION_SEND](https://developer.android.google.cn/reference/android/content/Intent.html#ACTION_SEND) 的动作和添加 [EXTRA_TEXT](https://developer.android.google.cn/reference/android/content/Intent.html#EXTRA_TEXT) 和 [EXTRA_STREAM](https://developer.android.google.cn/reference/android/content/Intent.html#EXTRA_STREAM) 附加数据。分配分享意图，首先发现相应的 [MenuItem](https://developer.android.google.cn/reference/android/view/MenuItem.html) 当你的菜单资源文件填充了你的 [Activity](https://developer.android.google.cn/reference/android/app/Activity.html) 或者 [Fragment](https://developer.android.google.cn/reference/android/app/Fragment.html) 。然后，调用 [MenuItem.getActionProvider()](https://developer.android.google.cn/reference/android/view/MenuItem.html#getActionProvider()) 来检索 [ShareActionProvider](https://developer.android.google.cn/reference/android/widget/ShareActionProvider.html) 的实例。使用 [setShareIntent()](https://developer.android.google.cn/reference/android/widget/ShareActionProvider.html#setShareIntent(android.content.Intent)) 去更新与该操作关联的分享意图。这里是个例子：

```java
private ShareActionProvider mShareActionProvider;
...

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu resource file.
    getMenuInflater().inflate(R.menu.share_menu, menu);

    // Locate MenuItem with ShareActionProvider
    MenuItem item = menu.findItem(R.id.menu_item_share);

    // Fetch and store ShareActionProvider
    mShareActionProvider = (ShareActionProvider) item.getActionProvider();

    // Return true to display menu
    return true;
}

// Call to update the share intent
private void setShareIntent(Intent shareIntent) {
    if (mShareActionProvider != null) {
        mShareActionProvider.setShareIntent(shareIntent);
    }
}
```
你可能需要在创建菜单只设置分享意图一次，或者你可能希望设置的时候然后去更新 UI 的改变。例如，当你在图库应用中全屏查看照片时， 当浏览的图片改变时分享意图也需要改变。

有关 [ShareActionProvider](https://developer.android.google.cn/reference/android/widget/ShareActionProvider.html) 对象的进一步讨论，请参阅 [Action Views and Action Providers](https://developer.android.google.cn/training/appbar/action-views.html)。

>翻译：[@edtj](https://github.com/edtj)    
原始文档：<https://developer.android.google.cn/training/basics/firstapp/creating-project.html>
