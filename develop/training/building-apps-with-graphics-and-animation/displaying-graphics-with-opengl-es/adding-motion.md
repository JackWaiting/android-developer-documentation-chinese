## 添加移动

在屏幕上绘制图像只是 OpenGL 比较基础的特性，除此之外还可以使用 Android 图像框架相关的对象，包括 [Canvas](https://developer.android.google.cn/reference/android/graphics/Canvas.html)、[Drawable](https://developer.android.google.cn/reference/android/graphics/drawable/Drawable.html) 等对象。OpenGL ES 提供了三维空间的图形移动和转换，或其它独特的功能，去创建非凡的用户体验。   

> 这一课，将进一步学习如何使用 OpenGL ES 对一个图形添加旋转动画。


## 旋转模型

在 OpenGL ES 2.0 中旋转绘图对象是相当简单的。在渲染器中创建一个新的转换矩阵（旋转矩阵），并将它合并到项目中，转换矩阵如下：

```java
	private float[] mRotationMatrix = new float[16];
	public void onDrawFrame(GL10 gl) {
	    float[] scratch = new float[16];
	
	    ...
	
	    // Create a rotation transformation for the triangle
	    long time = SystemClock.uptimeMillis() % 4000L;
	    float angle = 0.090f * ((int) time);
	    Matrix.setRotateM(mRotationMatrix, 0, angle, 0, 0, -1.0f);
	
	    // Combine the rotation matrix with the projection and camera view
	    // Note that the mMVPMatrix factor *must be first* in order
	    // for the matrix multiplication product to be correct.
	    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);
	
	    // Draw triangle
	    mTriangle.draw(scratch);
	}
```	
如果做了如上操作后三角图形并没有旋转，请检查是否有添加 [GLSurfaceView.RENDERMODE_WHEN_DIRTY](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html#RENDERMODE_WHEN_DIRTY) 设置，在下一节将会介绍该设置。


## 设置连续渲染

如果你认真的跟随本课示例代码实现到了这里，确认一下是否把设置渲染模式为 *GLSurfaceView.RENDERMODE_WHEN_DIRTY* 这一行注释掉了，否则 OpenGL 对此图形仅仅有一次旋转，然后就等待 [GLSurfaceView](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html) 容器的 [requestRender()](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html#requestRender()) 方法来触发再次执行：

```java
	public MyGLSurfaceView(Context context) {
	    ...
	    // Render the view only when there is a change in the drawing data.
	    // To allow the triangle to rotate automatically, this line is commented out:
	    //setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
	}
```
一般是将这个配置打开的，除非你所要操作的对象变化和用户交互无关。接下来这行代码不会注释，下一课将会再次使用。


> 翻译：[@iOnesmile](https://github.com/iOnesmile)   
> 审核：[@misparking](https://github.com/misparking)
> 原始文档：[https://developer.android.google.cn/training/graphics/opengl/motion.html#rotate](https://developer.android.google.cn/training/graphics/opengl/motion.html#rotate)
