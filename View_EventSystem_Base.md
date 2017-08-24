# View
view是什么呢？它是Android中所有控件的基类，可以理解为所有控件的抽象，同时它有一个子类ViewGroup，代表一组view，个人觉得可以理解为布局，那么就代表了控件可以是单个的也可以是一组的。//TO-DO


---
## view基础知识
1. View的位置参数
2. MotionEvent
3. TouchSlop
4. VelocityTracker
5. GestureDetector
6. view的滑动
7. view的生命周期

## 1.View的位置参数
想要学习 view，首先就得了解Android的坐标体系，android中的坐标原点通常是屏幕的左上角，然后x轴正方向水平向右，Y轴正方向水平向下。如下图：

view在屏幕的位置 可以用left top right bottom来表示，看一下view源码中对left的定义，其他的都类似。需要注意的是这里的坐标都是相对坐标，它是当前view相对于它的父容器的坐标。


> 当前View的左边缘距离它的父容器的左边缘的距离

  

	/**
     * The distance in pixels from the left edge of this view's parent
     * to the left edge of this view.
     * {@hide}
     */
    @ViewDebug.ExportedProperty(category = "layout")
    protected int mLeft;


View的宽高就是通过这四个参数计算出来的
width = right - left;
height = bottom - top

另外在3.0开始，View增加了x,y,translationX,translationY
x和y分别代表View左上角的坐标，translationX/Y代表各自方向上的位移（默认为0）

>x = left + translationX;

>y = top +translationY;


需要注意的是在view平移的过程中，xy是不会改变的，改变的是位移，在属性动画中改变的其实也是translationX/Y这俩个值。

## 2.MotionEvent

android将事件信息封装成这个类，然后给我们你使用，我们可以通过这个类来得到坐标信息和触摸的动作

### 主要的事件类型有:

1. ACTION\_DOWN: 表示用户开始触摸.

2. ACTION\_MOVE: 表示用户在移动(手指或者其他)

3. ACTION\_UP:表示用户抬起了手指

