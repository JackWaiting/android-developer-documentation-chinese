## 卡片翻转动画

这一课讲述如何自定义卡片翻转动画。卡片翻转在视图之间，展示模拟的翻页动画。

> 这里是一个卡片翻转

<video id="video" controls="" preload="none" poster="">
      <source id="mp4" src="https://developer.android.google.cn/training/animation/anim_card_flip.mp4">
      <p>卡片翻转动画</p>
</video>

> 点击设备屏幕重播视频

如果你想抢先看到所有动画示例，[下载](https://developer.android.google.cn/shareables/training/Animations.zip) 并运行示例应用，选择 Card Flip 示例。见如下文件的代码实现：

- src/CardFlipActivity.java

- animator/card\_flip\_right\_in.xml

- animator/card\_flip\_right\_out.xml

- animator/card\_flip\_left\_in.xml

- animator/card\_flip\_left\_out.xml

- layout/fragment\_card\_back.xml

- layout/fragment\_card\_front.xml


## 创建动画对象

创建卡片翻转动画。在卡片的正面需要两个动画，动画向左出和从左进。同样在卡片的反面也需要两个动画，动画从右进和向右出。

#### card\_flip\_left\_in.xml

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Before rotating, immediately set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:duration="0" />

    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="-180"
        android:valueTo="0"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 1. -->
    <objectAnimator
        android:valueFrom="0.0"
        android:valueTo="1.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

#### card\_flip\_left\_out.xml

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="0"
        android:valueTo="180"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

#### card\_flip\_right\_in.xml

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Before rotating, immediately set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:duration="0" />

    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="180"
        android:valueTo="0"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 1. -->
    <objectAnimator
        android:valueFrom="0.0"
        android:valueTo="1.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

#### card\_flip\_right\_out.xml

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="0"
        android:valueTo="-180"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

## 创建视图

卡片的每一面是单独的布局，包含着你想显示的任何内容。比如两屏文字，两张图片，或者任何视图的组合都可进行翻转。在即将显示的动画中，Fragment 需要使用两个布局。下面的布局是卡片的一面显示的文本：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#a6c"
    android:padding="16dp"
    android:gravity="bottom">

    <TextView android:id="@android:id/text1"
        style="?android:textAppearanceLarge"
        android:textStyle="bold"
        android:textColor="#fff"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/card_back_title" />

    <TextView style="?android:textAppearanceSmall"
        android:textAllCaps="true"
        android:textColor="#80ffffff"
        android:textStyle="bold"
        android:lineSpacingMultiplier="1.2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/card_back_description" />

</LinearLayout>
```

卡片的另外一面显示的 [ImageView](https://developer.android.google.cn/reference/android/widget/ImageView.html)：

```xml
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:src="@drawable/image1"
    android:scaleType="centerCrop"
    android:contentDescription="@string/description_image_1" />
```

## 创建 Fragment

创建卡片正面和反面的 Fragment 类。在每个 Fragment 类的 [onCreateView()](https://developer.android.google.cn/reference/android/app/Fragment.html#onCreateView(android.view.LayoutInflater,%20android.view.ViewGroup,%20android.os.Bundle)) 方法中返回预先创建的布局。你可以在父类的 Activity 中创建要显示的卡片的 Fragment 实例。如下示例在父 Activity 内嵌的 Fragment 类：

```java
public class CardFlipActivity extends Activity {
    ...
    /**
     * A fragment representing the front of the card.
     */
    public class CardFrontFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_card_front, container, false);
        }
    }

    /**
     * A fragment representing the back of the card.
     */
    public class CardBackFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_card_back, container, false);
        }
    }
}
```

## 卡片翻转动画

现在，你将需要在父 Activity 中显示 Fragment。为此，第一步先创建 Activity 的布局。下面的示例创建了 FrameLayout，可以在运行时添加 Fragment：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

在 Activity 代码中，设置刚才创建的布局。在 Activity 创建的时候设置一个默认 Fragment，通常是个不错的方式，因此下面示例的 Activity 展示了如何默认显示卡片正面：
```java
public class CardFlipActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_activity_card_flip);

        if (savedInstanceState == null) {
            getFragmentManager()
                    .beginTransaction()
                    .add(R.id.container, new CardFrontFragment())
                    .commit();
        }
    }
    ...
}
```

到目前为止卡片已经显示，在适当的时候通过动画翻转到卡片反面。创建方法显示卡片的另外一面，步骤如下：

- 给 Fragment 事务设置之前创建好的自定义动画。

- 用新的 Fragment 替换正在显示的 Fragment，伴随着已经创建的自定义动画。

- 添加准备显示的 Fragment 到返回栈，那么当用户按返回键时，卡片会翻回去。

```java
private void flipCard() {
    if (mShowingBack) {
        getFragmentManager().popBackStack();
        return;
    }

    // Flip to the back.

    mShowingBack = true;

    // Create and commit a new fragment transaction that adds the fragment for
    // the back of the card, uses custom animations, and is part of the fragment
    // manager's back stack.

    getFragmentManager()
            .beginTransaction()

            // Replace the default fragment animations with animator resources
            // representing rotations when switching to the back of the card, as
            // well as animator resources representing rotations when flipping
            // back to the front (e.g. when the system Back button is pressed).
            .setCustomAnimations(
                    R.animator.card_flip_right_in,
                    R.animator.card_flip_right_out,
                    R.animator.card_flip_left_in,
                    R.animator.card_flip_left_out)

            // Replace any fragments currently in the container view with a
            // fragment representing the next page (indicated by the
            // just-incremented currentPage variable).
            .replace(R.id.container, new CardBackFragment())

            // Add this transaction to the back stack, allowing users to press
            // Back to get to the front of the card.
            .addToBackStack(null)

            // Commit the transaction.
            .commit();
}
```


> 翻译：[@iOnesmile](https://github.com/ionesmile)    
> 校对：[@misparking](https://github.com/misparking)      
> 原始文档：<https://developer.android.google.cn/training/animation/cardflip.html>
