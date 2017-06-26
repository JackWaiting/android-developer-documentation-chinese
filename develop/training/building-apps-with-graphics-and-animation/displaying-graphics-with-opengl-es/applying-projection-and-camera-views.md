# 应用投影和相机视图

在OpenGL ES环境中，投影和摄像机视图允许你的眼睛看到物体对象更接近的方式显示绘制的对象。物理观察的这种模拟是通过绘制对象坐标的数学变换完成的：

- 投影 - 此变换根据显示 [GLSurfaceView](https://developer.android.com/reference/android/opengl/GLSurfaceView.html) 的宽度和高度来调整绘制对象的坐标。没有这个计算， OpenGL ES 绘制的对象被视图窗口的不等比例偏移。在渲染器的 [onSurfaceChanged()](https://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceChanged) 方法中建立或更改 OpenGL 视图的比例时，通常只需计算投影变换。有关 OpenGL ES 投影和坐标映射的更多信息，请[参阅绘制对象的坐标坐标](https://developer.android.com/guide/topics/graphics/opengl.html#coordinate-mapping)。 

 
- 相机视图 - 此转换根据虚拟相机位置调整绘制对象的坐标。请注意，OpenGL ES 不会定义实际的摄像机对象，而是提供了通过转换绘制对象的显示来模拟摄像机的实用方法。建立 [GLSurfaceView](https://developer.android.com/reference/android/opengl/GLSurfaceView.html) 时，相机视图转换可能只计算一次，或者可能会根据用户操作或应用程序的功能动态更改。


本课介绍如何创建投影和相机视图，并将其应用于 [GLSurfaceView](https://developer.android.com/reference/android/opengl/GLSurfaceView.html) 中绘制的形状。

## 定义投影

投影变换的数据在 [GLSurfaceView.Renderer](https://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html) 类的 [onSurfaceChanged()](https://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceChanged) 方法中计算。 以下示例代码使用 [GLSurfaceView](https://developer.android.com/reference/android/opengl/GLSurfaceView.html) 的高度和宽度，并使用它使用 [Matrix.frustumM()](https://developer.android.com/reference/android/opengl/Matrix.html#frustumM)) 方法填充投影变换矩阵：

	// mMVPMatrix is an abbreviation for "Model View Projection Matrix"
	private final float[] mMVPMatrix = new float[16];
	private final float[] mProjectionMatrix = new float[16];
	private final float[] mViewMatrix = new float[16];
	
	@Override
	public void onSurfaceChanged(GL10 unused, int width, int height) {
	    GLES20.glViewport(0, 0, width, height);
	
	    float ratio = (float) width / height;
	
	    // this projection matrix is applied to object coordinates
	    // in the onDrawFrame() method
	    Matrix.frustumM(mProjectionMatrix, 0, -ratio, ratio, -1, 1, 3, 7);
	}	

该代码填充一个投影矩阵 mProjectionMatrix，然后您可以在 [onDrawFrame()](https://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onDrawFrame) 方法中与相机视图转换相结合，下一节将显示。




> 注意：将投影变换应用于绘图对象通常会导致非常空的显示。 一般来说，您还必须应用相机视图转换才能在屏幕上显示任何内容。

## 定义相机视图

通过在渲染器中添加一个相机视图转换作为绘图过程的一部分，完成转换绘制对象的过程。 在以下示例代码中，使用 [Matrix.setLookAtM()](https://developer.android.com/reference/android/opengl/Matrix.html#setLookAtM) 方法计算相机视图转换，然后与先前计算的投影矩阵组合。 然后将组合的变换矩阵传递到绘制的形状。

	@Override
	public void onDrawFrame(GL10 unused) {
	    ...
	    // Set the camera position (View matrix)
	    Matrix.setLookAtM(mViewMatrix, 0, 0, 0, -3, 0f, 0f, 0f, 0f, 1.0f, 0.0f);
	
	    // Calculate the projection and view transformation
	    Matrix.multiplyMM(mMVPMatrix, 0, mProjectionMatrix, 0, mViewMatrix, 0);
	
	    // Draw shape
	    mTriangle.draw(mMVPMatrix);
	}

## 应用投影和相机转换

为了使用预览部分中显示的组合投影和相机视图转换矩阵，首先将矩阵变量添加到先前在三角形类中定义的顶点着色器：

	public class Triangle {
	
	    private final String vertexShaderCode =
	        // This matrix member variable provides a hook to manipulate
	        // the coordinates of the objects that use this vertex shader
	        "uniform mat4 uMVPMatrix;" +
	        "attribute vec4 vPosition;" +
	        "void main() {" +
	        // the matrix must be included as a modifier of gl_Position
	        // Note that the uMVPMatrix factor *must be first* in order
	        // for the matrix multiplication product to be correct.
	        "  gl_Position = uMVPMatrix * vPosition;" +
	        "}";
	
	    // Use to access and set the view transformation
	    private int mMVPMatrixHandle;
	
	    ...
	}

接下来，修改图形对象的draw（）方法以接受组合的转换矩阵并将其应用于形状：

	public void draw(float[] mvpMatrix) { // pass in the calculated transformation matrix
	    ...
	
	    // get handle to shape's transformation matrix
	    mMVPMatrixHandle = GLES20.glGetUniformLocation(mProgram, "uMVPMatrix");
	
	    // Pass the projection and view transformation to the shader
	    GLES20.glUniformMatrix4fv(mMVPMatrixHandle, 1, false, mvpMatrix, 0);
	
	    // Draw the triangle
	    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);
	
	    // Disable vertex array
	    GLES20.glDisableVertexAttribArray(mPositionHandle);
	}


一旦你正确地计算并应用了投影和相机视图转换，你的图形对象就按照正确的比例绘制，应该是这样的：

![](ogl-triangle-projected.png)

**图1.**应用投影和相机视图绘制的三角形。

现在，您有一个应用程序以正确的比例显示您的形状，现在是添加运动到您的形状的时间。				
>翻译：[JackWaiting](https://github.com/JackWaiting)    
> 审核：[@northJjL](https://github.com/northJjL)        
原始文档：[https://developer.android.com/training/graphics/opengl/projection.html#projection](https://developer.android.com/training/graphics/opengl/projection.html#projection)							