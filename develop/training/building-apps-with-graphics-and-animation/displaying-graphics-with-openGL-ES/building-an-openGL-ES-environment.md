# 

在 Android 应用中用 OpenGL ES 绘制图形，必须先创建一个 View 的容器。最直接的一种方式是实现 [GLSurfaceView](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html) 和 [GLSurfaceView.Renderer](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.Renderer.html)。 GLSurfaceView 是 OpenGL 绘制图形的 View 容器，GLSurfaceView.Renderer 是对 View 的控制。更多关于这一方面的信息，请见  [OpenGL ES](https://developer.android.google.cn/guide/topics/graphics/opengl.html) 开发指南。

GLSurfaceView 是集成 OpenGL ES 到应用中合适的一种方式。创建一个全屏或进全屏的图形界面，它是很合理的选择。开发者想要集成 OpenGL ES 到 layout 布局中，可以使用 [TextureView](https://developer.android.google.cn/reference/android/view/TextureView.html)。事实上，在自己开发时，使用 [SurfaceView](https://developer.android.google.cn/reference/android/view/SurfaceView.html) 去构建一个 OpenGL ES 也是很好的选择，但是这个选择需要编写大量的代码。

这一课将讲述在 Activity 中用最少的实现 [GLSurfaceView](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html) 和 GLSurfaceView.Renderer 去完成一个简单的应用。

## 在清单文件中声明 OpenGL ES 的使用

按照下面的方式，在应用中使用 OpenGL ES 2.0 API。你必须添加如下的声明到清单文件中：

    <uses-feature android:glEsVersion="0x00020000" android:required="true" />


如果在你应用中使用了压缩方式，你需要声明应用支持的压缩格式，以便于应用只安装在支持的设备上。

    <supports-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
    <supports-gl-texture android:name="GL_OES_compressed_paletted_texture" />


更多关于结构压缩格式的信息，参见 [OpenGL](https://developer.android.google.cn/guide/topics/graphics/opengl.html#textures) 开发者指南。

## 创建 OpenGL ES 图像的 Activity

使用 OpenGL ES 的 Android 应用和其他应用一样有一个用户界面。主要的不用点是其它应用给 Activity 设置的是 layout 布局。很多应用可能有使用 [TextView](https://developer.android.google.cn/reference/android/widget/TextView.html), [Button](https://developer.android.google.cn/reference/android/widget/Button.html) 和 [ListView](https://developer.android.google.cn/reference/android/widget/ListView.html)，当要使用 OpenGL ES 时也能添加 GLSurfaceView。

下面的代码是使用 GLSurfaceView 最少实现的示例：


    public class OpenGLES20Activity extends Activity {
    
        private GLSurfaceView mGLView;
    
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
    
            // Create a GLSurfaceView instance and set it
            // as the ContentView for this Activity.
            mGLView = new MyGLSurfaceView(this);
            setContentView(mGLView);
        }
    }
    
> 注：OpenGL ES 2.0 请求的最小版本是 Android  2.2（API 8）或更高，因此请调整项目的 API。


## 构建 GLSurfaceView 对象

GLSurfaceView 是针对 OpenGL ES 图形绘制的 View。但是它本身并没有做太多，通常需要设置 GLSurfaceView.Renderer 来绘制对象。事实上，这个对象的代码非常的少，你可能有兴趣不继承它而自己创建一个 GLSurfaceView 对象，但是不会这么做。你需要继承这个类，并重写触摸事件，涉及到 [响应触摸事件](https://developer.android.google.cn/training/graphics/opengl/environment.html#touch.html) 这一课。

下面是快速的实现 GLSurfaceView 必备的最少代码，通常会在 Activity 里面创建一个内部内：


    class MyGLSurfaceView extends GLSurfaceView {
    
        private final MyGLRenderer mRenderer;
    
        public MyGLSurfaceView(Context context){
            super(context);
    
            // Create an OpenGL ES 2.0 context
            setEGLContextClientVersion(2);
    
            mRenderer = new MyGLRenderer();
    
            // Set the Renderer for drawing on the GLSurfaceView
            setRenderer(mRenderer);
        }
    }

其它的操作是给 GLSurfaceView 设置渲染模式，只有当数据改变时才去绘制的模式是 [GLSurfaceView.RENDERMODE_WHEN_DIRTY](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html#RENDERMODE_WHEN_DIRTY)：


    // Render the view only when there is a change in the drawing data
    setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);

这个设置防止 GLSurfaceView 从开始到你调用 [requestRender()](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html#requestRender()) 反复绘制的问题，这个例子更有效。

## 构建和渲染类

使用 OpenGL ES 时需要实现 GLSurfaceView.Renderer 去做一些有趣的事情，这个类与 GLSurfaceView 绑定去控制它的绘制。下面是三个在 Android 系统中按序在 GLSurfaceView 绘制时被回调的方法：

- [onSurfaceCreated()](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceCreated(javax.microedition.khronos.opengles.GL10,javax.microedition.khronos.egl.EGLConfig)) - 在 view 的 OpenGL ES 环境创建时被调用。

- [onDrawFrame()](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.Renderer.html#onDrawFrame(javax.microedition.khronos.opengles.GL10)) - 在 View 绘制或重新绘制时的回调。

- [onSurfaceChanged()](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceChanged(javax.microedition.khronos.opengles.GL10,int,int)) - 在 View 的结构改变时回调，如设备屏幕方向改变。

这是 OpenGL ES 非常基础渲染的实现，在 GLSurfaceView 中绘制一个黑色背景没有更多：


    public class MyGLRenderer implements GLSurfaceView.Renderer {
    
        public void onSurfaceCreated(GL10 unused, EGLConfig config) {
            // Set the background frame color
            GLES20.glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
        }
    
        public void onDrawFrame(GL10 unused) {
            // Redraw background color
            GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
        }
    
        public void onSurfaceChanged(GL10 unused, int width, int height) {
            GLES20.glViewport(0, 0, width, height);
        }
    }
    
上面这些是实现它的所有步骤！上面的代码是使用 OpenGL 创建了简单的显示黑屏的应用。目前实现的这些代码还没有做任何有趣的事情，但是是使用 OpenGL 绘制最基本的需求。

注：你可能想知道在使用 OpengGL ES 2.0 API 时，为什么这些方法有 [GL10](https://developer.android.google.cn/reference/javax/microedition/khronos/opengles/GL10.html) 参数。这些方法标记是为了在 2.0 API 上简单的重复使用，并保持 Android 框架代码更简洁。

如果你熟悉了 OpenGL ES API，那么现在你应该可以搭建 OpenGL ES 环境和开始绘制图像。然而，如果需要更多的 OpenGL 帮助信息，继续下一课获取更多信息。



>翻译：[@iOnesmile](https://github.com/iOnesmile)       
原始文档：<https://developer.android.google.cn/training/graphics/opengl/environment.html#manifest>
