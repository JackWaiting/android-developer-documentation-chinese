# 创建自定义过渡动画


用自定义的形式创建的过渡动画是其它任何内置的过渡类都不能直接实现的。比如你可以定义一个改变文字的前景颜色和输入字段的区域为灰色，表示在此界面中这块区域不可用。这种改变的效果可以帮助用户清楚哪些区域不可用。

自定义的过渡就像内置的过渡类型一样，将动画应用于起始场景和结束场景的子视图中。但是，与内置的过渡类型不同的是：自定义过渡动画必须提供可以获取属性值并生成动画的代码。你也可以为某些控件的子集定义动画。

这一节课将帮助你通过获取属性值和生成动画去创建自定义过渡动画。

## 扩展过渡类
创建一个自定义过渡动画，需要先在项目工程中添加一个类，用来扩展 [Transition](https://developer.android.google.cn/reference/android/transition/Transition.html) 类以及重写它的一些方法。详见下面的代码段：

```java
public class CustomTransition extends Transition {

    @Override
    public void captureStartValues(TransitionValues values) {}

    @Override
    public void captureEndValues(TransitionValues values) {}

    @Override
    public Animator createAnimator(ViewGroup sceneRoot,
                                   TransitionValues startValues,
                                   TransitionValues endValues) {}
}
```
下一节将会详细讲解如何重写这些方法。

## 获取控件的属性值

过渡动画使用 [Property Animation](https://developer.android.google.cn/guide/topics/graphics/prop-animation.html) 描述的属性动画系统。属性动画会在指定的时间内更改起始值和结束值之间的控件属性，所以框架需要同时具有开始和结束时的属性值去构建这个动画。

然后，属性动画通常仅仅需要所有的控件属性值的一小部分。比如，颜色动画需要颜色属性值，而移动动画需要位置属性值。 由于这里动画所需要的属性值是针对于过渡动画的，所以这个框架不再提供每个属性值。而是提供一些回调方法以供过渡动画去获取一些需要的属性值并传递给框架进行保存。

## 获取开始时的值

通过开始时的框架控件值，实现 [capturestartvalues（transitionvalues）](https://developer.android.google.cn/reference/android/transition/Transition.html#captureStartValues(android.transition.TransitionValues)) 方法。框架中每一个在开始场景中控件都会调用此方法，方法的参数是一个包含对控件引用的 [transitionvalues](https://developer.android.google.cn/reference/android/transition/TransitionValues.html) 对象，可以使用 Map 实例保存你需要的控件值。在这个实现里，检索这些属性值并使用 Map 保存，再反馈到框架中。

为了确保一个属性值的键值不与其他 [TransitionValues](https://developer.android.google.cn/reference/android/transition/TransitionValues.html) 键值冲突，需要使用下面的命名方式：

```xml
package_name:transition_name:property_name
```
以下片段显示了 captureStartValues（）方法的实现：

```java
public class CustomTransition extends Transition {

    // Define a key for storing a property value in
    // TransitionValues.values with the syntax
    // package_name:transition_class:property_name to avoid collisions
    private static final String PROPNAME_BACKGROUND =
            "com.example.android.customtransition:CustomTransition:background";

    @Override
    public void captureStartValues(TransitionValues transitionValues) {
        // Call the convenience method captureValues
        captureValues(transitionValues);
    }


    // For the view in transitionValues.view, get the values you
    // want and put them in transitionValues.values
    private void captureValues(TransitionValues transitionValues) {
        // Get a reference to the view
        View view = transitionValues.view;
        // Store its background property in the values map
        transitionValues.values.put(PROPNAME_BACKGROUND, view.getBackground());
    }
    ...
}

```
## 获取结束时的值
针对每一个目标控件，框架需要在结束场景时调用一次 [captureEndValues(TransitionValues)](https://developer.android.google.cn/reference/android/transition/Transition.html#captureEndValues(android.transition.TransitionValues)) 方法。在任意情况下，[captureendvalues()](https://developer.android.google.cn/reference/android/transition/Transition.html#captureEndValues(android.transition.TransitionValues)) 的工作原理与 [capturestartvalues()](https://developer.android.google.cn/reference/android/transition/Transition.html#captureStartValues(android.transition.TransitionValues)) 都是一致的。

下面的代码段展示了 [captureendvalues()](https://developer.android.google.cn/reference/android/transition/Transition.html#captureEndValues(android.transition.TransitionValues)) 方法的实现：

```java
@Override
public void captureEndValues(TransitionValues transitionValues) {
    captureValues(transitionValues);
}
```

在这个例子中，[capturestartvalues()](https://developer.android.google.cn/reference/android/transition/Transition.html#captureStartValues(android.transition.TransitionValues))
 和 [captureendvalues()](https://developer.android.google.cn/reference/android/transition/Transition.html#captureEndValues(android.transition.TransitionValues))
 方法都调用了 *capturevalues()* 去检索和保存数据。每个控件属性都有相同的 *capturevalues()* 检索，但是有不同的开始场景和结束场景。框架为控件的开始和结束时的状态维护了独立的映射。

## 创建自定义动画对象

对一个控件进行过渡在它的起始场景和结束场景之间，通过动画的形式对控件从开始场景到结束场景之间进行改变，需要使用一个动画对象并重写 [createAnimator()](https://developer.android.google.cn/reference/android/transition/Transition.html#createAnimator(android.view.ViewGroup, android.transition.TransitionValues, android.transition.TransitionValues)) 方法。当框架调用此方法时，将会把你获取到的开始时的值和结束时的值传递到根控件和 [TransitionValues](https://developer.android.google.cn/reference/android/transition/TransitionValues.html) 对象中。

这个系统框架调用 [createAnimator()](https://developer.android.google.cn/reference/android/transition/Transition.html#createAnimator(android.view.ViewGroup, android.transition.TransitionValues, android.transition.TransitionValues)) 的次数取决于控件在开始和结束时的场景过程中的变化次数。例如，通过自定义过渡动画实现淡入淡出动画。如果在开始时的场景已知有五个目标动画效果，在结束的场景中移除了其中两个，同时另外三个新的动画效果会添加在新的目标任务。也就是说框架会调用 [createAnimator()](https://developer.android.google.cn/reference/android/transition/Transition.html#createAnimator(android.view.ViewGroup, android.transition.TransitionValues, android.transition.TransitionValues)) 六次：三次淡出和淡入；二次淡出以及一次在新的目标结束场景中的淡入效果。

在开始和结束的场景中，目标控件是一直存在的，框架提供了一个包含开始值参数和结束值参数的  [TransitionValues](https://developer.android.google.cn/reference/android/transition/TransitionValues.html) 对象。针对目标控件只存在于开始场景或者只存在于结束场景的情况，框架也提供了含有对应参数的 [TransitionValues](https://developer.android.google.cn/reference/android/transition/TransitionValues.html) 对象。
当你通过实现 [createAnimator(ViewGroup, TransitionValues, TransitionValues)](https://developer.android.google.cn/reference/android/transition/Transition.html#createAnimator(android.view.ViewGroup, android.transition.TransitionValues, android.transition.TransitionValues)) 方法去创建自定义过渡动画时，需要使用获取到的控件属性值创建动画管理对象并把属性值返回给框架。有一个简单的样例，请参阅 [CustomTransition](https://developer.android.google.cn/samples/CustomTransition/index.html) 示例中的 [ChangeColor](https://developer.android.google.cn/samples/CustomTransition/src/com.example.android.customtransition/ChangeColor.html) 类。有关属性动画的更多信息，请参见 [Property Animation](https://developer.android.google.cn/guide/topics/graphics/prop-animation.html)。

实现 [createanimator（ViewGroup，TransitionValues，TransitionValues）](https://developer.android.google.cn/reference/android/transition/Transition.html#createAnimator(android.view.ViewGroup, android.transition.TransitionValues, android.transition.TransitionValues)) 


## 使用自定义动画
自定义过渡动画和内置过渡类的实现原理类似。你可以参考 [Applying a Transition](https://developer.android.google.cn/training/transitions/transitions.html#Apply) 描述的那样，去使用过渡动画管理器应用自定义过渡动画。

>翻译：[@misparking](https://github.com/misparking)   
>审核：[@iOnesmile](https://github.com/ionesmile)      
原始文档：<https://developer.android.google.cn/training/transitions/custom-transitions.html>
