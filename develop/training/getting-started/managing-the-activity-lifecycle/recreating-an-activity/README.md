#重新创建 Activity

有些情况下，您的 Activity 会因正常应用行为而销毁，比如当用户按 返回按钮或您的 Activity 通过调用 finish()示意自己的销毁。 如果 Activity 当前被停止或长期未使用，或者前台 Activity 需要更多资源以致系统必须关闭后台进程恢复内存，系统也可能会销毁 Activity。

当您的 Activity 因用户按了返回 或 Activity 自行完成而被销毁时，系统的 Activity 实例概念将永久消失，因为行为指示不再需要 Activity。 但是，如果系统因系统局限性（而非正常应用行为）而销毁 Activity，尽管 Activity 实际实例已不在，系统会记住其存在，这样，如果用户导航回实例，系统会使用描述 Activity 被销毁时状态的一组已保存数据创建 Activity 的新实例。 系统用于恢复先前状态的已保存数据被称为“实例状态”，并且是 Bundle 对象中存储的键值对集合。

**注意：每次用户旋转屏幕时，您的 Activity 将被销毁并重新创建。 当屏幕方向变化时，系统会
销毁并重新创建前台 Activity，因为屏幕配置已更改并且您的 Activity 可能需要加载备用资源
(比如布局)。**

默认情况下，系统会使用 Bundle 实例状态保存您的 Activity 布局（比如，输入到 EditText 对象中的文本值）中有关每个 View 对象的信息。 这样，如果您的 Activity 实例被销毁并重新创建，布局状态便恢复为其先前的状态，且您无需代码。 但是，您的 Activity 可能具有您要恢复的更多状态信息，比如跟踪用户在 Activity 中进度的成员变量。

**注：为了 Android 系统恢复 Activity 中视图的状态，每个视图必须具有 android:id属性提供的唯一 ID。**

要保存有关 Activity 状态的其他数据，您必须替代 onSaveInstanceState() 回调方法。当用户要离开 Activity 并在 Activity 意外销毁时向其传递将保存的 Bundle 对象时，系统会调用此方法。 如果系统必须稍后重新创建 Activity 实例，它会将相同的 Bundle 对象同时传递给 onRestoreInstanceState() 和 onCreate() 方法。

![图片](https://github.com/JackWaiting/android-developer-documentation-chinese/blob/master/develop/training/getting-started/managing-the-activity-lifecycle/recreating-an-activity/basic-lifecycle-savestate.png)

图 2. 当系统开始停止您的 Activity 时，它会调用 onSaveInstanceState()(1)，因此，您可以指定您希望在 Activity 实例必须重新创建时保存的额外状态数据。如果 Activity 被销毁且必须重新创建相同的实例，系统将在 (1) 中定义的状态数据同时传递给 onCreate() 方法 (2) 和 onRestoreInstanceState() 方法 (3)。

##保存 Activity 状态

当您的 Activity 开始停止时，系统会调用 onSaveInstanceState() 以便您的 Activity 可以保存带有键值对集合的状态信息。 此方法的默认实现保存有关 Activity 视图层次的状态信息，例如 EditText 小部件中的文本或ListView 的滚动位置。

要保存 Activity 的更多状态信息，您必须实现 onSaveInstanceState() 并将键值对添加至 Bundle 对象。 例如：

		static final String STATE_SCORE = "playerScore";
		static final String STATE_LEVEL = "playerLevel";
		...
		
		@Override
		public void onSaveInstanceState(Bundle savedInstanceState) {
		    // Save the user's current game state
		    savedInstanceState.putInt(STATE_SCORE, mCurrentScore);
		    savedInstanceState.putInt(STATE_LEVEL, mCurrentLevel);
		
		    // Always call the superclass so it can save the view hierarchy state
		    super.onSaveInstanceState(savedInstanceState);
		}

**注意：始终调用 onSaveInstanceState() 的超类实现，以便默认实现可以保存视图层次结构的状态。**

恢复 Activity 状态
当您的 Activity 在先前销毁之后重新创建时，您可以从系统向 Activity 传递的 Bundle 恢复已保存的状态。onCreate() 和 onRestoreInstanceState() 回调方法均接收包含实例状态信息的相同 Bundle。

因为无论系统正在创建 Activity 的新实例还是重新创建先前的实例，都会调用 onCreate() 方法，因此您必须在尝试读取它之前检查状态 Bundle 是否为 null。 如果为 null，则系统将创建 Activity 的新实例，而不是恢复已销毁的先前实例。

例如，此处显示您如何可以在 onCreate() 中恢复一些状态数据：

		@Override
		protected void onCreate(Bundle savedInstanceState) {
		    super.onCreate(savedInstanceState); // Always call the superclass first
		
		    // Check whether we're recreating a previously destroyed instance
		    if (savedInstanceState != null) {
		        // Restore value of members from saved state
		        mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
		        mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
		    } else {
		        // Probably initialize members with default values for a new instance
		    }
		    ...
		}

您可以选择实现系统在 onStart() 方法之后调用的 onRestoreInstanceState()，而不是在 onCreate() 期间恢复状态。系统只在存在要恢复的已保存状态时调用 onRestoreInstanceState()，因此您无需检查 Bundle 是否为 null：

		public void onRestoreInstanceState(Bundle savedInstanceState) {
		    // Always call the superclass so it can restore the view hierarchy
		    super.onRestoreInstanceState(savedInstanceState);
		
		    // Restore state members from saved instance
		    mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
		    mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
		}

**注意：始终调用 onRestoreInstanceState() 的超类实现，以便默认实现可以恢复视图层次结构的状态。**

要了解更多有关因运行时重启事件（例如屏幕旋转时）而重新创建 Activity 的信息，请阅读[处理运行时变更](https://developer.android.com/guide/topics/resources/runtime-changes.html)。
