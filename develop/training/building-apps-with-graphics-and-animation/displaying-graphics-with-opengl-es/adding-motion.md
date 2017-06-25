## 添加移动

在屏幕上绘制图像只是 OpenGL 比较基础的特性，除此之外还可以使用 Android 图像框架相关的对象，包括 Canvas、Drawable 等对象。OpenGL ES 提供了三维图形的移动和转换，或其它独特的功能，去创建非凡的用户体验。   

> 这一课，将进一步学习使用 OpenGL ES 如何旋转一个模型。


## 旋转模型

在 OpenGL ES 2.0 中旋转绘图对象是相当简单的。在渲染器中创建一个新的转换矩阵（旋转矩阵），并将它合并到项目中，转换矩阵如下：

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
	
> 如果做了如上操作后三角图像并没有旋转，检查是否有添加 GLSurfaceView.RENDERMODE_WHEN_DIRTY 设置，在下一课将会介绍该设置。


## 设置连续不断的渲染

如果你认真的跟随本课示例代码实现到了这里，确认是否有注释渲染模式（当模型改变时渲染）这一行，否者 OpenGL 仅仅在模型有一个增量时旋转，然后就等待 GLSurfaceView 容器的 requestRender() 方法来触发：

	public MyGLSurfaceView(Context context) {
	    ...
	    // Render the view only when there is a change in the drawing data.
	    // To allow the triangle to rotate automatically, this line is commented out:
	    //setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
	}
	
排除任何用户交互有的改变，将这个标签打开通常是个好主意。准备好不注释这行代码，因为下一课的方法也适用。


> 翻译：[@iOnesmile](https://github.com/iOnesmile)   
> 原始文档：[https://developer.android.google.cn/training/graphics/opengl/motion.html#rotate](https://developer.android.google.cn/training/graphics/opengl/motion.html#rotate)
