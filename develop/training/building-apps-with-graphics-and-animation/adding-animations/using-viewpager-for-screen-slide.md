# 如何使用 ViewPager 进行屏幕滑动

屏幕滑动时一个屏幕与另一个屏幕之间的过滤，这种UI的使用在我们的APP中也是非常常见的。本课程将向你展示如何使用支持库提供的 ViewPager 进行屏幕滑动。 同时ViewPagers也可以自动添加动画。 下面这个就是从一个屏幕滑动到另一个屏幕：

![](https://developer.android.com/training/animation/anim_screenslide.webm)

如果你想跳过并查看完整的工作示例，请下载并运行示例应用程序并选择滑动示例。 有关代码实现，请参阅以下文件：



- src/ScreenSlidePageFragment.java
- src/ScreenSlideActivity.java
- layout/activity_screen_slide.xml
- layout/fragment_screen_slide_page.xml

## 创建Views

创建一个布局文件，稍后将用于 Fragment 的内容。 以下示例包含显示一些文本视图：

	<!-- fragment_screen_slide_page.xml -->
	<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/content"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent" >
	
	    <TextView style="?android:textAppearanceMedium"
	        android:padding="16dp"
	        android:lineSpacingMultiplier="1.2"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:text="@string/lorem_ipsum" />
	</ScrollView>

还为该 Fragment 的内容定义一个字符串。

## 创建Fragment

创建一个 Fragment 类，返回刚刚在 onCreateView（） 方法中创建的布局。 然后，你可以在需要向用户显示的新页面时在父 Activity 中创建此 Fragment 的实例：

	import android.support.v4.app.Fragment;
	...
	public class ScreenSlidePageFragment extends Fragment {
	
	    @Override
	    public View onCreateView(LayoutInflater inflater, ViewGroup container,
	            Bundle savedInstanceState) {
	        ViewGroup rootView = (ViewGroup) inflater.inflate(
	                R.layout.fragment_screen_slide_page, container, false);
	
	        return rootView;
	    }
	}

## 添加一个ViewPager

ViewPagers 具有内置的滑动手势，可以通过界面进行转换，并且默认情况下显示滑动动画，因此你不需要创建任何内容。 ViewPager使用PagerAdapters作为要显示的适配器，因此 PagerAdapter 将使用你之前创建的 Fragment 类。

首先，创建一个包含ViewPager的布局：

	<!-- activity_screen_slide.xml -->
	<android.support.v4.view.ViewPager
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/pager"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent" />

创建一个执行以下操作的 Activity：

- 使用 ViewPager 将内容视图设置为布局。
- 创建一个扩展 FragmentStatePagerAdapter 的抽象类，并实现 getItem（）方法，将 ScreenSlidePageFragment 的实例作为 content。 同时适配器还要求你实现getCount（）方法，该方法返回适配器将创建的页面数量（示例中为五个）。
- 将 PagerAdapter 设置到 ViewPager。
- 通过在虚拟堆栈中向后移动来处理设备的后退按钮。 如果用户已经在第一页上，请返回到 Activity 堆栈。

		import android.support.v4.app.Fragment;
		import android.support.v4.app.FragmentManager;
		...
		public class ScreenSlidePagerActivity extends FragmentActivity {
		    /**
		     * The number of pages (wizard steps) to show in this demo.
		     */
		    private static final int NUM_PAGES = 5;
		
		    /**
		     * The pager widget, which handles animation and allows swiping horizontally to access previous
		     * and next wizard steps.
		     */
		    private ViewPager mPager;
		
		    /**
		     * The pager adapter, which provides the pages to the view pager widget.
		     */
		    private PagerAdapter mPagerAdapter;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_screen_slide);
		
		        // Instantiate a ViewPager and a PagerAdapter.
		        mPager = (ViewPager) findViewById(R.id.pager);
		        mPagerAdapter = new ScreenSlidePagerAdapter(getSupportFragmentManager());
		        mPager.setAdapter(mPagerAdapter);
		    }
		
		    @Override
		    public void onBackPressed() {
		        if (mPager.getCurrentItem() == 0) {
		            // If the user is currently looking at the first step, allow the system to handle the
		            // Back button. This calls finish() on this activity and pops the back stack.
		            super.onBackPressed();
		        } else {
		            // Otherwise, select the previous step.
		            mPager.setCurrentItem(mPager.getCurrentItem() - 1);
		        }
		    }
		
		    /**
		     * A simple pager adapter that represents 5 ScreenSlidePageFragment objects, in
		     * sequence.
		     */
		    private class ScreenSlidePagerAdapter extends FragmentStatePagerAdapter {
		        public ScreenSlidePagerAdapter(FragmentManager fm) {
		            super(fm);
		        }
		
		        @Override
		        public Fragment getItem(int position) {
		            return new ScreenSlidePageFragment();
		        }
		
		        @Override
		        public int getCount() {
		            return NUM_PAGES;
		        }
		    }
		}

## 使用PageTransformer自定义动画