4. ACTION\_CANCEL:表示手势被取消了,一些关于这个事件类型的讨论见:  
[what-causes-a-motionevent-action-cancel-in-android](http://stackoverflow.com/questions/11960861/what-causes-a-motionevent-action-cancel-in-android)

	>**触发条件**:上层 View 是一个 RecyclerView，它收到了一个 ACTION_DOWN 事件，由于这个可能是点击事件，所以它先传递给对应 ItemView，询问 ItemView 是否需要这个事件，然而接下来又传递过来了一个 ACTION\_MOVE 事件，且移动的方向和 RecyclerView 的可滑动方向一致，所以 RecyclerView 判断这个事件是滚动事件，于是要收回事件处理权，这时候对应的 ItemView 会收到一个 ACTION\_CANCEL ，并且不会再收到后续事件。

5. 还有一个不常见的:

	ACTION\_OUTSIDE: 表示用户触碰超出了正常的UI边界.

	
	[参考:OUTSIDE_ACTION](https://stackoverflow.com/questions/8384067/how-to-dismiss-the-dialog-with-click-on-outside-of-the-dialog)  
	>设置视图的 WindowManager 布局参数的 flags为FLAG\_WATCH\_OUTSIDE\_TOUCH，这样点击事件发生在这个视图之外时，该视图就可以接收到一个 ACTION\_OUTSIDE 事件。

    
6. ACTION\_POINTER\_DOWN:有一个非主要的手指按下了.

7. ACTION\_POINTER\_UP:一个非主要的手指抬起来了

另外我们可以通过getX,getY，getRawX,getRawY来获取坐标，前者获取的是相对于当前view的左上角的坐标，后者获取的是相对于手机屏幕左上角的坐标。也可以理解为一个获取手指落在view上的坐标，一个获取手机落在屏幕上的坐标。

### 2.1多点触控  
>getActionMasked()	与 getAction() 类似，多点触控必须使用这个方法获取事件类型。  
getActionIndex()	获取该事件是哪个指针(手指)产生的。  
getPointerCount()	获取在屏幕上手指的个数。  
getPointerId(int pointerIndex)	获取一个指针(手指)的唯一标识符ID，在手指按下和抬起之间ID始终不变。  
findPointerIndex(int pointerId)	通过PointerId获取到当前状态下PointIndex，之后通过PointIndex获取其他内容。  
getX(int pointerIndex)	获取某一个指针(手指)的X坐标  
getY(int pointerIndex)	获取某一个指针(手指)的Y坐标  

#### 2.1.1 getAction 和getActionMasked  


## 3.TouchSlop
它的定义是：系统所能识别出的被认为是滑动的最小距离。

我们可以通过ViewConfiguration.get(getContext()).getScaledTouchSlop()

源码中的位置：frameworks/base/core/res/res/values/config.xml 

		/**
     * Distance a touch can wander before we think the user is scrolling in dips.
     * Note that this value defined here is only used as a fallback by legacy/misbehaving
     * applications that do not provide a Context for determining density/configuration-dependent
     * values.
     *
     * To alter this value, see the configuration resource config_viewConfigurationTouchSlop
     * in frameworks/base/core/res/res/values/config.xml or the appropriate device resource overlay.
     * It may be appropriate to tweak this on a device-specific basis in an overlay based on
     * the characteristics of the touch panel and firmware.
     */
 	   private static final int TOUCH_SLOP = 8;


## 4.VelocityTracker


> 
  Helper for tracking the velocity of touch events, for implementing
  flinging and other such gestures.

用来计算速度的一个帮助类，注意在获取到速度之前：


1.必须先调用computeCurrentVelocity方法（传入的参数的时间单位是ms毫秒）

2.这里速度的定义是 一段时间手指滑动过的像素数，注意速度是可以为负数的（配合android的坐标系），

>  速度 = (终点位置 - 起点位置) /时间段

3.最后在使用完这个类之后，一定要记得使用

	velocityTracker.clear();
	velocityTracker.recycle();

附上代码： 
	

    private VelocityTracker mVelocityTracker = null;

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        
        mVelocityTracker = VelocityTracker.obtain();

        mVelocityTracker.addMovement(event);

        mVelocityTracker.computeCurrentVelocity(1000);

        int xVelocity = (int) mVelocityTracker.getXVelocity();

        int yVelocity = (int) mVelocityTracker.getYVelocity();

        Log.d("VelocityTrackerView", "xVelocity:" + xVelocity);

        Log.d("VelocityTrackerView", "yVelocity:" + yVelocity);

        return true;
    }


## 5.GestureDetector

 > Creates a GestureDetector with the supplied listener.
 > 
 > You may only use this constructor from a UI thread (this is the usual situation).

手势检测，只能在UI线程中创建，用来辅助检测用户的单击，滑动，长按，双击等行为。通过这个类消耗了事件之后需要将结果返回。

使用方法：创建GestureDetector(),并且实现OnGestureListener接口，注意有个小bug的解决,在View的onTouchEvent中，我们将事件传递给GestureDetector 来进行处理，但是如果我在onSingleTapUp中想要消耗这个事件，返回true这个值 没有起到效果，看一下log


	    @Override
    public boolean onTouchEvent(MotionEvent event) {

        boolean resume = mGestureDetector.onTouchEvent(event);

        Log.d("VelocityTrackerView", "resume:" + resume);

        return resume;
    }

>08-14 10:32:01.109    9925-9925/com.marenbo.www.example D/VelocityTrackerView﹕ resume:false

可以看到这时候返回的resume并不是预期中的true.然后我们将onTouchEvent中的值手动设置为true之后，看一下log的值
>08-14 10:37:18.019  12347-12347/com.marenbo.www.example D/VelocityTrackerView﹕ resume:false

>08-14 10:37:18.109  12347-12347/com.marenbo.www.example D/VelocityTrackerView﹕ onSingleTapUp

>08-14 10:37:18.109  12347-12347/com.marenbo.www.example D/VelocityTrackerView﹕ resume:true

我的猜测是：onSingleTapUp只是消耗ACTION\_UP这个事件，但是一个事件肯定是从ACTION\_DOWN开始，然后有零个或者多个ACTION_MOVE，最后是ACTION\_UP结束。然而在ACTION\_DOWN的时候，我们并没有消耗它，所以返回了false，那么接下来同一个事件序列中的事件就都不会给到当前view了。我的解决办法是，如果你需要用到onSingleTapUp这一类的方法，那么就将onDown（）这个回调中返回true，或者你明确所有的事件都要view来解决 可以在onTouchEvent中直接返回true。

	 private GestureDetector mGestureDetector;

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        mGestureDetector = new GestureDetector(this);
        //解决长按屏幕后无法拖动的现象
        mGestureDetector.setIsLongpressEnabled(false);

        boolean resume = mGestureDetector.onTouchEvent(event);

        return resume;
    }

	 /**
     * 手指轻轻触摸屏幕的一瞬间，由一个ACTION_DOWN触发
     */
	@Override
    public boolean onDown(MotionEvent e) {
        return false;
    }

 	/**
     * 手指轻轻触摸屏幕，尚未松开或者拖动，由一个ACTION_DOWN触发
     * 注意和onDown（）的区别，它强调没有松开或者拖动的状态
     */
    @Override
    public void onShowPress(MotionEvent e) {

    }

	/**
     * 手指（轻轻触摸屏幕后）松开,伴随一个MotionEvent.ACTION_UP而触发，这是单击行为
     */
    @Override
    public boolean onSingleTapUp(MotionEvent e) {
        return false;
    }

	/**
     *  手指按下屏幕并拖动,由一个ACTION_DOWN,多个ACTION_MOVE触发，这是拖动行为
     */
    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
        return false;
    }

	/**
     * 用户长久按着屏幕不放，即长按动作
     */
    @Override
    public void onLongPress(MotionEvent e) {

    }

 	 /**
     * 用户按下触摸屏,快速滑动后松开,由一个ACTION_DOWN,多个ACTION_MOVE和一个ACTION_UP触发，这是快速滑动行为
     */
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
        return false;
    }



