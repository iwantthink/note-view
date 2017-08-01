# View的绘制原理
---  

## 1. ViewRoot 和DecorView
- ViewRoot对应于ViewRootImpl , 是连接WindowManager 和DecorView的纽带，View绘制的三大流程都是通过ViewRoot来完成的。  

- 在ActivityThread中，当activity 对象被创建之后，会将DecorView 添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl 对象和 DecorView建立关联。

- View的绘制流程从ViewRoot的**performTraversals**方法开始,然后会依次调用performMeasure,performLayout,performDraw 。  经过measure , layout , draw 三个过程 才能最终将一个view绘制出来 。  
例如:ViewGroup(performMeasure->measure->onMeasure)->View(measure),在onMeasure中会对所有的子类进行遍历并调用子类的measure

- measure的过程结束后，可以获得测量宽高，getMeasureWidth,几乎所有情况下 测量宽高都等同于 最终宽高。  
- performDraw的遍历过程是在draw方法中通过dispatchDraw来实现  
- Layout过程决定了View的四个顶点的坐标和实际的View的宽高，可以通过getWidth getHeight方法获取最终宽高
- android.R.id.content 是DecorView的content 部分的id




## 2. MeasureSpec基础知识

>A MeasureSpec encapsulates the layout requirements passed from parent to child.Each MeasureSpec represents a requirement for either the width or the height.A MeasureSpec is comprised of a size and a mode.

1. MeasureSpec封装了父布局传递给子View的布局要求  
2. MeasureSpec可以表示宽和高  
3. MeasureSpec由size和mode组成  

  
### 2.1 MeasureSpec 组成
MeasureSpec 是一个32位的int数据，高2位 代表SpecMdoe即某种测量模式，低30位为SpecSize 代表该模式下的大小信息  

- **获取size:** 
`MeasureSpec.getSize(measureSpec)`

- **获取mode:**
`MeasureSpec.getMode(measureSpec)`

- **生成specMode**
`MeasureSpec.makeMeasureSpec(size,mode)`
  
### 2.2 SpecMode类型
MeasureSpec.EXACTLY , MeasureSpec.AT_MOST ，MeasureSpec.UNSPECIFIED

#### 2.2.1 MeasureSpec.EXACTLY
>The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.

父容器已经检测出子类view所需要的精确大小。  
该模式下，View的测量大小即为SpecSize  

#### 2.2.2 MeasureSpec.AT_MOST

>The child can be as large as it wants up to the specified size.

父容器未检测出子View所需要的精确大小，但是指定了一个可用大小即SpecSize  
该模式下，View的测量大小最大不能超过SpecSize  

#### 2.2.3 MeasureSpec.UNSPECIFIED
>The parent has not imposed any constraint on the child. It can be whatever size it wants.

父容器不对子View做限制  
MeasureSpec.UNSPECIFIED这种模式一般用作Android系统内部，或者ListView和ScrollView等滑动控件


### 2.3 MeasureSpec 和LayoutParams 的关系 
LayoutParams 需要和父容器一起才能决定view的MeasureSpec。  

- 对于顶级DecorView来说，其MeasureSpec由窗口的尺寸和自身LayoutParams 共同决定.   
在ViewRootImpl 的measureHierarchy 中 有如下一段代码，展示了DecorView的MeasureSpec的创建过程

		childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
    	childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
   		performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);



    	private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }


- 对于普通view来说，view的measure过程由viewGroup传递而来，首先可以看下viewGroup的measureChildWithMargins方法

	    protected void measureChildWithMargins(View child,
        int parentWidthMeasureSpec, int widthUsed,
        int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    	}
该方法在调用子元素的measure之前，会先通过getChildMeasureSpec方法来获得子元素的MeasureSpec 。


	    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
   	    }
可以通过以上代码知道 **子元素的MeasureSpec 主要是根据 父容器的MeasureSpec和子元素本身的LayoutParams来组合而成的**！


简单的总结下：  
1. 当View采用固定宽高的时候，View的MeasureSpec都是精确模式切大小遵循LayoutParams的大小。     
2. 当View的宽高是match\_parent 如果父容器的模式是精准模式，那么view也是精准模式并且其大小是父容器的剩余空间,如果父容器的模式是最大模式，那么View也是最大模式，并且其大小不会超过父容器的剩余空间   
3. 当View的宽高是wrap\_content时，不管父容器是精准还是最大化，View的模式总是最大化，并且大小不可以超过父容器的剩余空间
 

## 3. View的工作流程
### 3.1 measure过程
measure过程要分情况，一种是View，那么通过measure方法就完成了其测量过程，一种是viewGroup，那么除了完成自己的测量过程，还需要去遍历调用所有子元素的measure方法。 

- View的measure 过程  

		protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    	}




   		public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    	}

	这里的specSize 就是测量后的View的大小,从中可以看出默认情况下AT\_MOST和EXACTLY 是同一种处理方式,根据getChildMeasureSpec方法，可以得出wrap\_content和match\_parent的specSize都是取父容器剩余空间大小，再根据这里的getDefaultSize处理方式。得出一个结论，如果自定义了一个view 确不对其wrap\_content 进行特殊处理，那么就会得到match\_parent一样的效果。 

  		protected int getSuggestedMinimumHeight() {
        return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());

    	}
	getSuggestedMinimumHeight()方法的逻辑：如果View没有设置背景，那么返回android:minWidth属性设置的值，这个值可以为0。如果有设置背景，则返回 minWidth和背景的最小宽度这俩者中的最大值，其实就是View在UNSPECIFIED情况下的测量宽高.