# 定义 Shapes

能够在 OpenGL ES 视图的上下文中定义要绘制的形状是创建高端图形杰作的第一步。如果没有一些 OpenGL ES 定义图形对象的基础知识，绘制图形可能会有一点困难。

<small>这节课讲解了相对于 Android 设备屏幕 OpenGL ES 的坐标系，定义形状和形状表面的基本知识，如定义一个三角形和一个下文形。</small>

## 定义一个三角形
OpenGL ES 允许在三维坐标里定义图像。所以，在画三角形前，必须定义坐标。在OpenGL ES里，要做到这一点的典型方法是定义一个浮点型的顶点数组。为获得做大效率，将坐标写进 [ByteBuffer](https://developer.android.google.cn/reference/java/nio/ByteBuffer.html) 里，然后传给 OpenGL ES 的图形处理流程中。

```java
public class Triangle {

    private FloatBuffer vertexBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float triangleCoords[] = {   // in counterclockwise order:
             0.0f,  0.622008459f, 0.0f, // top
            -0.5f, -0.311004243f, 0.0f, // bottom left
             0.5f, -0.311004243f, 0.0f  // bottom right
    };

    // Set color with red, green, blue and alpha (opacity) values
    float color[] = { 0.63671875f, 0.76953125f, 0.22265625f, 1.0f };

    public Triangle() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
                // (number of coordinate values * 4 bytes per float)
                triangleCoords.length * 4);
        // use the device hardware's native byte order
        bb.order(ByteOrder.nativeOrder());

        // create a floating point buffer from the ByteBuffer
        vertexBuffer = bb.asFloatBuffer();
        // add the coordinates to the FloatBuffer
        vertexBuffer.put(triangleCoords);
        // set the buffer to read the first coordinate
        vertexBuffer.position(0);
    }
}
```
默认情况下，OpenGL ES 假定一个坐标系，其中 [0,0,0]*(X,Y,Z)* 为 [GLSurfaceView](https://developer.android.google.cn/reference/android/opengl/GLSurfaceView.html) 的中心点，[1,1,0] 是指右上角，[-1,-1,0] 即左下角。想要了解更多的坐标系的图解说明，可以参考 [OpenGL ES](https://developer.android.google.cn/guide/topics/graphics/opengl.html#coordinate-mapping) 开发指南。    

注意，这种形状的坐标以逆时针顺序被定义。绘制顺序很重要，因为它定义了你要想绘制的哪一面是形状的正面，以及可以选择使用 OpenGL ES 不绘制背面。有关更多信息，请参阅 OpengGL ES 的开发者指南。

##定义正方形
在 OpenGL ES 里定义三角形是比较简单的，但如果你想绘制点更复杂的形状呢？比如，一个正方形？有许多方法可以做到这一点，但以在 OpenGL ES 绘制的形状的典型路径是使用两个三角形绘制： 
    
![image](ccw-square.png)    
图 1. 使用两个三角形绘制正方形    

再次，你应该定义一个逆时针顺序的顶点表示这种形状的三角形，并把值存到 ByteBuffer。为了避免定义由每个三角形使用两次共同坐标，使用绘图列表告诉 OpenGL ES 的图形管道如何绘制这些顶点。下面是这个形状的代码： 
   
```java
	public class Square {

    private FloatBuffer vertexBuffer;
    private ShortBuffer drawListBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float squareCoords[] = {
            -0.5f,  0.5f, 0.0f,   // top left
            -0.5f, -0.5f, 0.0f,   // bottom left
             0.5f, -0.5f, 0.0f,   // bottom right
             0.5f,  0.5f, 0.0f }; // top right

    private short drawOrder[] = { 0, 1, 2, 0, 2, 3 }; // order to draw vertices

    public Square() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 4 bytes per float)
                squareCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(squareCoords);
        vertexBuffer.position(0);

        // initialize byte buffer for the draw list
        ByteBuffer dlb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 2 bytes per short)
                drawOrder.length * 2);
        dlb.order(ByteOrder.nativeOrder());
        drawListBuffer = dlb.asShortBuffer();
        drawListBuffer.put(drawOrder);
        drawListBuffer.position(0);
    }
}
```

这个例子告诉你用 OpenGL 创建更复杂的形状。一般情况下，用三角形集合绘制对象。在下一课中，你将学习如何在屏幕上绘制这些形状。
>翻译：[@misparking](https://github.com/misparking)    
原始文档：<https://developer.android.google.cn/training/graphics/opengl/shapes.html>