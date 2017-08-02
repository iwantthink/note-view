# 自定义View
---
## 1 自定义view的分类
### 1.1 继承View重写onDraw方法
### 1.2 继承ViewGroup派生特殊的Layout
### 1.3 继承特定的View(比如TextView)
### 1.4 继承特定的ViewGroup(比如LinearLayout)

## 2 自定义Vie注意事项
### 2.1 让View支持wrap\_content
wrap\_content根据getChildMeasuredSpec 生成的是AT_MOST模式的measureSpec,默认的View的measure过程，AT\_MOST 和EXACTLY 是同样的处理方式.

### 2.2 让View支持padding
直接继承View的控件，如果不在draw方法中处理padding 那么padding是无法起作用的。 另外继承自ViewGroup的的控件需要在onMeasure onLayout中考虑padding 和子元素的margin对其造成的影响，不然将导致padding和 子元素margin 失效。

### 2.3 尽量不要再View中使用handler，没必要
View内部已经提供了post系列的方法了，完全可以替代Handler使用 

### 2.4 View中如果有线程或者动画，需要及时停止！
停止的时机.. onDetachedFromWindow 。  当包含此View 的Activity退出或者当前View被remove时，会被调用。与此方法对应的是onAttachedToWindow,当包含这个View的Activity启动 ，这个方法会被启动。 当View变得不可见时 我们也需要停止线程 和动画！

### 2.5 View带有滑动嵌套时，需要处理好滑动冲突！