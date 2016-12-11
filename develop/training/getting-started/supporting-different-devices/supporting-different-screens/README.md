# 支持不同屏幕

Android 使用两种常规属性来归类设备屏幕：尺寸和密度。你应当期望你的应用安装到有屏幕的设备上，能够适配其屏幕尺寸和密度。所以，你应当通过引入一些备选资源来优化你在不同屏幕尺寸和密度上的应用显示。

* 四种常规尺寸: 小, 一般, 大, 超大  
* 四种常规密度：低 (ldpi), 中等 (mdpi), 高 (hdpi), 超高 (xhdpi)

针对不同的屏幕的适配，你需要将这些不同的布局和图片的备选资源放置在单独的目录中，类似不同语言字符串的处理。

同样，你应当意识到屏幕的方向（横屏或竖屏）是被当做不同的屏幕尺寸，所以，很多应用需要针对不同的屏幕方向进行布局的用户体验优化。

## 创建不同的布局

为了优化不同屏幕尺寸上的用户体验，你应当针对你想要支持的的每一个屏幕尺寸创建一个唯一的布局 `XML` 文件。每一个布局文件应当被放置到正确的资源目录，以 `-<屏幕尺寸>` 的后缀来命名。例如，针对大屏幕的唯一布局应当存在 `res/layout-large/` 目录底下。

> **备注**：Android 会自动的调整你布局的比例，来正确的适配屏幕。所以，你不必担心不同屏幕尺寸上布局中的 UI 元素的绝对尺寸，反而需要注意的是关注影响用户体验的布局结构（例如重要视图之间的尺寸或位置）。

例如，这个工程包含一个默认的布局和一个针对大屏幕的备用布局：

```
MyProject/
    res/
        layout/
            main.xml
        layout-large/
            main.xml
```

文件名称必须完全一样，但是他们的内容是不同的，根据相关屏幕尺寸提供不同优化的 UI。

通常以以下方法在你的应用中简单的引入布局文件：

``` Java
@Override
 protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.main);
}
```

当你的应用运行时，系统会自动根据当前屏幕尺寸从布局目录中载入正确的布局文件。更多关于 Android 如何选择合适资源的信息，可以参见[资源提供向导](https://developer.android.com/guide/topics/resources/providing-resources.html#BestMatch)。

另外一个例子，这里有一个工程，有针对横屏的备用布局：

```
MyProject/
    res/
        layout/
            main.xml
        layout-land/
            main.xml
```

默认情况下，`layout/main.xml` 文件是针对竖屏的。

如果你想针对横屏并且是大屏幕提供一个指定的布局，你就需要同时使用 `large` 和 `land` 标签：

```
MyProject/
    res/
        layout/              # default (portrait)
            main.xml
        layout-land/         # landscape
            main.xml
        layout-large/        # large (portrait)
            main.xml
        layout-large-land/   # large landscape
            main.xml
```

> **备注**：Android 3.2 及以上系统，支持一个先进的方法来定义屏幕尺寸，它支持基于最小的以 `dip` 为单位的宽度和高度，来指定屏幕尺寸资源。这篇教程不包含此新技术。更多信息，请参阅[多屏幕设计](https://developer.android.com/training/multiscreen/index.html)。

## 创建不同的图片

你应当针对低、中、高和超高密度的屏幕分别提供正确缩放比例的图片。这会帮助你在所有的屏幕密度上获得好的图片质量和性能。

你应当通过矢量格式的原始资源来生成对应目睹的以下尺寸比例的图片：

* 超高密度: 2.0
* 高密度: 1.5
* 中等密度: 1.0 (基准)
* 低密度: 0.75

这就意味着，如果你针对超高密度设备生成的是一个 200x200 尺寸图片的话，那么针对高密度、中等密度和低密度，需要分别生成 150x150、100x100、75x75 尺寸的图片。

然后，将这些图片文件放置到正确的图片资源目录中：

```
MyProject/
    res/
        drawable-xhdpi/
            awesomeimage.png
        drawable-hdpi/
            awesomeimage.png
        drawable-mdpi/
            awesomeimage.png
        drawable-ldpi/
            awesomeimage.png
```

任何时候，只要你引用 `@drawable/awesomeimage` 资源，系统会根据当前的屏幕密度选择合适的图片。

> **备注**：低密度（ldpi）资源不一定总是必须的。当你只提供高分辨率资源，系统将会自动缩放 1.5 倍来适配低分辨率屏幕。

更多关于创建应用图标资源细节和指导原则，请参见[图标设计指导](https://material.google.com/style/icons.html)。

备注：  
翻译：[@ifeegoo](https://github.com/ifeegoo)  
原始文档：https://developer.android.com/training/basics/supporting-devices/screens.html
