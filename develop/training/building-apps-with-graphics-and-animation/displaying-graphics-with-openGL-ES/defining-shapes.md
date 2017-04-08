# 定义 Shapes

能够在 OpenGL ES 视图的上下文中定义要绘制的形状是创建高端图形杰作的第一步。如果没有一些 OpenGL ES 定义图形对象的基础知识，绘制图形可能会有一点困难。

<small>这节课讲解了相对于 Android 设备屏幕 OpenGL ES 的坐标系，定义形状和形状表面的基本知识，如定义一个三角形和一个下文形。</small>

## 定义一个三角形
OpenGL ES allows you to define drawn objects using coordinates in three-dimensional space. So, before you can draw a triangle, you must define its coordinates. In OpenGL, the typical way to do this is to define a vertex array of floating point numbers for the coordinates. For maximum efficiency, you write these coordinates into a ByteBuffer, that is passed into the OpenGL ES graphics pipeline for processing.

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
By default, OpenGL ES assumes a coordinate system where [0,0,0] (X,Y,Z) specifies the center of the GLSurfaceView frame, [1,1,0] is the top right corner of the frame and [-1,-1,0] is bottom left corner of the frame. For an illustration of this coordinate system, see the OpenGL ES developer guide.

Note that the coordinates of this shape are defined in a counterclockwise order. The drawing order is important because it defines which side is the front face of the shape, which you typically want to have drawn, and the back face, which you can choose to not draw using the OpenGL ES cull face feature. For more information about faces and culling, see the OpenGL ES developer guide.

默认情况下，OpenGL ES 假定一个坐标系，[0,0,0] (X,Y,Z) 为 GLSurfaceView 框的中心点，[1,1,0] 是指右上角，[-1,-1,0] 即左下角。想要坐标系的插图，可以参考[OpenGL ES]()开发指南。