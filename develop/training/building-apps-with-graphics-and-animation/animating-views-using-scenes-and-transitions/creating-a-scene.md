# 创建场景

场景存储视图层次结构的状态，包括其所有视图及其属性值。 转换框架可以在开始和结束场景之间运行动画。 起始场景通常根据用户界面的当前状态自动确定。 对于结束场景，该框架能够帮助你从布局资源文件或代码中的一组视图来创建场景。

本课程将向你展示如何在应用程序中创建场景以及如何定义场景动作。 下一课将向你展示如何在两个场景之间进行转换。

> 注意：该框架可以在不使用场景的情况下对单个视图层次结构中的更改进行动画处理，如“[应用无场景转换](https://developer.android.com/training/transitions/transitions.html#NoScenes)”中所述。 然而，理解这一课是处理转换的关键。

## 从布局资源创建场景

你可以直接从布局资源文件创建一个 [Scene](https://developer.android.com/reference/android/transition/Scene.html) 实例。当文件中的视图层次结构大部分是静态时，可以使用此技术。创建的场景表示视图层次结构的状态。 如果你更改视图层次结构，则必须重新创建场景。 因为框架从文件的整个视图层次结构中创建场景，所以你不能拿布局文件中的一部分去创建场景。

要从布局资源文件创建 [Scene](https://developer.android.com/reference/android/transition/Scene.html) 实例，请从布局中 [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup.html) 实例检索，然后调用[Scene.getSceneForLayout（）](https://developer.android.com/reference/android/transition/Scene.html#getSceneForLayout) 方法，其中包含场景以及视图层次结构的布局文件和资源ID。

## 定义场景布局

本节其余部分的代码段将向你展示如何使用具有相同根元素的场景创建两个不同的场景。这些代码段还表明你可以加载多个不相关的 [Scene](https://developer.android.com/reference/android/transition/Scene.html) 对象，而不意味着它们彼此相关。

该示例由以下布局定义组成：

- 具有文本标签和子版面的活动的主要布局。
- 具有两个文本字段的第一个场景的相对布局。
- 具有相同两个文本字段却不同顺序的第二个场景的相对布局。
- 该示例被设计为，所有动画都发生在该活动的主布局的子布局中。主布局中的文本标签保持静态。

活动的主要布局定义如下：

res/layout/ activity_main.xml

	LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/master_layout">
	    <TextView
	        android:id="@+id/title"
	        ...
	        android:text="Title"/>
	    <FrameLayout
	        android:id="@+id/scene_root">
	        <include layout="@layout/a_scene" />
	    </FrameLayout>
	</LinearLayout>

此布局定义包含场景的文本字段和子版式。 第一个场景的布局包含在主布局文件中。 因为框架只能将整个布局文件加载到场景中，所以应用程序将其显示为初始化用户界面的一部分，并将其加载到场景中。

第一个场景的布局定义如下：

res/layout/a_scene.xml

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/scene_container"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent" >
	    <TextView
	        android:id="@+id/text_view1
	        android:text="Text Line 1" />
	    <TextView
	        android:id="@+id/text_view2
	        android:text="Text Line 2" />
	</RelativeLayout>

第二个场景的布局包含相同的两个文本字段（具有相同的ID）以不同的顺序放置，并定义如下：

res/layout/another_scene.xml

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/scene_container"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent" >
	    <TextView
	        android:id="@+id/text_view2
	        android:text="Text Line 2" />
	    <TextView
	        android:id="@+id/text_view1
	        android:text="Text Line 1" />
	</RelativeLayout>

## 从布局生成场景
为两个相对布局创建定义后，你可以为它们每一个获取一个场景。 这使你可以稍后在两个UI配置之间进行转换。要获得场景，你需要参考场景根布局和布局资源ID。

以下代码片段显示了如何获取对场景根的引用，并从布局文件创建两个Scene对象：

	Scene mAScene;
	Scene mAnotherScene;
	
	// Create the scene root for the scenes in this app
	mSceneRoot = (ViewGroup) findViewById(R.id.scene_root);
	
	// Create the scenes
	mAScene = Scene.getSceneForLayout(mSceneRoot, R.layout.a_scene, this);
	mAnotherScene =
	    Scene.getSceneForLayout(mSceneRoot, R.layout.another_scene, this);

在应用程序中，现在有两个基于视图层次结构的 [Scene](https://developer.android.com/reference/android/transition/Scene.html) 对象。 两个场景都使用 res / layout / activity_main.xml 中的 FrameLayout 元素定义的场景根布局。

## 在你的代码中创建一个场景
你也可以在 [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup.html) 对象的代码中创建一个 [Scene](https://developer.android.com/reference/android/transition/Scene.html) 实例。 在代码中直接修改视图层次结构或动态生成视图层次结构时，请使用此技术。

要从代码中的视图层次结构创建场景，请使用 
[Scene（sceneRoot，viewHierarchy）](https://developer.android.com/reference/android/transition/Scene.html#Scene)构造函数。 当你已经填充了布局，调用此构造函数等同于在调用 [Scene.getSceneForLayout（）](https://developer.android.com/reference/android/transition/Scene.html#getSceneForLayout) 方法。

以下代码片段演示了如何从代码中的场景根元素和场景的视图层次结构创建一个 [Scene](https://developer.android.com/reference/android/transition/Scene.html) 实例：

	Scene mScene;
	
	// Obtain the scene root element
	mSceneRoot = (ViewGroup) mSomeLayoutElement;
	
	// Obtain the view hierarchy to add as a child of
	// the scene root when this scene is entered
	mViewHierarchy = (ViewGroup) someOtherLayoutElement;
	
	// Create a scene
	mScene = new Scene(mSceneRoot, mViewHierarchy);

## 创建场景动作

该框架使你能够定义系统在进入或退出场景时运行自定义场景动作。在许多情况下，定义自定义场景动作是不必要的，因为框架会自动动画场景之间的变化。

场景动作对于处理这些情况很有用：

- 动画不在同一层次结构中的视图。你可以使用退出和输入场景动作作为起始和结束场景的动画。
- 动画视图，转换框架不能自动动画，例如 [ListView](https://developer.android.com/reference/android/widget/ListView.html) 对象。有关详细信息，请参阅[限制](https://developer.android.com/training/transitions/overview.html#Limitations)。

要提供自定义场景动作，请将你的操作定义为[可运行](https://developer.android.com/reference/java/lang/Runnable.html)对象，并将其传递给 [Scene.setExitAction（）](https://developer.android.com/reference/android/transition/Scene.html#setExitAction(java.lang.Runnable)) 或 [Scene.setExitAction（）](https://developer.android.com/reference/android/transition/Scene.html#setExitAction(java.lang.Runnable)) 方法。在运行转换动画之后，框架在运行转换动画之前调用 [Scene.setExitAction（）](https://developer.android.com/reference/android/transition/Scene.html#setExitAction(java.lang.Runnable)) 方法，然后在结束场景上运行 [Scene.setExitAction（）](https://developer.android.com/reference/android/transition/Scene.html#setExitAction(java.lang.Runnable)) 方法。

> 注意：不要在开始和结束场景视图之间传递数据时使用场景动作。有关更多信息，请参阅[定义转换生命周期回调](https://developer.android.com/training/transitions/transitions.html#Callbacks)。

>翻译：[JackWaiting](https://github.com/JackWaiting)    
> 审核：[@northJjL](https://github.com/northJjL)        
原始文档：[https://developer.android.com/training/transitions/scenes.html](https://developer.android.com/training/transitions/scenes.html)	