# 交叉渐变的两个视图

交叉渐变动画（也称之为溶解）指的是一个UI控件渐渐淡出的同时，另一个控件也淡化。这个动画在你切换应用中的内容或者视图的时候非常有用。它是非常微妙且短暂的提供了一个页面到下个页面的流畅过渡。在你不使用它的时候，过渡提供的感觉生硬且仓促。

`这是一个从进度指示器到一些文本内容交叉渐变的例子 。`
<video controls>  <source src="anim_crossfade.mp4" type="video/mp4">

*交叉渐变动画*

*点击设备屏幕来重新播放电影*

如果你想跳到前面，并查看完整的工作示例，[下载](https://developer.android.google.cn/shareables/training/Animations.zip)并运行示例应用，然后选择交叉渐变的例子，请参阅下面代码实现的文件:

- `src/CrossfadeActivity.java`

- `layout/activity_crossfade.xml`

- `menu/activity_crossfade.xml`
 
## 创建视图

创建两个你想交叉渐变的视图，下面是创建进度指示器和滑动的文本视图的例子。

	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
	        android:id="@+id/content"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent">
	
	        <TextView style="?android:textAppearanceMedium"
	            android:lineSpacingMultiplier="1.2"
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:text="@string/lorem_ipsum"
	            android:padding="16dp" />
	
	    </ScrollView>
	
	    <ProgressBar android:id="@+id/loading_spinner"
	        style="?android:progressBarStyleLarge"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_gravity="center" />
	
	</FrameLayout>
	
## 设置动画

如何设置动画：

1.创建一个你想交叉渐变视图的成员变量，稍后在动画修改视图时，你需要这些引用。

2.将要淡入的视图，设置它的可见度为[GONE](https://developer.android.google.cn/reference/android/view/View.html#GONE),这样可以阻止视图占用布局的空间且布局计算中可忽略它，加快处理。

3.用一个成员变量缓存系统属性 [config_shortAnimTime](https://developer.android.google.cn/reference/android/R.integer.html#config_shortAnimTime),这个属性在动画中定义了短暂持续时间的标准，这种持续时间频繁出现在理想的精巧动画和动画。如果你想使用它们， [config_longAnimTime](https://developer.android.google.cn/reference/android/R.integer.html#config_longAnimTime)和 [config_mediumAnimTime](https://developer.android.google.cn/reference/android/R.integer.html#config_mediumAnimTime) 同样有效。

下面的示例是之前使用那段代码的布局作为Activity文本视图：

	public class CrossfadeActivity extends Activity {

    private View mContentView;
    private View mLoadingView;
    private int mShortAnimationDuration;

    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crossfade);

        mContentView = findViewById(R.id.content);
        mLoadingView = findViewById(R.id.loading_spinner);

        // Initially hide the content view.
        mContentView.setVisibility(View.GONE);

        // Retrieve and cache the system's default "short" animation time.
        mShortAnimationDuration = getResources().getInteger(
                android.R.integer.config_shortAnimTime);
    }

##交叉渐变视图

现在视图正确的设置好了，通过下面的步骤来让他们逐渐消失吧:

1.正要淡入的视图，设置透明值为0，并且可见度为[VISIBILE](https://developer.android.google.cn/reference/android/view/View.html#VISIBLE),（记得初始化它的时候设置为 [GONE](https://developer.android.google.cn/reference/android/view/View.html#GONE))这是让视图看得见但却完全透明。

2.正要淡入的视图，动画的透明值从0到1，同时，正要淡出的视图，动画的透明值从1到0。

3.使用 [Animator.AnimatorListener()](https://developer.android.google.cn/reference/android/animation/Animator.AnimatorListener.html) 中的 [onAnimationEnd()](https://developer.android.google.cn/reference/android/animation/Animator.AnimatorListener.html#onAnimationEnd(android.animation.Animator))，设置淡出的视图可见度为GONE，阻止视图占用布局的空间且布局计算中可忽略它，加快处理。

下面展示了如何执行操作的示例：

	private View mContentView;
	private View mLoadingView;
	private int mShortAnimationDuration;
	
	...
	
	private void crossfade() {
	
	    // Set the content view to 0% opacity but visible, so that it is visible
	    // (but fully transparent) during the animation.
	    mContentView.setAlpha(0f);
	    mContentView.setVisibility(View.VISIBLE);
	
	    // Animate the content view to 100% opacity, and clear any animation
	    // listener set on the view.
	    mContentView.animate()
	            .alpha(1f)
	            .setDuration(mShortAnimationDuration)
	            .setListener(null);
	
	    // Animate the loading view to 0% opacity. After the animation ends,
	    // set its visibility to GONE as an optimization step (it won't
	    // participate in layout passes, etc.)
	    mLoadingView.animate()
	            .alpha(0f)
	            .setDuration(mShortAnimationDuration)
	            .setListener(new AnimatorListenerAdapter() {
	                @Override
	                public void onAnimationEnd(Animator animation) {
	                    mLoadingView.setVisibility(View.GONE);
	                }
	            });
	}

>翻译：[@northJjL](https://github.com/northJjL)  
>     
>原始文档：<https://developer.android.google.cn/training/animation/crossfade.html>