上面已经可以实现了一些基础的方法，如果要使用双击这个回调还得去实现OnDoubleTapListener接口中的方法

	mGestureDetector.setOnDoubleTapListener(this);


    /**
     * 严格的单击行为
     *  注意它和onSingleTapUp的区别，如果触发了onSingleTapConfirmed,那么后面
     *  不可能再紧跟另一个单击行为，即这只可能是单击，而不可能是双击中的一次单击
     */
    @Override
    public boolean onSingleTapConfirmed(MotionEvent e) {
        return false;
    }

   	/**
     * 双击,由2次连续的组成，它不可能和onSingleTapConfirmed共存
     */
    @Override
    public boolean onDoubleTap(MotionEvent e) {
        return false;
    }

    /**
     * 表示发生了双击行为，在双击的期间，ACTION_DOWN,ACTION_MOVE，ACTION_UP都会触发此回调
     */
    @Override
    public boolean onDoubleTapEvent(MotionEvent e) {
        return false;
    }
 


----------
建议：如果只是监听滑动相关的，建议自己在onTouchEvent中实现，如果要监听双击这种行为的话，就使用GestureDetector

## 6.view的滑动

### 1.scrollTo,scrollBy

scrollBy:

	/**
     * Move the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the amount of pixels to scroll by horizontally
     * @param y the amount of pixels to scroll by vertically
     */
    public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }

scrollTo:

	 /**
     * Set the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the x position to scroll to
     * @param y the y position to scroll to
     */
    public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            invalidateParentCaches();
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            if (!awakenScrollBars()) {
                postInvalidateOnAnimation();
            }
        }
    }


可以看到scrollBy其实也是调用的scrollTo，在滑动过程中mScrollX,mScrollY这俩个值，分别等于**view的左边缘到View内容左边缘的水平方向距离**，View的上边缘到view内容上边缘的垂直方向距离。注意使用scrollTo和scrollBy这俩个方法只能改变view的内容的位置而不能改变view在布局中的位置。  

View的左边缘如果在内容左边缘的 右边，mScrollX 为正，所以scrollTo(正数)其实是往左移动。。

### 2.使用动画
使用view动画，属性动画俩种。

1.使用view动画的话需要注意，不会真正改变view的位置，也就是说View动画是对View的影响做操作，view的位置参数是不会改变的，并且如果我们希望动画完成后的状态保留，还必须将fillAfter属性设置为true，否则在动画结束后，动画的结果会消失。

2.使用属性动画，属性动画通过改变translationX,translationY来改变view的位置。从3.0开始属性动画增加到了android中，3.0之前的版本可以使用动画兼容库nineoldandroid来实现属性动画，不过其本质还是view动画。


### 3.改变布局参数
即改变LayoutParams,如果我们要左移一个view，我们只需要将这个view左边的marginLeft值增加相应的值即可完成移动。另外也可以通过在当前view的左边添加一个空的view，然后增加这个空view的width。

	    ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) getLayoutParams();
        
        params.width+=100;
        
        requestLayout();
        //或者setLayoutParams(params);

### 4.弹性滑动

#### 1.Scroller
弹性滑动对象,用于实现View的弹性滑动.当使用view的ScrollBy,ScrollTo来进行滑动的时候，是瞬间完成的，使用Scroller就不一样了。Scroller使用的代码是固定的。注意这里的滑动 指的是view内容的滑动而不是view本身的位置改变。

        private Scroller mScroller = new Scroller(mContext);

    private void smoothScrollTo(int destX, int destY) {

        int scrollX = getScrollX();

        int delta = destX - scrollX;
        //1000ms内平滑的滑向destX,
        mScroller.startScroll(scrollX, 0, delta, 0, 1000);

        invalidate();
    }

    @Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            
            postInvalidate();
        }
    }


实现流程：
invalidate（），会是的view重绘，在view的draw方法中会去调用computeScroll方法，而computeScroll又会去向Scroller获取当前的scrollX和scrollY然后通过scrollTo实现滑动，最后会调用postInvalidate方法进行第二次重绘，如此反复直到view通过ScrollTo滑动到了新的位置。

工作原理：
Scroller本身是不能实现view的滑动，需要配合View的computeScroll方法才能完成弹性滑动的效果，它不断地让View重绘，而每一次重绘滑动距滑动的起始时间都有一个时间间隔，通过这个时间间隔可以得出View当前的滑动位置，知道了滑动位置可以通过scrollTo方法完成view的滑动，就这样view的每次重绘都会使得view进行小幅度的滑动，许多的小幅度滑动就组成了弹性滑动。

#### 2.通过动画
可以通过属性动画设置一个值到另外一个值 指定时间段内变化。

        ValueAnimator animator = ValueAnimator.ofInt(0, 1).setDuration(1000);

        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {

                float fraction = animation.getAnimatedFraction();

                Log.d("VelocityTrackerActivity", "fraction:" + fraction);
                
                mView.scrollTo(startX+(int)(deltaX * fraction) ,0);

            }
        });

        animator.start();

#### 3.使用延时策略
1.Handler+postInvalidate

2.Thread+sleep

## 7.view的生命周期
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



















