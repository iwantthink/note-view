# 自定义View
---
## 1.自定义view的分类
### 1.1 继承View重写onDraw方法
### 1.2 继承ViewGroup派生特殊的Layout
### 1.3 继承特定的View(比如TextView)
### 1.4 继承特定的ViewGroup(比如LinearLayout)

## 2.自定义Vie注意事项
### 2.1 让View支持wrap\_content
wrap\_content根据getChildMeasuredSpec 生成的是AT_MOST模式的measureSpec,默认的View的measure过程，AT\_MOST 和EXACTLY 是同样的处理方式.

### 2.2 让View支持padding
直接继承View的控件，如果不在draw方法中处理padding 那么padding是无法起作用的。 另外继承自ViewGroup的的控件需要在onMeasure onLayout中考虑padding 和子元素的margin对其造成的影响，不然将导致padding和 子元素margin 失效。

### 2.3 尽量不要再View中使用handler，没必要
View内部已经提供了post系列的方法了，完全可以替代Handler使用 

### 2.4 View中如果有线程或者动画，需要及时停止！
停止的时机.. onDetachedFromWindow 。  当包含此View 的Activity退出或者当前View被remove时，会被调用。与此方法对应的是onAttachedToWindow,当包含这个View的Activity启动 ，这个方法会被启动。 当View变得不可见时 我们也需要停止线程 和动画！

### 2.5 View带有滑动嵌套时，需要处理好滑动冲突！

## 3.自定义属性的使用
1. 在values目录下面 创建自定义属性的XML文件，例如attrs.xml,DictView 是这个自定义属性集合的名字，然后这个集合里面可以有很多的自定义属性。 自定义属性的包括名字和格式
	    
		<declare-styleable name="DictView">
        	<attr name="sbName" format="string"></attr>
        	<attr name="sbAge" format="integer"></attr>
    	</declare-styleable>

2. 需要在自定义view的 构造方法中，解析自定义属性

		TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.DictView);
        String str = a.getString(R.styleable.DictView_sbName);
3. 直接在布局文件中使用,需要注意：为了使用自定义属性，必须得在布局文件中添加schemas声明xmlns:dict="http://schemas.android.com/apk/res-auto"



## 4 绘制流程

![](http://ww1.sinaimg.cn/large/6ab93b35gy1fikjvk8monj20fc0heaay.jpg)

**生命周期:**

	public class LifeCycleView extends View {

    private static final String TAG = LifeCycleView.class.getSimpleName();

    public LifeCycleView(Context context) {
        super(context);

        Log.d(TAG, "construction with one parameter");
    }

    public LifeCycleView(Context context, AttributeSet attrs) {
        super(context, attrs);

        Log.d(TAG, "construction with two parameter");
    }

    //xml布局被view完全解析了之后会调用
    //也就是说 如果从代码中创建view 不会回调该接口
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        Log.d(TAG, "onFinishInflate");
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        Log.d(TAG, "onMeasure");
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);

        Log.d(TAG, "onLayout");
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        Log.d(TAG, "onDraw");
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);

        Log.d(TAG, "onSizeChanged");
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();

        Log.d(TAG, "onAttachedToWindow");
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();

        Log.d(TAG, "onDetachedFromWindow");
    }

    @Override
    protected void onWindowVisibilityChanged(int visibility) {
        super.onWindowVisibilityChanged(visibility);

        Log.d(TAG, "onWindowVisibilityChanged");
    }
	}

看一下log：

	08-14 17:45:58.279  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ construction with two parameter
	08-14 17:45:58.279  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onFinishInflate
	08-14 17:45:58.329  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onAttachedToWindow
	08-14 17:45:58.329  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onWindowVisibilityChanged
	08-14 17:45:58.329  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onMeasure
	08-14 17:45:58.329  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onMeasure
	08-14 17:45:58.469  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onSizeChanged
	08-14 17:45:58.469  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onLayout
	08-14 17:45:58.629  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onMeasure
	08-14 17:45:58.629  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onMeasure
	08-14 17:45:58.629  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onLayout
	08-14 17:45:58.629  12523-12523/com.marenbo.www.example D/LifeCycleView﹕ onDraw

## 5 重要函数介绍
[官方API文档---VIEW](https://developer.android.com/reference/android/view/View.html)
### 5.1 构造函数
	//在代码中创建时使用
	public void SampleView(Context context) {}
	//在xml中使用时使用,关于属性会通过attrs传入
	public void SampleView(Context context, AttributeSet attrs) {}  
	public void SampleView(Context context, AttributeSet attrs, int defStyleAttr) {}  
	public void SampleView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}   


### 5.2 onSizeChanged
在视图大小发生改变时调用

### 5.3 setPivotX()---setPivotY()
设置View旋转或缩放的中心坐标