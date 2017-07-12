#缩放视图

这节课展示了如何实现触摸缩放动画，这对于可以支持缩略图到屏幕全尺寸图像的动画效果的应用
程序如相册画廊是比较实用的。
下面是一个触摸缩放动画，看起来像是把图片从缩略图扩展到全屏幕。

<iframe  src="anim_zoom.mp4"></iframe>
>*交叉渐变动画*    
>*点击设备屏幕来重新播放电影*

如果您想跳到前面查看完整的工作示例，请[点击下载](https://commondatastorage.googleapis.com/androiddevelopers/shareables/training/Animations.zip)并运行示例应用程序，然后选择缩放示例。请参见下面的代码实现文件：


* `src/TouchHighlightImageButton.java`(一个简单的帮助类，当按下图像按钮时会显示蓝色的触摸高亮。)

* `src/ZoomActivity.java`
* `layout/activity_zoom.xml`

## 创建视图

为想要缩放的内容创建包含小版本和大版本的布局文件。下面的示例为可点击的图片缩略图创建了 [ImageButton](https://developer.android.google.cn/reference/android/widget/ImageButton.html) 以及可用来展示图片大图的 [ImageView](https://developer.android.google.cn/reference/android/widget/ImageView.html)。

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <ImageButton
            android:id="@+id/thumb_button_1"
            android:layout_width="100dp"
            android:layout_height="75dp"
            android:layout_marginRight="1dp"
            android:src="@drawable/thumb1"
            android:scaleType="centerCrop"
            android:contentDescription="@string/description_image_1" />

    </LinearLayout>

    <!-- This initially-hidden ImageView will hold the expanded/zoomed version of
         the images above. Without transformations applied, it takes up the entire
         screen. To achieve the "zoom" animation, this view's bounds are animated
         from the bounds of the thumbnail button above, to its final laid-out
         bounds.
         -->

    <ImageView
        android:id="@+id/expanded_image"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="invisible"
        android:contentDescription="@string/description_zoom_touch_close" />

</FrameLayout>
```
##设置缩放动画

一旦实现了布局，我们需要设置触发缩放动画的事件 handler。下面的例子为 [ImageButton](https://developer.android.google.cn/reference/android/widget/ImageButton.html)添加了一个 View.OnClickListener，当用户点击按钮时它执行缩放动画。

```java

public class ZoomActivity extends FragmentActivity {
    // Hold a reference to the current animator,
    // so that it can be canceled mid-way.
    private Animator mCurrentAnimator;

    // The system "short" animation time duration, in milliseconds. This
    // duration is ideal for subtle animations or animations that occur
    // very frequently.
    private int mShortAnimationDuration;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_zoom);

        // Hook up clicks on the thumbnail views.

        final View thumb1View = findViewById(R.id.thumb_button_1);
        thumb1View.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                zoomImageFromThumb(thumb1View, R.drawable.image1);
            }
        });

        // Retrieve and cache the system's default "short" animation time.
        mShortAnimationDuration = getResources().getInteger(
                android.R.integer.config_shortAnimTime);
    }
    ...
}

```

##缩放视图


你需要在合适的时候进行缩放动画。一般来说，使用动画时需要按照边界来把正常尺寸的视图扩展到大尺寸的视图，下面的方法展示了如何实现缩放动画：

1.把高清图像资源设置到已经被隐藏的“放大版”的 [ImageView](https://developer.android.google.cn/reference/android/widget/ImageView.html) 中。下面的示例在 UI 线程上加载一个大的图像资源以简化操作。但是我们需要在一个单独的线程中来加载以免阻塞UI线程，然后再回到UI线程中设置。理想状况下，图片不要大过屏幕尺寸。

2.计算 [ImageView](https://developer.android.google.cn/reference/android/widget/ImageView.html) 开始和结束时的边界。

3.从起始边到结束边同步地动态改变四个点的位置和大小属性 [X](https://developer.android.google.cn/reference/android/view/View.html#X)，[Y](https://developer.android.google.cn/reference/android/view/View.html#Y)（[SCALE_X](https://developer.android.google.cn/reference/android/view/View.html#SCALE_X) 和 [SCALE_Y](https://developer.android.google.cn/reference/android/view/View.html#SCALE_Y)）。这四个动画被加入到了 [AnimatorSet](https://developer.android.google.cn/reference/android/animation/AnimatorSet.html)，所以它们可以同一时间开始。

4.通过运行类似的动画可以用来缩小图像，即当用户在点击屏幕放大图像时是相反的。我们可以在ImageView中添加一个 [View.OnClickListener](https://developer.android.google.cn/reference/android/view/View.OnClickListener.html) 来实现它。当点击时，ImageView 缩回到原来缩略图的大小，然后设置它的 visibility 为 [GONE](https://developer.android.google.cn/reference/android/view/View.html#GONE) 来隐藏。

```java

