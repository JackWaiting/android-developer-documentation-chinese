# 支持不同屏幕

Android 使用两种常规属性来归类设备屏幕：尺寸和密度。你应当期望你的应用安装到有屏幕的设备上，能够适配其屏幕尺寸和密度。所以，你应当通过引入一些备选资源来优化你在不同屏幕尺寸和密度上的应用显示。

* 四种常规尺寸: 小, 一般, 大, 超大  
* 四种常规密度：低 (ldpi), 中等 (mdpi), 高 (hdpi), 超高 (xhdpi)

针对不同的屏幕的适配，你需要将这些不同的布局和图片的备选资源放置在单独的目录中，类似不同语言字符串的处理。

同样，你应当意识到屏幕的方向（横屏或竖屏）是被当做不同的屏幕尺寸，所以，很多应用需要针对不同的屏幕方向进行布局的用户体验优化。

## 创建不同的布局

为了优化不同屏幕尺寸上的用户体验，你应当针对你想要支持的的每一个屏幕尺寸创建一个唯一的布局 `XML` 文件。

To optimize your user experience on different screen sizes, you should create a unique layout XML file for each screen size you want to support. Each layout should be saved into the appropriate resources directory, named with a -<screen_size> suffix. For example, a unique layout for large screens should be saved under res/layout-large/.


> **Note**:




