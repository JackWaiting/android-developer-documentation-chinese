# 

# 暂停和继续 Activity#

在使用应用期间，应用有时会失去焦点，同时会导致 Activity 暂停。例如，当应用运行在 [多窗口模式](https://developer.android.com/guide/topics/ui/multi-window.html) 下，只会有一个应用拥有焦点，其它应用都会被系统暂停。同样的，当一个透明的 Activity 打开（如：对话框），被覆盖的 Activity 将会暂停，在 Activity 被挡住时会一直没有焦点，它保持暂停状态。    

然而，一旦 Activity 被完全挡住并且不可见，它会是 **停止** 状态（在下一课讨论）。     

当窗体进入到暂停状态，系统会回调 Activity 的 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 方法，在这个方法中，你可以停止不应该在暂停时继续运行的操作，或持久化相关信息在用户离开应用时。如果用户在   Activity 暂停状态下返回，系统会恢复 Activity 并调用 [onResume()](https://developer.android.com/reference/android/app/Activity.html#onResume()) 方法。    

> 注：在系统调用 Activity 的  [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 方法时，可能表示 Activity 会暂停一会之后再次获取焦点，或应用在多窗口模式；也有可能这个方法被调用后用户退出了 Activity。   

![生命周期图](https://developer.android.com/images/training/basics/basic-lifecycle-paused.png)   


**图1**，当半透明的窗体挡住 Activity时，系统会回调   [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 方法，这时 Activity 保持在 onPause()① 状态，当用户返回时系统回调  [onResume()](https://developer.android.com/reference/android/app/Activity.html#onResume())②。    


## 暂停 Activity ##

当系统调用 Activity 的 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 时，从技术上来说它是部分可见。但是大多数情况下是用户将要离开 Activity，马上会变成停止状态。你通常应该使用 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 回调：   

- 检查 Activity 是否可见。如果不可见，请停止动画或其他正在进行的可能消耗 CPU 的操作。记住，从 Android 7.0 开始，暂停的应用可能会在多窗口模式下运行。 在本示例中，你可能不想停止动画或视频播放。   

- 释放系统资源，比如广播接收器、传感器手柄（如 GPS），或当你的 Activity 暂停时用户不需要它们且可能影响电池寿命的其他资源。   

例如，如果你使用 [Camera](https://developer.android.com/reference/android/hardware/Camera.html)，[onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 方法是释放它的好位置。   


	@Override
	public void onPause() {
	    super.onPause();  // 第一步总是回调父类的方法
	
	    // 释放相机资源，因为在暂停时不需要它
	    // 并且其它 Activity 可能需要使用它
	    if (mCamera != null) {
	        mCamera.release();
	        mCamera = null;
	    }
	}


一般情况下，你不应该使用 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 永久性存储用户更改（比如输入表单的个人信息）。 只有在你确定用户希望自动保存这些更改的情况（比如，草拟电子邮件时）下，才能在 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 中永久性存储用户更改。但你应避免在 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 期间执行 CPU 密集型工作，比如向数据库写入信息，因为这会拖慢向下一 Activity 过渡的过程（你应改为在 [onStop()](https://developer.android.com/reference/android/app/Activity.html#onStop()) 间执行高负载关机操作）。     

你应通过相对简单的方式在 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 方法中完成大量操作，这样才能加快在你的 Activity 确实停止的情况下用户向下一个目标过渡的速度。   

> 注：当你的 Activity 暂停时，[Activity](https://developer.android.com/reference/android/app/Activity.html) 实例将驻留在内存中并且在 Activity 继续时被再次调用。你无需重新初始化在执行任何导致进入“继续”状态的回调方法期间创建的组件。   



## 继续 Activity ##

当用户从“暂停”状态回到 Activity 时，系统会调用 [onResume()](https://developer.android.com/reference/android/app/Activity.html#onResume()) 方法。   

请注意，每当你的 Activity 进入前台时系统便会调用此方法，包括它初次创建之时。 同样地，你应实现 [onResume()](https://developer.android.com/reference/android/app/Activity.html#onResume()) 以初始化你在 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 期间释放的组件，并执行每当 Activity 进入“继续”状态时必须进行的任何其他初始化操作（比如开始动画和初始化只在 Activity 具有用户焦点时使用的组件）。   

[onResume()](https://developer.android.com/reference/android/app/Activity.html#onResume()) 的以下示例对应于以上的 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 示例，因此它初始化 Activity 暂停时释放的照相机。   

	@Override
	public void onResume() {
	    super.onResume();  // 第一步总是回调父类的方法
	
	    // 获取相机实例
	    if (mCamera == null) {
	        initializeCamera(); // 调用本地方法初始化相机
	    }
	}







>翻译：[@iOnesmile](https://github.com/iOnesmile)    
原始文档：<https://developer.android.com/training/basics/activity-lifecycle/pausing.html>
