# 支持不同平台版本

尽管最新的 Android 版本给你的应用提供极好的 API，但是也仍然需要继续支持旧的 Android 版本，直到更多的设备更新到最新的系统。这节课将教你在使用最新的 API 的同时，继续支持旧版本的系统。


更新的平台版本的[表格](http://developer.android.com/about/dashboards/index.html)展示了基于访问 Google Play 商店设备的数量，以及运行在活跃设备上的每一个 Android 版本的分布情况。通常，支持 约 90% 的活跃设备，以及保证你的应用采用最新的版本编译，是一个良好的习惯。

> **备注**：为了能够针对一些 Android 版本提供最好的特性和功能，你应当在应用中使用 [Android Support Library](https://developer.android.com/tools/support-library/index.html)，他可以让你在旧版本的平台上使用一些最新的平台 API。


## 指定最新和目标 API 级

[AndroidManifest.xml](https://developer.android.com/guide/topics/manifest/manifest-intro.html) 文件中描述了你的应用细节和标识了支持的 Android 版本。一般通过 `<uses-sdk>` 元素的 `minSdkVersion` 和 `targetSdkVersion` 属性来分别指定你应用适配的最低版本的 API 以及你设计和测试应用的最高版本 API。

例如：

``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" ... >
    <uses-sdk android:minSdkVersion="4" android:targetSdkVersion="15" />
    ...
</manifest>
```

新的 Android 版本发布的时候，一些样式和特征可能会发生变化。为了允许你的应用使用这些变化，并且确保你的应用适配每一个用户设备的样式，你应当将 [`targetSdkVersion`](https://developer.android.com/guide/topics/manifest/uses-sdk-element.html#target) 的值设置成当前最新可用的 Android 版本。

## 运行时检测系统版本

Android 在 [Build](https://developer.android.com/reference/android/os/Build.html) 常量类中提供了针对每一个平台版本的唯一代码。 请在你的应用中使用这些代码来构建条件，以确保依赖于更高级别的 API 的代码在当前系统可用的情况下执行。

```
private void setUpActionBar() {
    // Make sure we're running on Honeycomb or higher to use ActionBar APIs
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        ActionBar actionBar = getActionBar();
        actionBar.setDisplayHomeAsUpEnabled(true);
    }
}
```

> **备注**：Android 解析 XML 资源的时候，会忽略当前设备不支持的 XML 属性。所以你可以安全的使用只有新版本的系统才支持的 XML 属性，而不必担心旧版本的系统因为这些代码而挂掉。例如，如果你设置 `targetSdkVersion="11"`，那么你的应用会在 Android 3.0 或者以上版本默认包含 [ActionBar](https://developer.android.com/reference/android/app/ActionBar.html) 。然后如果你需要在 Action Bar 上添加目录条目，你需要在你的目录 XML 资源中设置 `android:showAsAction="ifRoom"`，在跨版本的 XML 文件中这么做是安全的，因为旧版本的 Andorid 系统会简单的忽略 `showAsAction ` 属性。（也就是说，你不需要再在 `res/menu-v11/` 目录底下单独的给出一个 xml 文件）。

## 使用平台样式和主题

Android 提供了让你的应用有潜在的操作系统的视觉和感觉的用户体验主题。这些主题可以通过主配置文件加入到你的应用。通过使用这些内置的样式和主题，你的应用将会自然跟随每一个最新版本的 Android 系统的视觉和感觉设计。

让你的 Activity 看起来像一个对话框：

```
<activity android:theme="@android:style/Theme.Dialog">
```

让你的 Activity 背景透明：

```
<activity android:theme="@android:style/Theme.Translucent">
```

载入通过 `/res/values/styles.xml` 文件自定义的主题：


```
<activity android:theme="@style/CustomTheme">
```

在 `<application>` 元素中添加 `android:theme` 让你整个应用（所有 Activity）都是用同一个主题：

```
<application android:theme="@style/CustomTheme">
```

更多关于创建和使用主题的信息，请阅读[样式和主题](https://developer.android.com/guide/topics/ui/themes.html)指导。

备注：  
翻译：[@ifeegoo](https://github.com/ifeegoo)  
原始文档：https://developer.android.com/training/basics/supporting-devices/platforms.html
