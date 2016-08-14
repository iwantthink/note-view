#View的事件分发机制
---
###1.public boolean dispatchTouchEvent(MotionEvent event)
用来进行事件的分发。如果事件能够传递给当前view，那么此方法就一定会调用（所以我们可能在这里处理一些一定能执行到的逻辑）。返回结果受到当前view的onTouchEvent和下级view的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

###2.public boolean onInterceptTouchEvent(MotionEvent event)
在dispatchTouchEvent方法中被调用，用来判断是否拦截某个事件，如果当前view拦截了某个事件，那么在同一个事件序列中，当前方法不会再次被调用，返回结果表示是否拦截当前事件。
###3.public boolean onTouchEvent(MotionEvent event)
在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前view无法再次接收到事件。

下面用一段伪代码表示的逻辑(ViewGroup的逻辑)：

	public boolean dispatchTouchEvent(MotionEvent ev){

		boolean consume = false;

		if(onInterceptTouchEvent(ev))

			consume = onTouchEvent(ev);

		else

			consume = child.dispatchTouchEvent(ev);

		return consume;
	}
	

##View中Listener的优先级
当一个view需要处理事件时，如果它设置了onTouchListener，那么onTouchListener中的onTouch方法会被回调。这时事件如何处理还要看onTouch的返回值，如果返回false，那么当前view的onTouchEvent方法会被调用；如果返回true，那么onTouchEvent将不会再被调用！

在onTouchEvent方法中，如果当前设置有onClickListener，那么它的onClick方法会被调用。




##View中的事件传递顺序
当一个点击事件产生之后，它的传递过程遵循如下顺序:Activity->Window->View，即事件总是先传递给Activity，Activity再传递给Window，最后Window再传递给顶级View。顶级View接收到事件后，就会按照事件分发机制去分发事件。

如果一个View的onTouchEvent返回false，那么它的父容器的onTouchEvent将会被调用，以此类推。如果所有的元素都不处理这个事件，那么这个事件将会最终传递给Activity处理，即Activity的onTouchEvent方法会被调用。

##事件传递的一些结论：
1.同一个事件序列是指从手指接触屏幕开始，到手指离开屏幕那一刻结束，在这个过程当中所产生的一系列事件，这个事件序列以down事件开始，中间含有数量不定的move事件，最终以up事件结束。

2.正常情况下，一个事件序列只能被一个View拦截且消耗，这一条可以参考（3），因为一旦一个元素拦截了某个事件，那么同一个事件序列中的所有事件都会直接交给它处理，因此同一个事件序列中的事件不能分别由俩个View同时处理，但是通过特殊的手段也是可以实现的，例如一个view将本该自己处理的事件通过onTouchEvent强行传递给其他view进行处理。

3.某个View一旦决定拦截，那个这一个事件序列都会交给它来处理（如果事件序列能够传递给它），并且它的onTnterceptTouchEvent不会再被调用。这条也很好理解，就是说当一个view决定拦截一个事件后，那么系统会把同一个事件序列内的其他方法都直接交给它处理，因此就不用在调用这个view的onInterceptTouchEvent去询问它是否要拦截了。

4.某个view一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回了false），那么同一个事件序列中的其他事件都不会再交给它来处理，并且事件将重新交由它的父元素去处理，即父元素的onTouchEvent会被调用。意思就是说，如果事件交给了一个view来处理，那么它就一定要消耗掉，否则接下来同一个事件序列中剩下的事件就不再交给它处理了。

5.如果view不消耗除ACTION_DOWN意外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前view可以持续收到后续的事件，最终这些消失的点击事件会传递给Activity处理。

6.ViewGroup默认不拦截任何事件！Android源码中ViewGroup的onInterceptTouchEvent方法默认返回的是false。

7.view没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。

8.View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longclickable同时都为false）。View的clickable属性默认都为false，clickable属性要分情况，比如button的clickable默认为true，而textview的clickable属性默认为false。

9.View的enable属性不影响onTouchEvent的默认返回值。哪怕一个view是disable状态的，只要它的clickable和longclickable 有一个为true，那么它的onTouchEvent就返回true。

10.onClick会发生的前提是当前的view可点击，并且它收到了down和up的事件。

11.事件传递过程是由外向内的，即事件总是先传递给父元素，然后再由父元素分发给子view，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。