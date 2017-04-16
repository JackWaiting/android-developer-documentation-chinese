# 控制相机 

在本课中，我们将讨论如何使用框架 API 直接控制摄像机硬件。

直接控制设备相机需要比从现有相机应用程序中请求图片或视频更多的代码。 但是，如果您要构建专门的相机应用程序或者完全集成到应用程序UI中的东西，本课程将向您展示。

## 打开相机对象

获取 [Camera](https://developer.android.com/reference/android/hardware/Camera.html) 对象的实例是直接控制相机过程的第一步。 作为 Android 自己的相机应用程序，推荐的访问相机的方法是在从 [onCreate()](https://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 启动的单独线程上打开相机。 这种方法是一个好主意，因为它可能需要一段时间，并可能会杀死UI线程。 在更基本的实现中，打开相机可以推迟到 [onResume()](https://developer.android.com/reference/android/app/Activity.html#onResume()) 方法，以方便代码重用，并保持控制流程简单。

如果相机已被另一个应用程序使用，调用 [Camera.open()](https://developer.android.com/reference/android/hardware/Camera.html#open()) 会抛出异常，因此我们将其包装在一个 try 块中。

	private boolean safeCameraOpen(int id) {
    	boolean qOpened = false;

    	try {
        	releaseCameraAndPreview();
        	mCamera = Camera.open(id);
        	qOpened = (mCamera != null);
    	} catch (Exception e) {
        	Log.e(getString(R.string.app_name), "failed to open Camera");
        	e.printStackTrace();
    	}

    	return qOpened;
	}

	private void releaseCameraAndPreview() {
    	mPreview.setCamera(null);
    	if (mCamera != null) {
        	mCamera.release();
        	mCamera = null;
   		}
	}

自 API 版本 9 以来，相机框架支持多台相机。 如果您使用旧版 API 并调用 [open()](https://developer.android.com/reference/android/hardware/Camera.html#open()) 而不带参数，则会获得第一个后置摄像头。

## 创建相机预览

拍摄照片通常需要您的用户在点击快门之前看到他们的主题的预览。 为此，你可以使用 [SurfaceView](https://developer.android.com/reference/android/view/SurfaceView.html) 绘制相机传感器拾取的预览。

## 预览课程

要开始显示预览，您需要预览类。 该预览需要实现 android.view.SurfaceHolder.Callback 接口，用于将图像数据从摄像机硬件传递到应用程序。        

	class Preview extends ViewGroup implements SurfaceHolder.Callback {
	
	    SurfaceView mSurfaceView;
	    SurfaceHolder mHolder;
	
	    Preview(Context context) {
	        super(context);
	
	        mSurfaceView = new SurfaceView(context);
	        addView(mSurfaceView);
	
	        // Install a SurfaceHolder.Callback so we get notified when the
	        // underlying surface is created and destroyed.
	        mHolder = mSurfaceView.getHolder();
	        mHolder.addCallback(this);
	        mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
	    }
	...
	} 

在开始实时图像预览之前，必须将预览类传递给 [Camera](https://developer.android.com/reference/android/hardware/Camera.html) 对象，如下一节所示。

## 设置并启动预览

相机实例及其相关预览必须以特定顺序创建，相机对象为第一个。 在下面的代码片段中，初始化相机的过程将被封装，以便每当用户执行某些操作才能更改相机时，将通过 setCamera() 方法调用 [Camera.startPreview()](https://developer.android.com/reference/android/hardware/Camera.html#startPreview()) 。 预览也必须在预览类surfaceChanged() 回调方法中重新启动。
	
	public void setCamera(Camera camera) {
	    if (mCamera == camera) { return; }
	
	    stopPreviewAndFreeCamera();
	
	    mCamera = camera;
	
	    if (mCamera != null) {
	        List<Size> localSizes = mCamera.getParameters().getSupportedPreviewSizes();
	        mSupportedPreviewSizes = localSizes;
	        requestLayout();
	
	        try {
	            mCamera.setPreviewDisplay(mHolder);
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	
	        // Important: Call startPreview() to start updating the preview
	        // surface. Preview must be started before you can take a picture.
	        mCamera.startPreview();
	    }
	} 

## 修改相机设置

相机设置会更改相机拍摄的方式，从缩放级别到曝光补偿。 此示例仅更改预览大小; 请参阅相机应用程序的源代码更多。

	public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
	    // Now that the size is known, set up the camera parameters and begin
	    // the preview.
	    Camera.Parameters parameters = mCamera.getParameters();
	    parameters.setPreviewSize(mPreviewSize.width, mPreviewSize.height);
	    requestLayout();
	    mCamera.setParameters(parameters);
	
	    // Important: Call startPreview() to start updating the preview surface.
	    // Preview must be started before you can take a picture.
	    mCamera.startPreview();
	}	


## 设置预览方向

大多数相机应用程序将显示器锁定到横向模式，因为这是相机传感器的自然方向。此设置不会阻止您拍摄肖像模式照片，因为设备的方向记录在EXIF标题中。 [setCameraDisplayOrientation()](https://developer.android.com/reference/android/hardware/Camera.html#setDisplayOrientation(int)) 方法可以改变预览的显示方式，而不影响图像的记录方式。但是，在 API 级别为 14 之前的 Android 中，您必须先停止预览，然后再更改方向，然后重新启动。

## 拍照
预览开始后，使用 [Camera.takePicture()](https://developer.android.com/reference/android/hardware/Camera.PictureCallback.html) 方法拍摄照片。您可以创建 [Camera.PictureCallback](https://developer.android.com/reference/android/hardware/Camera.PictureCallback.html) 和 [Camera.ShutterCallback](https://developer.android.com/reference/android/hardware/Camera.ShutterCallback.html) 对象，并将它们传递到 [Camera.takePicture()](https://developer.android.com/reference/android/hardware/Camera.PictureCallback.html) 中。

如果要持续抓取图像，可以创建一个实现 [onPreviewFrame()](https://developer.android.com/reference/android/hardware/Camera.PreviewCallback.html#onPreviewFrame) 的 [Camera.PictureCallback](https://developer.android.com/reference/android/hardware/Camera.PictureCallback.html)。对于两者之间的内容，您只能捕获所选的预览帧，或设置一个延迟动作来调用 [takePicture()](https://developer.android.com/reference/android/hardware/Camera.html#takePicture)。

## 重新启动预览

拍摄照片后，您必须重新启动预览，然后用户才能拍摄照片。 在此示例中，重新启动是通过重新启动快门按钮完成的。

	@Override
	public void onClick(View v) {
	    switch(mPreviewState) {
	    case K_STATE_FROZEN:
	        mCamera.startPreview();
	        mPreviewState = K_STATE_PREVIEW;
	        break;
	
	    default:
	        mCamera.takePicture( null, rawCallback, null);
	        mPreviewState = K_STATE_BUSY;
	    } // switch
	    shutterBtnConfig();
	}


## 停止预览并释放相机

一旦您的应用程序完成使用相机，它是时候清理。 特别是，您必须释放 [Camera](https://developer.android.com/reference/android/hardware/Camera.html) 对象，否则可能会导致其他应用程序崩溃，包括您自己应用程序的新实例。

什么时候应该停止预览并释放相机？ 那么你的预览表面被破坏是一个非常好的提示，现在是停止预览和释放相机的时间，如预览类中的这些方法所示。


	public void surfaceDestroyed(SurfaceHolder holder) {
	    // Surface will be destroyed when we return, so stop the preview.
	    if (mCamera != null) {
	        // Call stopPreview() to stop updating the preview surface.
	        mCamera.stopPreview();
	    }
	}
	
	/**
	 * When this function returns, mCamera will be null.
	 */
	private void stopPreviewAndFreeCamera() {
	
	    if (mCamera != null) {
	        // Call stopPreview() to stop updating the preview surface.
	        mCamera.stopPreview();
	
	        // Important: Call release() to release the camera for use by other
	        // applications. Applications should release the camera immediately
	        // during onPause() and re-open() it during onResume()).
	        mCamera.release();
	
	        mCamera = null;
	    }
	}


在课程之前，此过程也是 setCamera() 方法的一部分，因此初始化摄像机始终以停止预览开始。


备注：  
翻译：[@JackWaiting](https://github.com/JackWaiting)  
原始文档：[https://developer.android.com/training/camera/cameradirect.html#TaskOrientation](https://developer.android.com/training/camera/cameradirect.html#TaskOrientation)