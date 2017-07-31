#Android提供的工具类
---
Android 提供了一些工具类，在我们creating custom views 时使用，能够比较方便的获得一些数据。

  
参考链接：  
[Android官方文档-custom-views 链接](https://developer.android.com/training/custom-views/index.html "Android官方文档-custom-views")
##1.1 Configuration  
>This class describes all device configuration information that can impact the resources the application retrieves. This includes both user-specified configuration options (locale list and scaling) as well as device configurations (such as input modes, screen size and screen orientation).

>You can acquire this object from Resources, using getConfiguration(). Thus, from an activity, you can get it by chaining the request with getResources():


>`Configuration config = getResources().getConfiguration();`

简而言之：获取设备的一些信息，输入模式，屏幕大小，屏幕方向等等！或者是用户的一些配置信息，locale和scaling等等！


##1.2 ViewConfiguration

>Contains methods to standard constants used in the UI for timeouts, sizes, and distances.

>`ViewConfiguration confg = ViewConfiguration.get(context)`

提供一些自定义控件会用到的标准常量，例如尺寸大小，滑动距离，敏感度等等。。

##1.3 GestureDetector  
用来在onTouchEvent()处理手势

<pre>

比如将Activity的Touch事件交给GestureDetector处理
@Override  
public boolean onTouchEvent(MotionEvent event) {  
     return mGestureDetector.onTouchEvent(event);  
} 


比如将View的Touch事件交给GestureDetector处理
mButton=(Button) findViewById(R.id.button);  
mButton.setOnTouchListener(new OnTouchListener() {            
   @Override  
   public boolean onTouch(View arg0, MotionEvent event) {  
          return mGestureDetector.onTouchEvent(event);  
      }  
});  
</pre>

##1.4 VelocityTracker
用来获取速度。。  
速度 = (终点位置-起点位置)/时间段  
速度可以为负的，说明当前是逆着坐标轴正方向滑动  
<pre>
VelocityTracker vt = VelocityTracker.obtain();  
vt.addMovement(motionEvent);  
vt.computeCurrentVelocity(1000);
vt.getXVelocity();
vt.clear();
vt.recycle();
</pre>


##1.5 Scroller  


scrollBy 最终还是调用了scrollTo  
<pre>
public void scrollBy(int x, int y) {   
       scrollTo(mScrollX + x, mScrollY + y);   
} 
</pre>

**scrollTo 和scrollBy 实际上只是移动了view的内容**  


scrollTo（０,２５），实际上会向上移动25 而不是向下。  

##1.6 ViewDragHelper
用来处理拖拽 子View ，提供了一些辅助方法和与其相关的状态记录。  

<pre>

public class MyLinearLayout extends LinearLayout {
    private ViewDragHelper mViewDragHelper;

    public MyLinearLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        initViewDragHelper();
    }

    //初始化ViewDragHelper
    private void initViewDragHelper() {
        mViewDragHelper = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
            @Override
            public boolean tryCaptureView(View child, int pointerId) {
                return true;
            }

            //处理水平方向的越界
            @Override
            public int clampViewPositionHorizontal(View child, int left, int dx) {
                int fixedLeft;
                View parent = (View) child.getParent();
                int leftBound = parent.getPaddingLeft();
                int rightBound = parent.getWidth() - child.getWidth() - parent.getPaddingRight();

                if (left < leftBound) {
                    fixedLeft = leftBound;
                } else if (left > rightBound) {
                    fixedLeft = rightBound;
                } else {
                    fixedLeft = left;
                }
                return fixedLeft;
            }

            //处理垂直方向的越界
            @Override
            public int clampViewPositionVertical(View child, int top, int dy) {
                int fixedTop;
                View parent = (View) child.getParent();
                int topBound = getPaddingTop();
                int bottomBound = getHeight() - child.getHeight() - parent.getPaddingBottom();
                if (top < topBound) {
                    fixedTop = topBound;
                } else if (top > bottomBound) {
                    fixedTop = bottomBound;
                } else {
                    fixedTop = top;
                }
                return fixedTop;
            }

            //监听拖动状态的改变
            @Override
            public void onViewDragStateChanged(int state) {
                super.onViewDragStateChanged(state);
                switch (state) {
                    case ViewDragHelper.STATE_DRAGGING:
                        System.out.println("STATE_DRAGGING");
                        break;
                    case ViewDragHelper.STATE_IDLE:
                        System.out.println("STATE_IDLE");
                        break;
                    case ViewDragHelper.STATE_SETTLING:
                        System.out.println("STATE_SETTLING");
                        break;
                }
            }

            //捕获View
            @Override
            public void onViewCaptured(View capturedChild, int activePointerId) {
                super.onViewCaptured(capturedChild, activePointerId);
                System.out.println("ViewCaptured");
            }

            //释放View
            @Override
            public void onViewReleased(View releasedChild, float xvel, float yvel) {
                super.onViewReleased(releasedChild, xvel, yvel);
                System.out.println("ViewReleased");
            }
        });
    }

    //将事件拦截交给ViewDragHelper处理
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return mViewDragHelper.shouldInterceptTouchEvent(ev);
    }


    //将Touch事件交给ViewDragHelper处理
    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        mViewDragHelper.processTouchEvent(ev);
        return true;
    }
}
</pre>