private void zoomImageFromThumb(final View thumbView, int imageResId) {
    // If there's an animation in progress, cancel it
    // immediately and proceed with this one.
    if (mCurrentAnimator != null) {
        mCurrentAnimator.cancel();
    }

    // Load the high-resolution "zoomed-in" image.
    final ImageView expandedImageView = (ImageView) findViewById(
            R.id.expanded_image);
    expandedImageView.setImageResource(imageResId);

    // Calculate the starting and ending bounds for the zoomed-in image.
    // This step involves lots of math. Yay, math.
    final Rect startBounds = new Rect();
    final Rect finalBounds = new Rect();
    final Point globalOffset = new Point();

    // The start bounds are the global visible rectangle of the thumbnail,
    // and the final bounds are the global visible rectangle of the container
    // view. Also set the container view's offset as the origin for the
    // bounds, since that's the origin for the positioning animation
    // properties (X, Y).
    thumbView.getGlobalVisibleRect(startBounds);
    findViewById(R.id.container)
            .getGlobalVisibleRect(finalBounds, globalOffset);
    startBounds.offset(-globalOffset.x, -globalOffset.y);
    finalBounds.offset(-globalOffset.x, -globalOffset.y);

    // Adjust the start bounds to be the same aspect ratio as the final
    // bounds using the "center crop" technique. This prevents undesirable
    // stretching during the animation. Also calculate the start scaling
    // factor (the end scaling factor is always 1.0).
    float startScale;
    if ((float) finalBounds.width() / finalBounds.height()
            > (float) startBounds.width() / startBounds.height()) {
        // Extend start bounds horizontally
        startScale = (float) startBounds.height() / finalBounds.height();
        float startWidth = startScale * finalBounds.width();
        float deltaWidth = (startWidth - startBounds.width()) / 2;
        startBounds.left -= deltaWidth;
        startBounds.right += deltaWidth;
    } else {
        // Extend start bounds vertically
        startScale = (float) startBounds.width() / finalBounds.width();
        float startHeight = startScale * finalBounds.height();
        float deltaHeight = (startHeight - startBounds.height()) / 2;
        startBounds.top -= deltaHeight;
        startBounds.bottom += deltaHeight;
    }

    // Hide the thumbnail and show the zoomed-in view. When the animation
    // begins, it will position the zoomed-in view in the place of the
    // thumbnail.
    thumbView.setAlpha(0f);
    expandedImageView.setVisibility(View.VISIBLE);

    // Set the pivot point for SCALE_X and SCALE_Y transformations
    // to the top-left corner of the zoomed-in view (the default
    // is the center of the view).
    expandedImageView.setPivotX(0f);
    expandedImageView.setPivotY(0f);

    // Construct and run the parallel animation of the four translation and
    // scale properties (X, Y, SCALE_X, and SCALE_Y).
    AnimatorSet set = new AnimatorSet();
    set
            .play(ObjectAnimator.ofFloat(expandedImageView, View.X,
                    startBounds.left, finalBounds.left))
            .with(ObjectAnimator.ofFloat(expandedImageView, View.Y,
                    startBounds.top, finalBounds.top))
            .with(ObjectAnimator.ofFloat(expandedImageView, View.SCALE_X,
            startScale, 1f)).with(ObjectAnimator.ofFloat(expandedImageView,
                    View.SCALE_Y, startScale, 1f));
    set.setDuration(mShortAnimationDuration);
    set.setInterpolator(new DecelerateInterpolator());
    set.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            mCurrentAnimator = null;
        }

        @Override
        public void onAnimationCancel(Animator animation) {
            mCurrentAnimator = null;
        }
    });
    set.start();
    mCurrentAnimator = set;

    // Upon clicking the zoomed-in image, it should zoom back down
    // to the original bounds and show the thumbnail instead of
    // the expanded image.
    final float startScaleFinal = startScale;
    expandedImageView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            if (mCurrentAnimator != null) {
                mCurrentAnimator.cancel();
            }

            // Animate the four positioning/sizing properties in parallel,
            // back to their original values.
            AnimatorSet set = new AnimatorSet();
            set.play(ObjectAnimator
                        .ofFloat(expandedImageView, View.X, startBounds.left))
                        .with(ObjectAnimator
                                .ofFloat(expandedImageView,
                                        View.Y,startBounds.top))
                        .with(ObjectAnimator
                                .ofFloat(expandedImageView,
                                        View.SCALE_X, startScaleFinal))
                        .with(ObjectAnimator
                                .ofFloat(expandedImageView,
                                        View.SCALE_Y, startScaleFinal));
            set.setDuration(mShortAnimationDuration);
            set.setInterpolator(new DecelerateInterpolator());
            set.addListener(new AnimatorListenerAdapter() {
                @Override
                public void onAnimationEnd(Animator animation) {
                    thumbView.setAlpha(1f);
                    expandedImageView.setVisibility(View.GONE);
                    mCurrentAnimator = null;
                }

                @Override
                public void onAnimationCancel(Animator animation) {
                    thumbView.setAlpha(1f);
                    expandedImageView.setVisibility(View.GONE);
                    mCurrentAnimator = null;
                }
            });
            set.start();
            mCurrentAnimator = set;
        }
    });
}
```
>翻译：[@misparking](https://github.com/misparking)   and
原始文档：<https://developer.android.google.cn/training/animation/zoom.html#animate>