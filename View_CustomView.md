# View的绘制原理
---  

## 1. ViewRoot 和DecorView
ViewRoot对应于ViewRootImpl , 是连接WindowManager 和DecorView的纽带，View绘制的三大流程都是通过ViewRoot来完成的。  
在ActivityThread中，当activity 对象被创建之后，会将DecorView 添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl 对象和 DecorView建立关联。

View的绘制流程从ViewRoot的performTraversals方法开始。经过measure , layout , draw 三个过程 才能最终将一个view绘制出来

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



	