要从默认滑动动画转换成显示不同的动画，请调用 ViewPager.PageTransformer 并将其提供给适配器。该界面暴露了一种方法 transformPage（）。每个可见页面（通常只有一个可见页面）和显示在屏幕上的两个界面调用此方法。例如，如果第三页是可见的，并且用户拖动到第四页，则在手势的每个步骤都会调用 transformPage（） 来调用页面2，3和4。

在实现 transformPage（） 时，你可以通过从页面的位置参数 transformPage（） 方法获取的位置确定哪些页面需要进行变换来创建自定义的滑动动画。

位置参数指示给定页面相对于屏幕中心的位置。它是一种动态属性，随着用户滚动页面而改变。当页面填满屏幕时，其位置值为0.当页面刚刚从屏幕右侧绘制时，其位置值为1.如果用户在第一页和第二页之间滚动一半，则第一页的位置为-0.5和第二页的位置为0.5。根据屏幕上页面的位置，你可以通过使用setAlpha（），setTranslationX（） 或 setScaleY（） 等方法设置页面属性来创建自定义滑动动画。

当你实现一个 PageTransformer 时，请在实现中调用 setPageTransformer（） 来应用自定义动画。例如，如果你有一个名为 ZoomOutPageTransformer 的 PageTransformer，你可以设置自定义动画，如下所示：

	ViewPager mPager = (ViewPager) findViewById(R.id.pager);
	...
	mPager.setPageTransformer(true, new ZoomOutPageTransformer());

有关PageTransformer的示例和视频，请参阅 Zoom-out page transformer 和 Depth page transformer 部分。

## Zoom-out page transformer

当在相邻页面之间滚动时，此页面 transformer 收缩并淡化页面。 随着页面越来越接近中心，它逐渐恢复到正常的大小并且渐渐衰

	public class ZoomOutPageTransformer implements ViewPager.PageTransformer {
	    private static final float MIN_SCALE = 0.85f;
	    private static final float MIN_ALPHA = 0.5f;
	
	    public void transformPage(View view, float position) {
	        int pageWidth = view.getWidth();
	        int pageHeight = view.getHeight();
	
	        if (position < -1) { // [-Infinity,-1)
	            // This page is way off-screen to the left.
	            view.setAlpha(0);
	
	        } else if (position <= 1) { // [-1,1]
	            // Modify the default slide transition to shrink the page as well
	            float scaleFactor = Math.max(MIN_SCALE, 1 - Math.abs(position));
	            float vertMargin = pageHeight * (1 - scaleFactor) / 2;
	            float horzMargin = pageWidth * (1 - scaleFactor) / 2;
	            if (position < 0) {
	                view.setTranslationX(horzMargin - vertMargin / 2);
	            } else {
	                view.setTranslationX(-horzMargin + vertMargin / 2);
	            }
	
	            // Scale the page down (between MIN_SCALE and 1)
	            view.setScaleX(scaleFactor);
	            view.setScaleY(scaleFactor);
	
	            // Fade the page relative to its size.
	            view.setAlpha(MIN_ALPHA +
	                    (scaleFactor - MIN_SCALE) /
	                    (1 - MIN_SCALE) * (1 - MIN_ALPHA));
	
	        } else { // (1,+Infinity]
	            // This page is way off-screen to the right.
	            view.setAlpha(0);
	        }
	    }
	}

## Depth page transformer

该页面 transformer 使用默认的滑动动画来滑动左侧的页面，同时使用“深度”动画将页面滑动到右侧。 这个深度动画使页面淡出，并将其线性缩放。

> 注意：在深度动画过程中，默认动画（滑动动画）仍然发生，因此你必须使用 -X 旋转来抵消滑动动画。 例如：

	view.setTranslationX（-1 * view.getWidth（）* position）;

以下示例显示了如何抵消默认滑动动画：

	public class DepthPageTransformer implements ViewPager.PageTransformer {
	    private static final float MIN_SCALE = 0.75f;
	
	    public void transformPage(View view, float position) {
	        int pageWidth = view.getWidth();
	
	        if (position < -1) { // [-Infinity,-1)
	            // This page is way off-screen to the left.
	            view.setAlpha(0);
	
	        } else if (position <= 0) { // [-1,0]
	            // Use the default slide transition when moving to the left page
	            view.setAlpha(1);
	            view.setTranslationX(0);
	            view.setScaleX(1);
	            view.setScaleY(1);
	
	        } else if (position <= 1) { // (0,1]
	            // Fade the page out.
	            view.setAlpha(1 - position);
	
	            // Counteract the default slide transition
	            view.setTranslationX(pageWidth * -position);
	
	            // Scale the page down (between MIN_SCALE and 1)
	            float scaleFactor = MIN_SCALE
	                    + (1 - MIN_SCALE) * (1 - Math.abs(position));
	            view.setScaleX(scaleFactor);
	            view.setScaleY(scaleFactor);
	
	        } else { // (1,+Infinity]
	            // This page is way off-screen to the right.
	            view.setAlpha(0);
	        }
	    }
	}

