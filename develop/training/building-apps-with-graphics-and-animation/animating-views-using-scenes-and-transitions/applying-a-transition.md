## 引用过渡动画

在过渡框架中，动画创建一系列框架，描绘起始和结束场景中的视图层次结构之间的变化。框架将这些动画描绘成包含动画信息的过渡对象。 要运行动画，你可以向过渡管理器提供使用过渡和结束场景。

本课程教你如何使用内置的过渡框架来在两个场景之间运行一个动画来移动，调整大小和淡化视图。 下一课向你展示如何自定义过渡动画。

### 创建过渡动画

在上一课中，学习了如何创建表示不同视图层次结构状态的场景。 一旦定义了要在其间切换的起始场景和结束场景，则需要创建一个定义动画的 [Transition](https://developer.android.google.cn/reference/android/transition/Transition.html) 对象。 该框架使你能够在资源文件中指定内置的过渡，并在代码中加载它，或直接在代码中创建内置过渡类的实例。

表 1，内置过渡类型。

Class |	Tag  | 	Attributes |	Effect
-------|--------|---------------|----------------------------
[AutoTransition](https://developer.android.google.cn/reference/android/transition/AutoTransition.html) | 	\<autoTransition/\> | 	- |	Default transition. Fade out, move and resize, and fade in views, in that order.
[Fade](https://developer.android.google.cn/reference/android/transition/Fade.html) |	\<fade/\> |	android:fadingMode="<br>[*fade\_in* &#124; <br>*fade\_out* &#124; <br>*nfade\_in\_out*]" |	*fade\_in* fades in views <br> *fade\_out* fades out views <br> *fade\_in\_out* (default) does a fade\_out followed by a fade\_in.
[ChangeBounds](https://developer.android.google.cn/reference/android/transition/ChangeBounds.html) |	\<changeBounds/\> |	- |	Moves and resizes views.


### 从资源文件创建过渡实例

这种技术使你能够修改过渡效果的定义，而无需更改 Activity 中的代码。 这种技术也可用于将复杂的过渡效果定义与应用程序代码分开，如 “[Specify Multiple Transitions](https://developer.android.google.cn/training/transitions/transitions.html#Multiple)” 所示。

要在资源文件中指定内置的过渡，请按照下列步骤操作：

1. 将 res/transition/ 目录添加到项目中。

2. 在此目录中创建一个新的 XML 资源文件。

3. 添加一个内置过渡效果的 XML 节点。

例如，以下资源文件指定了 [Fade](https://developer.android.google.cn/reference/android/transition/Fade.html) 过渡：

res/transition/fade_transition.xml

```xml
<fade xmlns:android="http://schemas.android.com/apk/res/android" />
```	
	
以下代码片段显示了如何从资源文件中充实活动中的Transition实例：

```java
Transition mFadeTransition =
        TransitionInflater.from(this).
        inflateTransition(R.transition.fade_transition);
```


### 在代码中创建一个过渡实例

如果你在代码中的修改用户界面，并且创建简单的内置过渡实例（很少或没有参数），则此技术对于动态创建过渡对象很有用。

要创建内置过渡的实例，请调用 [Transition](https://developer.android.google.cn/reference/android/transition/Transition.html) 类的子类中的一个公共构造函数。 例如，以下代码片段创建了一个 [Fade](https://developer.android.google.cn/reference/android/transition/Fade.html) 过渡效果的实例：

```java
Transition mFadeTransition = new Fade();
```

### 引用过渡动画

响应不同视图层次结构之间的事件触发（例如用户操作），通常可以通过引用过渡效果来实现。

通常使用过渡效果的场景是在不同的视图层次结构之间响应事件操作
你通常应用过渡以在响应于事件（例如用户操作）的不同视图层次结构之间进行更改。 例如，考虑一个搜索应用程序：当用户输入搜索字词并点击搜索按钮时，应用程序将界面展示更改为搜索结果的布局场景，同时搜索按钮渐变淡出，且搜索结果渐变淡入的过渡效果。

要对 Activity 中的某些事件应用过渡效果并产生场景变化时，请调用 [TransitionManager.go()](https://developer.android.google.cn/reference/android/transition/TransitionManager.html#go(android.transition.Scene)) 静态方法，并使用结尾场景和过渡实例来使用动画，如以下代码片段所示：

```java
TransitionManager.go(mEndingScene, mFadeTransition);
```

在运行由过渡实例指定的动画的情况下，框架将从场景根目录中的视图层次结构更改为结束场景中的视图层次结构。 起始场景是上一次过渡动效的结束场景。 如果没有以前的过渡动效，则从用户界面的当前状态自动确定起始场景。

如果不指定过渡实例，过渡管理器可以应用自动过渡，这对大多数情况都是合理的。 有关更多信息，请参阅 [TransitionManager](https://developer.android.google.cn/reference/android/transition/TransitionManager.html) 类的 API 参考。


## 选择具体目标视图

默认情况下，该框架将过渡应用于起始和结束场景中的所有视图。 在某些情况下，你可能只想将动画应用于场景中的一部分视图。 例如，框架不支持 [ListView](https://developer.android.google.cn/reference/android/widget/ListView.html) 对象的动画变化，因此就不用再尝试在过渡动画期间为它们设置动画。该框架使你能够选择需要进行动画的特定视图。

过渡动画的每个视图称为目标。你只能选择与场景相关联的视图层次结构的一部分做为目标。

要从目标列表中删除一个或多个视图，请在开始过渡之前调用 [removeTarget()](https://developer.android.google.cn/reference/android/transition/Transition.html#removeTarget) 方法。 仅将你指定的视图添加到目标列表，请调用 [addTarget()](https://developer.android.google.cn/reference/android/transition/Transition.html#addTarget(android.view.View)) 方法。 有关更多信息，请参阅 [Transition](https://developer.android.google.cn/reference/android/transition/Transition.html) 类的API参考。


## 指定多个过渡

如果想达到比较好的动画效果，应该将各个场景之间发生的动画变化类型进行匹配。例如，如果要删除某些视图并在场景之间添加其他视图，则动画中的淡出/淡入可以表示某些视图不再可用。如果你将视图移动到屏幕上的不同点，更好的选择是为其设置动画，以便用户注意到视图的新位置。

你不必仅仅选择一个动画，因为过渡框架使你能够将动画效果组合在包含一组单独的内置或自定义过渡动画中。

要从XML中的过渡集合中定义过渡集，请在 *res / transitions /* 目录中创建资源文件，并列出 *transitionSet* 元素下的过渡效果。例如，以下代码段显示了如何指定与 [AutoTransition](https://developer.android.google.cn/reference/android/transition/AutoTransition.html) 类具有相同行为的过渡集：
	
	<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
	    android:transitionOrdering="sequential">
	    <fade android:fadingMode="fade_out" />
	    <changeBounds />
	    <fade android:fadingMode="fade_in" />
	</transitionSet>
	
要将代码中的 [TransitionSet](https://developer.android.google.cn/reference/android/transition/TransitionSet.html) 对象充满过渡集，请调用活动中的 [TransitionInflater.from（）](https://developer.android.google.cn/reference/android/transition/TransitionInflater.html#from(android.content.Context)) 方法。 TransitionSet 类从 Transition 类扩展，因此你可以像过渡实例一样使用过渡管理器。


## 应用没有场景的过渡

更改视图层次结构不是修改用户界面的唯一方法。你还可以通过在当前层次结构中添加，修改和删除子视图进行更改。例如，你可以仅使用单个布局实现搜索交互。从显示搜索条目栏和搜索图标的布局开始。要更改用户界面以显示结果，当用户通过调用 [ViewGroup.removeView（） ](https://developer.android.google.cn/reference/android/view/ViewGroup.html#removeView(android.view.View))方法单击它时，删除搜索按钮，并通过调用 [ViewGroup.addView（）](https://developer.android.google.cn/reference/android/view/ViewGroup.html#addView(android.view.View)) 方法添加搜索结果。

如果替代方案有两个几乎相同的层次结构，则可能需要使用这种方法。不必在用户界面中创建和维护两个单独的布局文件，而是可以在代码中修改一个包含视图层次结构的布局文件。

如果你以此方式在当前视图层次结构中进行更改，则无需创建场景。相反，你可以使用延迟过渡创建并应用视图层次结构的两个状态之间的过渡。过渡框架的此功能以当前视图层次结构状态开始，记录对其视图所做的更改，并在系统重新绘制用户界面时应用一个动画变化的过渡。

要在单个视图层次结构中创建延迟过渡，请按照下列步骤操作：

1. 当触发过渡的事件发生时，调用 [TransitionManager.beginDelayedTransition（）](https://developer.android.google.cn/reference/android/transition/TransitionManager.html#beginDelayedTransition(android.view.ViewGroup)) 方法，提供要更改的所有视图的父视图以及要使用的过渡。框架存储子视图及其属性值的当前状态。

2. 根据用例的要求更改子视图。框架记录你对子视图及其属性所做的更改。

3. 当系统根据你的更改重新绘制用户界面时，框架将动画化原始状态和新状态之间的变化。

以下示例显示了如何使用延迟过渡将文本视图添加到视图层次结构中。第一个片段显示布局定义文件：

res/layout/activity_main.xml

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <EditText
        android:id="@+id/inputText"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    ...
</RelativeLayout>
```

下一个代码段显示了添加文本视图的代码：

MainActivity.java

```java
private TextView mLabelText;
private Fade mFade;
private ViewGroup mRootView;
...

// Load the layout
this.setContentView(R.layout.activity_main);
...

// Create a new TextView and set some View properties
mLabelText = new TextView();
mLabelText.setText("Label").setId("1");

// Get the root view and create a transition
mRootView = (ViewGroup) findViewById(R.id.mainLayout);
mFade = new Fade(IN);

// Start recording changes to the view hierarchy
TransitionManager.beginDelayedTransition(mRootView, mFade);

// Add the new TextView to the view hierarchy
mRootView.addView(mLabelText);

// When the system redraws the screen to show this update,
// the framework will animate the addition as a fade in
```

## 定义过渡动画生命周期回调

过渡动画生命周期类似于 activity 生命周期。它表示框架在调用 [TransitionManager.go（）](https://developer.android.google.cn/reference/android/transition/TransitionManager.html#go(android.transition.Scene)) 方法和完成动画之间的时间期间监视的过渡状态。在重要的生命周期状态下，框架调用由 [TransitionListener](https://developer.android.google.cn/reference/android/transition/Transition.TransitionListener.html) 接口定义的回调。

过渡生命周期回调是有用的，例如，在场景更改期间将视图属性值从起始视图层次结构复制到结束视图层次结构。你不能简单地将值从起始视图复制到结束视图层次结构中的视图，因为在过渡动画完成之前结束视图层次结构不能增大。相反，你需要将值存储在变量中，然后在框架完成过渡后将其复制到结束视图层次结构中。要在过渡完成时收到通知，可以在你的 Activity 中实现 [TransitionListener.onTransitionEnd（）](https://developer.android.google.cn/reference/android/transition/Transition.TransitionListener.html#onTransitionEnd(android.transition.Transition)) 方法。

有关更多信息，请参阅 TransitionListener 类的 API 参考。



>翻译：[@iOnesmile](https://github.com/iOnesmile) 
>审核：[@misparking](https://github.com/misparking) 
原始文档：<https://developer.android.google.cn/training/transitions/transitions.html#Callbacks>
