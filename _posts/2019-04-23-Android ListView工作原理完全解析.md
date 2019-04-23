---
layout: post
title: "Android ListView工作原理完全解析"
tags: [Android, ListView]
comments: true
---

在Android所有常用的原生控件当中，用法最复杂的应该就是ListView了，它专门用于处理那种内容元素很多，手机屏幕无法展示出所有内容的情况。ListView可以使用列表的形式来展示内容，超出屏幕部分的内容只需要通过手指滑动就可以移动到屏幕内了。

另外ListView还有一个非常神奇的功能，我相信大家应该都体验过，即使在ListView中加载非常非常多的数据，比如达到成百上千条甚至更多，ListView都不会发生OOM或者崩溃，而且随着我们手指滑动来浏览更多数据时，程序所占用的内存竟然都不会跟着增长。那么ListView是怎么实现这么神奇的功能的呢？

# 继承结构
---
首先我们先来看一下ListView的继承结构，如下图所示：

![img](/images/24/1.png)

可以看到，ListView的继承结构还是相当复杂的，它是直接继承自的AbsListView，而AbsListView有两个子实现类，一个是ListView，另一个就是GridView，因此我们从这一点就可以猜出来，ListView和GridView在工作原理和实现上都是有很多共同点的。然后AbsListView又继承自AdapterView，AdapterView继承自ViewGroup，后面就是我们所熟知的了。先把ListView的继承结构了解一下，待会儿有助于我们更加清晰地分析代码。

# Adapter的作用
---
Adapter相信大家都不会陌生，我们平时使用ListView的时候一定都会用到它。那么话说回来大家有没有仔细想过，为什么需要Adapter这个东西呢？总感觉正因为有了Adapter，ListView的使用变得要比其它控件复杂得多。那么这里我们就先来学习一下Adapter到底起到了什么样的一个作用。

其实说到底，控件就是为了交互和展示数据用的，只不过ListView更加特殊，它是为了展示很多很多数据用的，但是ListView只承担交互和展示工作而已，至于这些数据来自哪里，ListView是不关心的。因此，我们能设想到的最基本的ListView工作模式就是要有一个ListView控件和一个数据源。

不过如果真的让ListView和数据源直接打交道的话，那ListView所要做的适配工作就非常繁杂了。因为数据源这个概念太模糊了，我们只知道它包含了很多数据而已，至于这个数据源到底是什么样类型，并没有严格的定义，有可能是数组，也有可能是集合，甚至有可能是数据库表中查询出来的游标。所以说如果ListView真的去为每一种数据源都进行适配操作的话，一是扩展性会比较差，内置了几种适配就只有几种适配，不能动态进行添加。二是超出了它本身应该负责的工作范围，不再是仅仅承担交互和展示工作就可以了，这样ListView就会变得比较臃肿。

那么显然Android开发团队是不会允许这种事情发生的，于是就有了Adapter这样一个机制的出现。顾名思义，Adapter是适配器的意思，它在ListView和数据源之间起到了一个桥梁的作用，ListView并不会直接和数据源打交道，而是会借助Adapter这个桥梁来去访问真正的数据源，与之前不同的是，Adapter的接口都是统一的，因此ListView不用再去担心任何适配方面的问题。而Adapter又是一个接口(interface)，它可以去实现各种各样的子类，每个子类都能通过自己的逻辑来去完成特定的功能，以及与特定数据源的适配操作，比如说ArrayAdapter可以用于数组和List类型的数据源适配，SimpleCursorAdapter可以用于游标类型的数据源适配，这样就非常巧妙地把数据源适配困难的问题解决掉了，并且还拥有相当不错的扩展性。简单的原理示意图如下所示：

![img](/images/24/2.png)

当然Adapter的作用不仅仅只有数据源适配这一点，还有一个非常非常重要的方法也需要我们在Adapter当中去重写，就是getView()方法，这个在下面的文章中还会详细讲到。

# RecycleBin机制
---
那么在开始分析ListView的源码之前，还有一个东西是我们提前需要了解的，就是RecycleBin机制，这个机制也是ListView能够实现成百上千条数据都不会OOM最重要的一个原因。其实RecycleBin的代码并不多，只有300行左右，它是写在AbsListView中的一个内部类，所以所有继承自AbsListView的子类，也就是ListView和GridView，都可以使用这个机制。那我们来看一下RecycleBin中的主要代码，如下所示：
```java

/**
 * The RecycleBin facilitates reuse of views across layouts. The RecycleBin
 * has two levels of storage: ActiveViews and ScrapViews. ActiveViews are
 * those views which were onscreen at the start of a layout. By
 * construction, they are displaying current information. At the end of
 * layout, all views in ActiveViews are demoted to ScrapViews. ScrapViews
 * are old views that could potentially be used by the adapter to avoid
 * allocating views unnecessarily.
 * 
 * @see android.widget.AbsListView#setRecyclerListener(android.widget.AbsListView.RecyclerListener)
 * @see android.widget.AbsListView.RecyclerListener
 */
class RecycleBin {
	private RecyclerListener mRecyclerListener;
 
	/**
	 * The position of the first view stored in mActiveViews.
	 */
	private int mFirstActivePosition;
 
	/**
	 * Views that were on screen at the start of layout. This array is
	 * populated at the start of layout, and at the end of layout all view
	 * in mActiveViews are moved to mScrapViews. Views in mActiveViews
	 * represent a contiguous range of Views, with position of the first
	 * view store in mFirstActivePosition.
	 */
	private View[] mActiveViews = new View[0];
 
	/**
	 * Unsorted views that can be used by the adapter as a convert view.
	 */
	private ArrayList<View>[] mScrapViews;
 
	private int mViewTypeCount;
 
	private ArrayList<View> mCurrentScrap;
 
	/**
	 * Fill ActiveViews with all of the children of the AbsListView.
	 * 
	 * @param childCount
	 *            The minimum number of views mActiveViews should hold
	 * @param firstActivePosition
	 *            The position of the first view that will be stored in
	 *            mActiveViews
	 */
	void fillActiveViews(int childCount, int firstActivePosition) {
		if (mActiveViews.length < childCount) {
			mActiveViews = new View[childCount];
		}
		mFirstActivePosition = firstActivePosition;
		final View[] activeViews = mActiveViews;
		for (int i = 0; i < childCount; i++) {
			View child = getChildAt(i);
			AbsListView.LayoutParams lp = (AbsListView.LayoutParams) child.getLayoutParams();
			// Don't put header or footer views into the scrap heap
			if (lp != null && lp.viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
				// Note: We do place AdapterView.ITEM_VIEW_TYPE_IGNORE in
				// active views.
				// However, we will NOT place them into scrap views.
				activeViews[i] = child;
			}
		}
	}
 
	/**
	 * Get the view corresponding to the specified position. The view will
	 * be removed from mActiveViews if it is found.
	 * 
	 * @param position
	 *            The position to look up in mActiveViews
	 * @return The view if it is found, null otherwise
	 */
	View getActiveView(int position) {
		int index = position - mFirstActivePosition;
		final View[] activeViews = mActiveViews;
		if (index >= 0 && index < activeViews.length) {
			final View match = activeViews[index];
			activeViews[index] = null;
			return match;
		}
		return null;
	}
 
	/**
	 * Put a view into the ScapViews list. These views are unordered.
	 * 
	 * @param scrap
	 *            The view to add
	 */
	void addScrapView(View scrap) {
		AbsListView.LayoutParams lp = (AbsListView.LayoutParams) scrap.getLayoutParams();
		if (lp == null) {
			return;
		}
		// Don't put header or footer views or views that should be ignored
		// into the scrap heap
		int viewType = lp.viewType;
		if (!shouldRecycleViewType(viewType)) {
			if (viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
				removeDetachedView(scrap, false);
			}
			return;
		}
		if (mViewTypeCount == 1) {
			dispatchFinishTemporaryDetach(scrap);
			mCurrentScrap.add(scrap);
		} else {
			dispatchFinishTemporaryDetach(scrap);
			mScrapViews[viewType].add(scrap);
		}
 
		if (mRecyclerListener != null) {
			mRecyclerListener.onMovedToScrapHeap(scrap);
		}
	}
 
	/**
	 * @return A view from the ScrapViews collection. These are unordered.
	 */
	View getScrapView(int position) {
		ArrayList<View> scrapViews;
		if (mViewTypeCount == 1) {
			scrapViews = mCurrentScrap;
			int size = scrapViews.size();
			if (size > 0) {
				return scrapViews.remove(size - 1);
			} else {
				return null;
			}
		} else {
			int whichScrap = mAdapter.getItemViewType(position);
			if (whichScrap >= 0 && whichScrap < mScrapViews.length) {
				scrapViews = mScrapViews[whichScrap];
				int size = scrapViews.size();
				if (size > 0) {
					return scrapViews.remove(size - 1);
				}
			}
		}
		return null;
	}
 
	public void setViewTypeCount(int viewTypeCount) {
		if (viewTypeCount < 1) {
			throw new IllegalArgumentException("Can't have a viewTypeCount < 1");
		}
		// noinspection unchecked
		ArrayList<View>[] scrapViews = new ArrayList[viewTypeCount];
		for (int i = 0; i < viewTypeCount; i++) {
			scrapViews[i] = new ArrayList<View>();
		}
		mViewTypeCount = viewTypeCount;
		mCurrentScrap = scrapViews[0];
		mScrapViews = scrapViews;
	}
}
```
这里的RecycleBin代码并不全，我只是把最主要的几个方法提了出来。那么我们先来对这几个方法进行简单解读，这对后面分析ListView的工作原理将会有很大的帮助。

* fillActiveViews() 这个方法接收两个参数，第一个参数表示要存储的view的数量，第二个参数表示ListView中第一个可见元素的position值。RecycleBin当中使用mActiveViews这个数组来存储View，调用这个方法后就会根据传入的参数来将ListView中的指定元素存储到mActiveViews数组当中。
* getActiveView() 这个方法和fillActiveViews()是对应的，用于从mActiveViews数组当中获取数据。该方法接收一个position参数，表示元素在ListView当中的位置，方法内部会自动将position值转换成mActiveViews数组对应的下标值。需要注意的是，mActiveViews当中所存储的View，一旦被获取了之后就会从mActiveViews当中移除，下次获取同样位置的View将会返回null，也就是说mActiveViews不能被重复利用。
* addScrapView() 用于将一个废弃的View进行缓存，该方法接收一个View参数，当有某个View确定要废弃掉的时候(比如滚动出了屏幕)，就应该调用这个方法来对View进行缓存，RecycleBin当中使用mScrapViews和mCurrentScrap这两个List来存储废弃View。
* getScrapView 用于从废弃缓存中取出一个View，这些废弃缓存中的View是没有顺序可言的，因此getScrapView()方法中的算法也非常简单，就是直接从mCurrentScrap当中获取尾部的一个scrap view进行返回。
* setViewTypeCount() 我们都知道Adapter当中可以重写一个getViewTypeCount()来表示ListView中有几种类型的数据项，而setViewTypeCount()方法的作用就是为每种类型的数据项都单独启用一个RecycleBin缓存机制。实际上，getViewTypeCount()方法通常情况下使用的并不是很多，所以我们只要知道RecycleBin当中有这样一个功能就行了。


了解了RecycleBin中的主要方法以及它们的用处之后，下面就可以开始来分析ListView的工作原理了，这里我将还是按照以前分析源码的方式来进行，即跟着主线执行流程来逐步阅读并点到即止，不然的话要是把ListView所有的代码都贴出来，那么本篇文章将会很长很长了。

# 第一次Layout
---
不管怎么说，ListView即使再特殊最终还是继承自View的，因此它的执行流程还将会按照View的规则来执行。

View的执行流程无非就分为三步，onMeasure()用于测量View的大小，onLayout()用于确定View的布局，onDraw()用于将View绘制到界面上。而在ListView当中，onMeasure()并没有什么特殊的地方，因为它终归是一个View，占用的空间最多并且通常也就是整个屏幕。onDraw()在ListView当中也没有什么意义，因为ListView本身并不负责绘制，而是由ListView当中的子元素来进行绘制的。那么ListView大部分的神奇功能其实都是在onLayout()方法中进行的了，因此我们本篇文章也是主要分析的这个方法里的内容。

如果你到ListView源码中去找一找，你会发现ListView中是没有onLayout()这个方法的，这是因为这个方法是在ListView的父类AbsListView中实现的，代码如下所示：
```java
/**
 * Subclasses should NOT override this method but {@link #layoutChildren()}
 * instead.
 */
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
	super.onLayout(changed, l, t, r, b);
	mInLayout = true;
	if (changed) {
		int childCount = getChildCount();
		for (int i = 0; i < childCount; i++) {
			getChildAt(i).forceLayout();
		}
		mRecycler.markChildrenDirty();
	}
	layoutChildren();
	mInLayout = false;
}
```
可以看到，onLayout()方法中并没有做什么复杂的逻辑操作，主要就是一个判断，如果ListView的大小或者位置发生了变化，那么changed变量就会变成true，此时会要求所有的子布局都强制进行重绘。除此之外倒没有什么难理解的地方了，不过我们注意到，在第16行调用了layoutChildren()这个方法，从方法名上我们就可以猜出这个方法是用来进行子元素布局的，不过进入到这个方法当中你会发现这是个空方法，没有一行代码。这当然是可以理解的了，因为子元素的布局应该是由具体的实现类来负责完成的，而不是由父类完成。那么进入ListView的layoutChildren()方法，代码如下所示：
```java
@Override
protected void layoutChildren() {
    final boolean blockLayoutRequests = mBlockLayoutRequests;
    if (!blockLayoutRequests) {
        mBlockLayoutRequests = true;
    } else {
        return;
    }
    try {
        super.layoutChildren();
        invalidate();
        if (mAdapter == null) {
            resetList();
            invokeOnItemScrollListener();
            return;
        }
        int childrenTop = mListPadding.top;
        int childrenBottom = getBottom() - getTop() - mListPadding.bottom;
        int childCount = getChildCount();
        int index = 0;
        int delta = 0;
        View sel;
        View oldSel = null;
        View oldFirst = null;
        View newSel = null;
        View focusLayoutRestoreView = null;
        // Remember stuff we will need down below
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            index = mNextSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                newSel = getChildAt(index);
            }
            break;
        case LAYOUT_FORCE_TOP:
        case LAYOUT_FORCE_BOTTOM:
        case LAYOUT_SPECIFIC:
        case LAYOUT_SYNC:
            break;
        case LAYOUT_MOVE_SELECTION:
        default:
            // Remember the previously selected view
            index = mSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                oldSel = getChildAt(index);
            }
            // Remember the previous first child
            oldFirst = getChildAt(0);
            if (mNextSelectedPosition >= 0) {
                delta = mNextSelectedPosition - mSelectedPosition;
            }
            // Caution: newSel might be null
            newSel = getChildAt(index + delta);
        }
        boolean dataChanged = mDataChanged;
        if (dataChanged) {
            handleDataChanged();
        }
        // Handle the empty set by removing all views that are visible
        // and calling it a day
        if (mItemCount == 0) {
            resetList();
            invokeOnItemScrollListener();
            return;
        } else if (mItemCount != mAdapter.getCount()) {
            throw new IllegalStateException("The content of the adapter has changed but "
                    + "ListView did not receive a notification. Make sure the content of "
                    + "your adapter is not modified from a background thread, but only "
                    + "from the UI thread. [in ListView(" + getId() + ", " + getClass() 
                    + ") with Adapter(" + mAdapter.getClass() + ")]");
        }
        setSelectedPositionInt(mNextSelectedPosition);
        // Pull all children into the RecycleBin.
        // These views will be reused if possible
        final int firstPosition = mFirstPosition;
        final RecycleBin recycleBin = mRecycler;
        // reset the focus restoration
        View focusLayoutRestoreDirectChild = null;
        // Don't put header or footer views into the Recycler. Those are
        // already cached in mHeaderViews;
        if (dataChanged) {
            for (int i = 0; i < childCount; i++) {
                recycleBin.addScrapView(getChildAt(i));
                if (ViewDebug.TRACE_RECYCLER) {
                    ViewDebug.trace(getChildAt(i),
                            ViewDebug.RecyclerTraceType.MOVE_TO_SCRAP_HEAP, index, i);
                }
            }
        } else {
            recycleBin.fillActiveViews(childCount, firstPosition);
        }
        // take focus back to us temporarily to avoid the eventual
        // call to clear focus when removing the focused child below
        // from messing things up when ViewRoot assigns focus back
        // to someone else
        final View focusedChild = getFocusedChild();
        if (focusedChild != null) {
            // TODO: in some cases focusedChild.getParent() == null
            // we can remember the focused view to restore after relayout if the
            // data hasn't changed, or if the focused position is a header or footer
            if (!dataChanged || isDirectChildHeaderOrFooter(focusedChild)) {
                focusLayoutRestoreDirectChild = focusedChild;
                // remember the specific view that had focus
                focusLayoutRestoreView = findFocus();
                if (focusLayoutRestoreView != null) {
                    // tell it we are going to mess with it
                    focusLayoutRestoreView.onStartTemporaryDetach();
                }
            }
            requestFocus();
        }
        // Clear out old views
        detachAllViewsFromParent();
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            if (newSel != null) {
                sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
            } else {
                sel = fillFromMiddle(childrenTop, childrenBottom);
            }
            break;
        case LAYOUT_SYNC:
            sel = fillSpecific(mSyncPosition, mSpecificTop);
            break;
        case LAYOUT_FORCE_BOTTOM:
            sel = fillUp(mItemCount - 1, childrenBottom);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_FORCE_TOP:
            mFirstPosition = 0;
            sel = fillFromTop(childrenTop);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_SPECIFIC:
            sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
            break;
        case LAYOUT_MOVE_SELECTION:
            sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
            break;
        default:
            if (childCount == 0) {
                if (!mStackFromBottom) {
                    final int position = lookForSelectablePosition(0, true);
                    setSelectedPositionInt(position);
                    sel = fillFromTop(childrenTop);
                } else {
                    final int position = lookForSelectablePosition(mItemCount - 1, false);
                    setSelectedPositionInt(position);
                    sel = fillUp(mItemCount - 1, childrenBottom);
                }
            } else {
                if (mSelectedPosition >= 0 && mSelectedPosition < mItemCount) {
                    sel = fillSpecific(mSelectedPosition,
                            oldSel == null ? childrenTop : oldSel.getTop());
                } else if (mFirstPosition < mItemCount) {
                    sel = fillSpecific(mFirstPosition,
                            oldFirst == null ? childrenTop : oldFirst.getTop());
                } else {
                    sel = fillSpecific(0, childrenTop);
                }
            }
            break;
        }
        // Flush any cached views that did not get reused above
        recycleBin.scrapActiveViews();
        if (sel != null) {
            // the current selected item should get focus if items
            // are focusable
            if (mItemsCanFocus && hasFocus() && !sel.hasFocus()) {
                final boolean focusWasTaken = (sel == focusLayoutRestoreDirectChild &&
                        focusLayoutRestoreView.requestFocus()) || sel.requestFocus();
                if (!focusWasTaken) {
                    // selected item didn't take focus, fine, but still want
                    // to make sure something else outside of the selected view
                    // has focus
                    final View focused = getFocusedChild();
                    if (focused != null) {
                        focused.clearFocus();
                    }
                    positionSelector(sel);
                } else {
                    sel.setSelected(false);
                    mSelectorRect.setEmpty();
                }
            } else {
                positionSelector(sel);
            }
            mSelectedTop = sel.getTop();
        } else {
            if (mTouchMode > TOUCH_MODE_DOWN && mTouchMode < TOUCH_MODE_SCROLL) {
                View child = getChildAt(mMotionPosition - mFirstPosition);
                if (child != null) positionSelector(child);
            } else {
                mSelectedTop = 0;
                mSelectorRect.setEmpty();
            }
            // even if there is not selected position, we may need to restore
            // focus (i.e. something focusable in touch mode)
            if (hasFocus() && focusLayoutRestoreView != null) {
                focusLayoutRestoreView.requestFocus();
            }
        }
        // tell focus view we are done mucking with it, if it is still in
        // our view hierarchy.
        if (focusLayoutRestoreView != null
                && focusLayoutRestoreView.getWindowToken() != null) {
            focusLayoutRestoreView.onFinishTemporaryDetach();
        }
        mLayoutMode = LAYOUT_NORMAL;
        mDataChanged = false;
        mNeedSync = false;
        setNextSelectedPositionInt(mSelectedPosition);
        updateScrollIndicators();
        if (mItemCount > 0) {
            checkSelectionChanged();
        }
        invokeOnItemScrollListener();
    } finally {
        if (!blockLayoutRequests) {
            mBlockLayoutRequests = false;
        }
    }
}
```
这段代码比较长，我们挑重点的看。首先可以确定的是，ListView当中目前还没有任何子View，数据都还是由Adapter管理的，并没有展示到界面上，因此第19行getChildCount()方法得到的值肯定是0。接着在第81行会根据dataChanged这个布尔型的值来判断执行逻辑，dataChanged只有在数据源发生改变的情况下才会变成true，其它情况都是false，因此这里会进入到第90行的执行逻辑，调用RecycleBin的fillActiveViews()方法。按理来说，调用fillActiveViews()方法是为了将ListView的子View进行缓存的，可是目前ListView中还没有任何的子View，因此这一行暂时还起不了任何作用。

接下来在第114行会根据mLayoutMode的值来决定布局模式，默认情况下都是普通模式LAYOUT_NORMAL，因此会进入到第140行的default语句当中。而下面又会紧接着进行两次if判断，childCount目前是等于0的，并且默认的布局顺序是从上往下，因此会进入到第145行的fillFromTop()方法，我们跟进去瞧一瞧：
```java
/**
 * Fills the list from top to bottom, starting with mFirstPosition
 *
 * @param nextTop The location where the top of the first item should be
 *        drawn
 *
 * @return The view that is currently selected
 */
private View fillFromTop(int nextTop) {
    mFirstPosition = Math.min(mFirstPosition, mSelectedPosition);
    mFirstPosition = Math.min(mFirstPosition, mItemCount - 1);
    if (mFirstPosition < 0) {
        mFirstPosition = 0;
    }
    return fillDown(mFirstPosition, nextTop);
}
```
从这个方法的注释中可以看出，它所负责的主要任务就是从mFirstPosition开始，自顶至底去填充ListView。而这个方法本身并没有什么逻辑，就是判断了一下mFirstPosition值的合法性，然后调用fillDown()方法，那么我们就有理由可以猜测，填充ListView的操作是在fillDown()方法中完成的。进入fillDown()方法，代码如下所示：
```java
/**
 * Fills the list from pos down to the end of the list view.
 *
 * @param pos The first position to put in the list
 *
 * @param nextTop The location where the top of the item associated with pos
 *        should be drawn
 *
 * @return The view that is currently selected, if it happens to be in the
 *         range that we draw.
 */
private View fillDown(int pos, int nextTop) {
    View selectedView = null;
    int end = (getBottom() - getTop()) - mListPadding.bottom;
    while (nextTop < end && pos < mItemCount) {
        // is this the selected item?
        boolean selected = pos == mSelectedPosition;
        View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);
        nextTop = child.getBottom() + mDividerHeight;
        if (selected) {
            selectedView = child;
        }
        pos++;
    }
    return selectedView;
}
```
可以看到，这里使用了一个while循环来执行重复逻辑，一开始nextTop的值是第一个子元素顶部距离整个ListView顶部的像素值，pos则是刚刚传入的mFirstPosition的值，而end是ListView底部减去顶部所得的像素值，mItemCount则是Adapter中的元素数量。因此一开始的情况下nextTop必定是小于end值的，并且pos也是小于mItemCount值的。那么每执行一次while循环，pos的值都会加1，并且nextTop也会增加，当nextTop大于等于end时，也就是子元素已经超出当前屏幕了，或者pos大于等于mItemCount时，也就是所有Adapter中的元素都被遍历结束了，就会跳出while循环。

那么while循环当中又做了什么事情呢？值得让人留意的就是第18行调用的makeAndAddView()方法，进入到这个方法当中，代码如下所示：
```java
/**
 * Obtain the view and add it to our list of children. The view can be made
 * fresh, converted from an unused view, or used as is if it was in the
 * recycle bin.
 *
 * @param position Logical position in the list
 * @param y Top or bottom edge of the view to add
 * @param flow If flow is true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @return View that was added
 */
private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
        boolean selected) {
    View child;
    if (!mDataChanged) {
        // Try to use an exsiting view for this position
        child = mRecycler.getActiveView(position);
        if (child != null) {
            // Found it -- we're using an existing child
            // This just needs to be positioned
            setupChild(child, position, y, flow, childrenLeft, selected, true);
            return child;
        }
    }
    // Make a new view for this position, or convert an unused view if possible
    child = obtainView(position, mIsScrap);
    // This needs to be positioned and measured
    setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);
    return child;
}
```
这里在第19行尝试从RecycleBin当中快速获取一个active view，不过很遗憾的是目前RecycleBin当中还没有缓存任何的View，所以这里得到的值肯定是null。那么取得了null之后就会继续向下运行，到第28行会调用obtainView()方法来再次尝试获取一个View，这次的obtainView()方法是可以保证一定返回一个View的，于是下面立刻将获取到的View传入到了setupChild()方法当中。那么obtainView()内部到底是怎么工作的呢？我们先进入到这个方法里面看一下：
```java
/**
 * Get a view and have it show the data associated with the specified
 * position. This is called when we have already discovered that the view is
 * not available for reuse in the recycle bin. The only choices left are
 * converting an old view or making a new one.
 * 
 * @param position
 *            The position to display
 * @param isScrap
 *            Array of at least 1 boolean, the first entry will become true
 *            if the returned view was taken from the scrap heap, false if
 *            otherwise.
 * 
 * @return A view displaying the data associated with the specified position
 */
View obtainView(int position, boolean[] isScrap) {
	isScrap[0] = false;
	View scrapView;
	scrapView = mRecycler.getScrapView(position);
	View child;
	if (scrapView != null) {
		child = mAdapter.getView(position, scrapView, this);
		if (child != scrapView) {
			mRecycler.addScrapView(scrapView);
			if (mCacheColorHint != 0) {
				child.setDrawingCacheBackgroundColor(mCacheColorHint);
			}
		} else {
			isScrap[0] = true;
			dispatchFinishTemporaryDetach(child);
		}
	} else {
		child = mAdapter.getView(position, null, this);
		if (mCacheColorHint != 0) {
			child.setDrawingCacheBackgroundColor(mCacheColorHint);
		}
	}
	return child;
}
```
obtainView()方法中的代码并不多，但却包含了非常非常重要的逻辑，不夸张的说，整个ListView中最重要的内容可能就在这个方法里了。那么我们还是按照执行流程来看，在第19行代码中调用了RecycleBin的getScrapView()方法来尝试获取一个废弃缓存中的View，同样的道理，这里肯定是获取不到的，getScrapView()方法会返回一个null。这时该怎么办呢？没有关系，代码会执行到第33行，调用mAdapter的getView()方法来去获取一个View。那么mAdapter是什么呢？当然就是当前ListView关联的适配器了。而getView()方法又是什么呢？还用说吗，这个就是我们平时使用ListView时最最经常重写的一个方法了，这里getView()方法中传入了三个参数，分别是position，null和this。

那么我们平时写ListView的Adapter时，getView()方法通常会怎么写呢？这里我举个简单的例子：
```java
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	Fruit fruit = getItem(position);
	View view;
	if (convertView == null) {
		view = LayoutInflater.from(getContext()).inflate(resourceId, null);
	} else {
		view = convertView;
	}
	ImageView fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
	TextView fruitName = (TextView) view.findViewById(R.id.fruit_name);
	fruitImage.setImageResource(fruit.getImageId());
	fruitName.setText(fruit.getName());
	return view;
}
```
getView()方法接受的三个参数，第一个参数position代表当前子元素的的位置，我们可以通过具体的位置来获取与其相关的数据。第二个参数convertView，刚才传入的是null，说明没有convertView可以利用，因此我们会调用LayoutInflater的inflate()方法来去加载一个布局。接下来会对这个view进行一些属性和值的设定，最后将view返回。

那么这个View也会作为obtainView()的结果进行返回，并最终传入到setupChild()方法当中。其实也就是说，第一次layout过程当中，所有的子View都是调用LayoutInflater的inflate()方法加载出来的，这样就会相对比较耗时，但是不用担心，后面就不会再有这种情况了，那么我们继续往下看：
```java

/**
 * Add a view as a child and make sure it is measured (if necessary) and
 * positioned properly.
 *
 * @param child The view to add
 * @param position The position of this child
 * @param y The y position relative to which this view will be positioned
 * @param flowDown If true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @param recycled Has this view been pulled from the recycle bin? If so it
 *        does not need to be remeasured.
 */
private void setupChild(View child, int position, int y, boolean flowDown, int childrenLeft,
        boolean selected, boolean recycled) {
    final boolean isSelected = selected && shouldShowSelector();
    final boolean updateChildSelected = isSelected != child.isSelected();
    final int mode = mTouchMode;
    final boolean isPressed = mode > TOUCH_MODE_DOWN && mode < TOUCH_MODE_SCROLL &&
            mMotionPosition == position;
    final boolean updateChildPressed = isPressed != child.isPressed();
    final boolean needToMeasure = !recycled || updateChildSelected || child.isLayoutRequested();
    // Respect layout params that are already in the view. Otherwise make some up...
    // noinspection unchecked
    AbsListView.LayoutParams p = (AbsListView.LayoutParams) child.getLayoutParams();
    if (p == null) {
        p = new AbsListView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT, 0);
    }
    p.viewType = mAdapter.getItemViewType(position);
    if ((recycled && !p.forceAdd) || (p.recycledHeaderFooter &&
            p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER)) {
        attachViewToParent(child, flowDown ? -1 : 0, p);
    } else {
        p.forceAdd = false;
        if (p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
            p.recycledHeaderFooter = true;
        }
        addViewInLayout(child, flowDown ? -1 : 0, p, true);
    }
    if (updateChildSelected) {
        child.setSelected(isSelected);
    }
    if (updateChildPressed) {
        child.setPressed(isPressed);
    }
    if (needToMeasure) {
        int childWidthSpec = ViewGroup.getChildMeasureSpec(mWidthMeasureSpec,
                mListPadding.left + mListPadding.right, p.width);
        int lpHeight = p.height;
        int childHeightSpec;
        if (lpHeight > 0) {
            childHeightSpec = MeasureSpec.makeMeasureSpec(lpHeight, MeasureSpec.EXACTLY);
        } else {
            childHeightSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
        }
        child.measure(childWidthSpec, childHeightSpec);
    } else {
        cleanupLayoutState(child);
    }
    final int w = child.getMeasuredWidth();
    final int h = child.getMeasuredHeight();
    final int childTop = flowDown ? y : y - h;
    if (needToMeasure) {
        final int childRight = childrenLeft + w;
        final int childBottom = childTop + h;
        child.layout(childrenLeft, childTop, childRight, childBottom);
    } else {
        child.offsetLeftAndRight(childrenLeft - child.getLeft());
        child.offsetTopAndBottom(childTop - child.getTop());
    }
    if (mCachingStarted && !child.isDrawingCacheEnabled()) {
        child.setDrawingCacheEnabled(true);
    }
}
```
setupChild()方法当中的代码虽然比较多，但是我们只看核心代码的话就非常简单了，刚才调用obtainView()方法获取到的子元素View，这里在第40行调用了addViewInLayout()方法将它添加到了ListView当中。那么根据fillDown()方法中的while循环，会让子元素View将整个ListView控件填满然后就跳出，也就是说即使我们的Adapter中有一千条数据，ListView也只会加载第一屏的数据，剩下的数据反正目前在屏幕上也看不到，所以不会去做多余的加载工作，这样就可以保证ListView中的内容能够迅速展示到屏幕上。

那么到此为止，第一次Layout过程结束。

# 第二次Layout
---
虽然我在源码中并没有找出具体的原因，但如果你自己做一下实验的话就会发现，即使是一个再简单的View，在展示到界面上之前都会经历至少两次onMeasure()和两次onLayout()的过程。其实这只是一个很小的细节，平时对我们影响并不大，因为不管是onMeasure()或者onLayout()几次，反正都是执行的相同的逻辑，我们并不需要进行过多关心。但是在ListView中情况就不一样了，因为这就意味着layoutChildren()过程会执行两次，而这个过程当中涉及到向ListView中添加子元素，如果相同的逻辑执行两遍的话，那么ListView中就会存在一份重复的数据了。因此ListView在layoutChildren()过程当中做了第二次Layout的逻辑处理，非常巧妙地解决了这个问题，下面我们就来分析一下第二次Layout的过程。

其实第二次Layout和第一次Layout的基本流程是差不多的，那么我们还是从layoutChildren()方法开始看起：
```java
@Override
protected void layoutChildren() {
    final boolean blockLayoutRequests = mBlockLayoutRequests;
    if (!blockLayoutRequests) {
        mBlockLayoutRequests = true;
    } else {
        return;
    }
    try {
        super.layoutChildren();
        invalidate();
        if (mAdapter == null) {
            resetList();
            invokeOnItemScrollListener();
            return;
        }
        int childrenTop = mListPadding.top;
        int childrenBottom = getBottom() - getTop() - mListPadding.bottom;
        int childCount = getChildCount();
        int index = 0;
        int delta = 0;
        View sel;
        View oldSel = null;
        View oldFirst = null;
        View newSel = null;
        View focusLayoutRestoreView = null;
        // Remember stuff we will need down below
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            index = mNextSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                newSel = getChildAt(index);
            }
            break;
        case LAYOUT_FORCE_TOP:
        case LAYOUT_FORCE_BOTTOM:
        case LAYOUT_SPECIFIC:
        case LAYOUT_SYNC:
            break;
        case LAYOUT_MOVE_SELECTION:
        default:
            // Remember the previously selected view
            index = mSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                oldSel = getChildAt(index);
            }
            // Remember the previous first child
            oldFirst = getChildAt(0);
            if (mNextSelectedPosition >= 0) {
                delta = mNextSelectedPosition - mSelectedPosition;
            }
            // Caution: newSel might be null
            newSel = getChildAt(index + delta);
        }
        boolean dataChanged = mDataChanged;
        if (dataChanged) {
            handleDataChanged();
        }
        // Handle the empty set by removing all views that are visible
        // and calling it a day
        if (mItemCount == 0) {
            resetList();
            invokeOnItemScrollListener();
            return;
        } else if (mItemCount != mAdapter.getCount()) {
            throw new IllegalStateException("The content of the adapter has changed but "
                    + "ListView did not receive a notification. Make sure the content of "
                    + "your adapter is not modified from a background thread, but only "
                    + "from the UI thread. [in ListView(" + getId() + ", " + getClass() 
                    + ") with Adapter(" + mAdapter.getClass() + ")]");
        }
        setSelectedPositionInt(mNextSelectedPosition);
        // Pull all children into the RecycleBin.
        // These views will be reused if possible
        final int firstPosition = mFirstPosition;
        final RecycleBin recycleBin = mRecycler;
        // reset the focus restoration
        View focusLayoutRestoreDirectChild = null;
        // Don't put header or footer views into the Recycler. Those are
        // already cached in mHeaderViews;
        if (dataChanged) {
            for (int i = 0; i < childCount; i++) {
                recycleBin.addScrapView(getChildAt(i));
                if (ViewDebug.TRACE_RECYCLER) {
                    ViewDebug.trace(getChildAt(i),
                            ViewDebug.RecyclerTraceType.MOVE_TO_SCRAP_HEAP, index, i);
                }
            }
        } else {
            recycleBin.fillActiveViews(childCount, firstPosition);
        }
        // take focus back to us temporarily to avoid the eventual
        // call to clear focus when removing the focused child below
        // from messing things up when ViewRoot assigns focus back
        // to someone else
        final View focusedChild = getFocusedChild();
        if (focusedChild != null) {
            // TODO: in some cases focusedChild.getParent() == null
            // we can remember the focused view to restore after relayout if the
            // data hasn't changed, or if the focused position is a header or footer
            if (!dataChanged || isDirectChildHeaderOrFooter(focusedChild)) {
                focusLayoutRestoreDirectChild = focusedChild;
                // remember the specific view that had focus
                focusLayoutRestoreView = findFocus();
                if (focusLayoutRestoreView != null) {
                    // tell it we are going to mess with it
                    focusLayoutRestoreView.onStartTemporaryDetach();
                }
            }
            requestFocus();
        }
        // Clear out old views
        detachAllViewsFromParent();
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            if (newSel != null) {
                sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
            } else {
                sel = fillFromMiddle(childrenTop, childrenBottom);
            }
            break;
        case LAYOUT_SYNC:
            sel = fillSpecific(mSyncPosition, mSpecificTop);
            break;
        case LAYOUT_FORCE_BOTTOM:
            sel = fillUp(mItemCount - 1, childrenBottom);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_FORCE_TOP:
            mFirstPosition = 0;
            sel = fillFromTop(childrenTop);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_SPECIFIC:
            sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
            break;
        case LAYOUT_MOVE_SELECTION:
            sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
            break;
        default:
            if (childCount == 0) {
                if (!mStackFromBottom) {
                    final int position = lookForSelectablePosition(0, true);
                    setSelectedPositionInt(position);
                    sel = fillFromTop(childrenTop);
                } else {
                    final int position = lookForSelectablePosition(mItemCount - 1, false);
                    setSelectedPositionInt(position);
                    sel = fillUp(mItemCount - 1, childrenBottom);
                }
            } else {
                if (mSelectedPosition >= 0 && mSelectedPosition < mItemCount) {
                    sel = fillSpecific(mSelectedPosition,
                            oldSel == null ? childrenTop : oldSel.getTop());
                } else if (mFirstPosition < mItemCount) {
                    sel = fillSpecific(mFirstPosition,
                            oldFirst == null ? childrenTop : oldFirst.getTop());
                } else {
                    sel = fillSpecific(0, childrenTop);
                }
            }
            break;
        }
        // Flush any cached views that did not get reused above
        recycleBin.scrapActiveViews();
        if (sel != null) {
            // the current selected item should get focus if items
            // are focusable
            if (mItemsCanFocus && hasFocus() && !sel.hasFocus()) {
                final boolean focusWasTaken = (sel == focusLayoutRestoreDirectChild &&
                        focusLayoutRestoreView.requestFocus()) || sel.requestFocus();
                if (!focusWasTaken) {
                    // selected item didn't take focus, fine, but still want
                    // to make sure something else outside of the selected view
                    // has focus
                    final View focused = getFocusedChild();
                    if (focused != null) {
                        focused.clearFocus();
                    }
                    positionSelector(sel);
                } else {
                    sel.setSelected(false);
                    mSelectorRect.setEmpty();
                }
            } else {
                positionSelector(sel);
            }
            mSelectedTop = sel.getTop();
        } else {
            if (mTouchMode > TOUCH_MODE_DOWN && mTouchMode < TOUCH_MODE_SCROLL) {
                View child = getChildAt(mMotionPosition - mFirstPosition);
                if (child != null) positionSelector(child);
            } else {
                mSelectedTop = 0;
                mSelectorRect.setEmpty();
            }
            // even if there is not selected position, we may need to restore
            // focus (i.e. something focusable in touch mode)
            if (hasFocus() && focusLayoutRestoreView != null) {
                focusLayoutRestoreView.requestFocus();
            }
        }
        // tell focus view we are done mucking with it, if it is still in
        // our view hierarchy.
        if (focusLayoutRestoreView != null
                && focusLayoutRestoreView.getWindowToken() != null) {
            focusLayoutRestoreView.onFinishTemporaryDetach();
        }
        mLayoutMode = LAYOUT_NORMAL;
        mDataChanged = false;
        mNeedSync = false;
        setNextSelectedPositionInt(mSelectedPosition);
        updateScrollIndicators();
        if (mItemCount > 0) {
            checkSelectionChanged();
        }
        invokeOnItemScrollListener();
    } finally {
        if (!blockLayoutRequests) {
            mBlockLayoutRequests = false;
        }
    }
}
```
同样还是在第19行，调用getChildCount()方法来获取子View的数量，只不过现在得到的值不会再是0了，而是ListView中一屏可以显示的子View数量，因为我们刚刚在第一次Layout过程当中向ListView添加了这么多的子View。下面在第90行调用了RecycleBin的fillActiveViews()方法，这次效果可就不一样了，因为目前ListView中已经有子View了，这样所有的子View都会被缓存到RecycleBin的mActiveViews数组当中，后面将会用到它们。

接下来将会是非常非常重要的一个操作，在第113行调用了detachAllViewsFromParent()方法。这个方法会将所有ListView当中的子View全部清除掉，从而保证第二次Layout过程不会产生一份重复的数据。那有的朋友可能会问了，这样把已经加载好的View又清除掉，待会还要再重新加载一遍，这不是严重影响效率吗？不用担心，还记得我们刚刚调用了RecycleBin的fillActiveViews()方法来缓存子View吗，待会儿将会直接使用这些缓存好的View来进行加载，而并不会重新执行一遍inflate过程，因此效率方面并不会有什么明显的影响。

那么我们接着看，在第141行的判断逻辑当中，由于不再等于0了，因此会进入到else语句当中。而else语句中又有三个逻辑判断，第一个逻辑判断不成立，因为默认情况下我们没有选中任何子元素，mSelectedPosition应该等于-1。第二个逻辑判断通常是成立的，因为mFirstPosition的值一开始是等于0的，只要adapter中的数据大于0条件就成立。那么进入到fillSpecific()方法当中，代码如下所示：
```java
/**
 * Put a specific item at a specific location on the screen and then build
 * up and down from there.
 *
 * @param position The reference view to use as the starting point
 * @param top Pixel offset from the top of this view to the top of the
 *        reference view.
 *
 * @return The selected view, or null if the selected view is outside the
 *         visible area.
 */
private View fillSpecific(int position, int top) {
    boolean tempIsSelected = position == mSelectedPosition;
    View temp = makeAndAddView(position, top, true, mListPadding.left, tempIsSelected);
    // Possibly changed again in fillUp if we add rows above this one.
    mFirstPosition = position;
    View above;
    View below;
    final int dividerHeight = mDividerHeight;
    if (!mStackFromBottom) {
        above = fillUp(position - 1, temp.getTop() - dividerHeight);
        // This will correct for the top of the first view not touching the top of the list
        adjustViewsUpOrDown();
        below = fillDown(position + 1, temp.getBottom() + dividerHeight);
        int childCount = getChildCount();
        if (childCount > 0) {
            correctTooHigh(childCount);
        }
    } else {
        below = fillDown(position + 1, temp.getBottom() + dividerHeight);
        // This will correct for the bottom of the last view not touching the bottom of the list
        adjustViewsUpOrDown();
        above = fillUp(position - 1, temp.getTop() - dividerHeight);
        int childCount = getChildCount();
        if (childCount > 0) {
             correctTooLow(childCount);
        }
    }
    if (tempIsSelected) {
        return temp;
    } else if (above != null) {
        return above;
    } else {
        return below;
    }
}
```
fillSpecific()这算是一个新方法了，不过其实它和fillUp()、fillDown()方法功能也是差不多的，主要的区别在于，fillSpecific()方法会优先将指定位置的子View先加载到屏幕上，然后再加载该子View往上以及往下的其它子View。那么由于这里我们传入的position就是第一个子View的位置，于是fillSpecific()方法的作用就基本上和fillDown()方法是差不多的了，这里我们就不去关注太多它的细节，而是将精力放在makeAndAddView()方法上面。再次回到makeAndAddView()方法，代码如下所示：
```java
/**
 * Obtain the view and add it to our list of children. The view can be made
 * fresh, converted from an unused view, or used as is if it was in the
 * recycle bin.
 *
 * @param position Logical position in the list
 * @param y Top or bottom edge of the view to add
 * @param flow If flow is true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @return View that was added
 */
private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
        boolean selected) {
    View child;
    if (!mDataChanged) {
        // Try to use an exsiting view for this position
        child = mRecycler.getActiveView(position);
        if (child != null) {
            // Found it -- we're using an existing child
            // This just needs to be positioned
            setupChild(child, position, y, flow, childrenLeft, selected, true);
            return child;
        }
    }
    // Make a new view for this position, or convert an unused view if possible
    child = obtainView(position, mIsScrap);
    // This needs to be positioned and measured
    setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);
    return child;
}
```
仍然还是在第19行尝试从RecycleBin当中获取Active View，然而这次就一定可以获取到了，因为前面我们调用了RecycleBin的fillActiveViews()方法来缓存子View。那么既然如此，就不会再进入到第28行的obtainView()方法，而是会直接进入setupChild()方法当中，这样也省去了很多时间，因为如果在obtainView()方法中又要去infalte布局的话，那么ListView的初始加载效率就大大降低了。

注意在第23行，setupChild()方法的最后一个参数传入的是true，这个参数表明当前的View是之前被回收过的，那么我们再次回到setupChild()方法当中：
```java

/**
 * Add a view as a child and make sure it is measured (if necessary) and
 * positioned properly.
 *
 * @param child The view to add
 * @param position The position of this child
 * @param y The y position relative to which this view will be positioned
 * @param flowDown If true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @param recycled Has this view been pulled from the recycle bin? If so it
 *        does not need to be remeasured.
 */
private void setupChild(View child, int position, int y, boolean flowDown, int childrenLeft,
        boolean selected, boolean recycled) {
    final boolean isSelected = selected && shouldShowSelector();
    final boolean updateChildSelected = isSelected != child.isSelected();
    final int mode = mTouchMode;
    final boolean isPressed = mode > TOUCH_MODE_DOWN && mode < TOUCH_MODE_SCROLL &&
            mMotionPosition == position;
    final boolean updateChildPressed = isPressed != child.isPressed();
    final boolean needToMeasure = !recycled || updateChildSelected || child.isLayoutRequested();
    // Respect layout params that are already in the view. Otherwise make some up...
    // noinspection unchecked
    AbsListView.LayoutParams p = (AbsListView.LayoutParams) child.getLayoutParams();
    if (p == null) {
        p = new AbsListView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT, 0);
    }
    p.viewType = mAdapter.getItemViewType(position);
    if ((recycled && !p.forceAdd) || (p.recycledHeaderFooter &&
            p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER)) {
        attachViewToParent(child, flowDown ? -1 : 0, p);
    } else {
        p.forceAdd = false;
        if (p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
            p.recycledHeaderFooter = true;
        }
        addViewInLayout(child, flowDown ? -1 : 0, p, true);
    }
    if (updateChildSelected) {
        child.setSelected(isSelected);
    }
    if (updateChildPressed) {
        child.setPressed(isPressed);
    }
    if (needToMeasure) {
        int childWidthSpec = ViewGroup.getChildMeasureSpec(mWidthMeasureSpec,
                mListPadding.left + mListPadding.right, p.width);
        int lpHeight = p.height;
        int childHeightSpec;
        if (lpHeight > 0) {
            childHeightSpec = MeasureSpec.makeMeasureSpec(lpHeight, MeasureSpec.EXACTLY);
        } else {
            childHeightSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
        }
        child.measure(childWidthSpec, childHeightSpec);
    } else {
        cleanupLayoutState(child);
    }
    final int w = child.getMeasuredWidth();
    final int h = child.getMeasuredHeight();
    final int childTop = flowDown ? y : y - h;
    if (needToMeasure) {
        final int childRight = childrenLeft + w;
        final int childBottom = childTop + h;
        child.layout(childrenLeft, childTop, childRight, childBottom);
    } else {
        child.offsetLeftAndRight(childrenLeft - child.getLeft());
        child.offsetTopAndBottom(childTop - child.getTop());
    }
    if (mCachingStarted && !child.isDrawingCacheEnabled()) {
        child.setDrawingCacheEnabled(true);
    }
}
```
可以看到，setupChild()方法的最后一个参数是recycled，然后在第32行会对这个变量进行判断，由于recycled现在是true，所以会执行attachViewToParent()方法，而第一次Layout过程则是执行的else语句中的addViewInLayout()方法。这两个方法最大的区别在于，如果我们需要向ViewGroup中添加一个新的子View，应该调用addViewInLayout()方法，而如果是想要将一个之前detach的View重新attach到ViewGroup上，就应该调用attachViewToParent()方法。那么由于前面在layoutChildren()方法当中调用了detachAllViewsFromParent()方法，这样ListView中所有的子View都是处于detach状态的，所以这里attachViewToParent()方法是正确的选择。

经历了这样一个detach又attach的过程，ListView中所有的子View又都可以正常显示出来了，那么第二次Layout过程结束。

# 滑动加载更多数据
---
经历了两次Layout过程，虽说我们已经可以在ListView中看到内容了，然而关于ListView最神奇的部分我们却还没有接触到，因为目前ListView中只是加载并显示了第一屏的数据而已。比如说我们的Adapter当中有1000条数据，但是第一屏只显示了10条，ListView中也只有10个子View而已，那么剩下的990是怎样工作并显示到界面上的呢？这就要看一下ListView滑动部分的源码了，因为我们是通过手指滑动来显示更多数据的。

由于滑动部分的机制是属于通用型的，即ListView和GridView都会使用同样的机制，因此这部分代码就肯定是写在AbsListView当中的了。那么监听触控事件是在onTouchEvent()方法当中进行的，我们就来看一下AbsListView中的这个方法：
```java
@Override
public boolean onTouchEvent(MotionEvent ev) {
	if (!isEnabled()) {
		// A disabled view that is clickable still consumes the touch
		// events, it just doesn't respond to them.
		return isClickable() || isLongClickable();
	}
	final int action = ev.getAction();
	View v;
	int deltaY;
	if (mVelocityTracker == null) {
		mVelocityTracker = VelocityTracker.obtain();
	}
	mVelocityTracker.addMovement(ev);
	switch (action & MotionEvent.ACTION_MASK) {
	case MotionEvent.ACTION_DOWN: {
		mActivePointerId = ev.getPointerId(0);
		final int x = (int) ev.getX();
		final int y = (int) ev.getY();
		int motionPosition = pointToPosition(x, y);
		if (!mDataChanged) {
			if ((mTouchMode != TOUCH_MODE_FLING) && (motionPosition >= 0)
					&& (getAdapter().isEnabled(motionPosition))) {
				// User clicked on an actual view (and was not stopping a
				// fling). It might be a
				// click or a scroll. Assume it is a click until proven
				// otherwise
				mTouchMode = TOUCH_MODE_DOWN;
				// FIXME Debounce
				if (mPendingCheckForTap == null) {
					mPendingCheckForTap = new CheckForTap();
				}
				postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
			} else {
				if (ev.getEdgeFlags() != 0 && motionPosition < 0) {
					// If we couldn't find a view to click on, but the down
					// event was touching
					// the edge, we will bail out and try again. This allows
					// the edge correcting
					// code in ViewRoot to try to find a nearby view to
					// select
					return false;
				}
 
				if (mTouchMode == TOUCH_MODE_FLING) {
					// Stopped a fling. It is a scroll.
					createScrollingCache();
					mTouchMode = TOUCH_MODE_SCROLL;
					mMotionCorrection = 0;
					motionPosition = findMotionRow(y);
					reportScrollStateChange(OnScrollListener.SCROLL_STATE_TOUCH_SCROLL);
				}
			}
		}
		if (motionPosition >= 0) {
			// Remember where the motion event started
			v = getChildAt(motionPosition - mFirstPosition);
			mMotionViewOriginalTop = v.getTop();
		}
		mMotionX = x;
		mMotionY = y;
		mMotionPosition = motionPosition;
		mLastY = Integer.MIN_VALUE;
		break;
	}
	case MotionEvent.ACTION_MOVE: {
		final int pointerIndex = ev.findPointerIndex(mActivePointerId);
		final int y = (int) ev.getY(pointerIndex);
		deltaY = y - mMotionY;
		switch (mTouchMode) {
		case TOUCH_MODE_DOWN:
		case TOUCH_MODE_TAP:
		case TOUCH_MODE_DONE_WAITING:
			// Check if we have moved far enough that it looks more like a
			// scroll than a tap
			startScrollIfNeeded(deltaY);
			break;
		case TOUCH_MODE_SCROLL:
			if (PROFILE_SCROLLING) {
				if (!mScrollProfilingStarted) {
					Debug.startMethodTracing("AbsListViewScroll");
					mScrollProfilingStarted = true;
				}
			}
			if (y != mLastY) {
				deltaY -= mMotionCorrection;
				int incrementalDeltaY = mLastY != Integer.MIN_VALUE ? y - mLastY : deltaY;
				// No need to do all this work if we're not going to move
				// anyway
				boolean atEdge = false;
				if (incrementalDeltaY != 0) {
					atEdge = trackMotionScroll(deltaY, incrementalDeltaY);
				}
				// Check to see if we have bumped into the scroll limit
				if (atEdge && getChildCount() > 0) {
					// Treat this like we're starting a new scroll from the
					// current
					// position. This will let the user start scrolling back
					// into
					// content immediately rather than needing to scroll
					// back to the
					// point where they hit the limit first.
					int motionPosition = findMotionRow(y);
					if (motionPosition >= 0) {
						final View motionView = getChildAt(motionPosition - mFirstPosition);
						mMotionViewOriginalTop = motionView.getTop();
					}
					mMotionY = y;
					mMotionPosition = motionPosition;
					invalidate();
				}
				mLastY = y;
			}
			break;
		}
		break;
	}
	case MotionEvent.ACTION_UP: {
		switch (mTouchMode) {
		case TOUCH_MODE_DOWN:
		case TOUCH_MODE_TAP:
		case TOUCH_MODE_DONE_WAITING:
			final int motionPosition = mMotionPosition;
			final View child = getChildAt(motionPosition - mFirstPosition);
			if (child != null && !child.hasFocusable()) {
				if (mTouchMode != TOUCH_MODE_DOWN) {
					child.setPressed(false);
				}
				if (mPerformClick == null) {
					mPerformClick = new PerformClick();
				}
				final AbsListView.PerformClick performClick = mPerformClick;
				performClick.mChild = child;
				performClick.mClickMotionPosition = motionPosition;
				performClick.rememberWindowAttachCount();
				mResurrectToPosition = motionPosition;
				if (mTouchMode == TOUCH_MODE_DOWN || mTouchMode == TOUCH_MODE_TAP) {
					final Handler handler = getHandler();
					if (handler != null) {
						handler.removeCallbacks(mTouchMode == TOUCH_MODE_DOWN ? mPendingCheckForTap
								: mPendingCheckForLongPress);
					}
					mLayoutMode = LAYOUT_NORMAL;
					if (!mDataChanged && mAdapter.isEnabled(motionPosition)) {
						mTouchMode = TOUCH_MODE_TAP;
						setSelectedPositionInt(mMotionPosition);
						layoutChildren();
						child.setPressed(true);
						positionSelector(child);
						setPressed(true);
						if (mSelector != null) {
							Drawable d = mSelector.getCurrent();
							if (d != null && d instanceof TransitionDrawable) {
								((TransitionDrawable) d).resetTransition();
							}
						}
						postDelayed(new Runnable() {
							public void run() {
								child.setPressed(false);
								setPressed(false);
								if (!mDataChanged) {
									post(performClick);
								}
								mTouchMode = TOUCH_MODE_REST;
							}
						}, ViewConfiguration.getPressedStateDuration());
					} else {
						mTouchMode = TOUCH_MODE_REST;
					}
					return true;
				} else if (!mDataChanged && mAdapter.isEnabled(motionPosition)) {
					post(performClick);
				}
			}
			mTouchMode = TOUCH_MODE_REST;
			break;
		case TOUCH_MODE_SCROLL:
			final int childCount = getChildCount();
			if (childCount > 0) {
				if (mFirstPosition == 0
						&& getChildAt(0).getTop() >= mListPadding.top
						&& mFirstPosition + childCount < mItemCount
						&& getChildAt(childCount - 1).getBottom() <= getHeight()
								- mListPadding.bottom) {
					mTouchMode = TOUCH_MODE_REST;
					reportScrollStateChange(OnScrollListener.SCROLL_STATE_IDLE);
				} else {
					final VelocityTracker velocityTracker = mVelocityTracker;
					velocityTracker.computeCurrentVelocity(1000, mMaximumVelocity);
					final int initialVelocity = (int) velocityTracker
							.getYVelocity(mActivePointerId);
					if (Math.abs(initialVelocity) > mMinimumVelocity) {
						if (mFlingRunnable == null) {
							mFlingRunnable = new FlingRunnable();
						}
						reportScrollStateChange(OnScrollListener.SCROLL_STATE_FLING);
						mFlingRunnable.start(-initialVelocity);
					} else {
						mTouchMode = TOUCH_MODE_REST;
						reportScrollStateChange(OnScrollListener.SCROLL_STATE_IDLE);
					}
				}
			} else {
				mTouchMode = TOUCH_MODE_REST;
				reportScrollStateChange(OnScrollListener.SCROLL_STATE_IDLE);
			}
			break;
		}
		setPressed(false);
		// Need to redraw since we probably aren't drawing the selector
		// anymore
		invalidate();
		final Handler handler = getHandler();
		if (handler != null) {
			handler.removeCallbacks(mPendingCheckForLongPress);
		}
		if (mVelocityTracker != null) {
			mVelocityTracker.recycle();
			mVelocityTracker = null;
		}
		mActivePointerId = INVALID_POINTER;
		if (PROFILE_SCROLLING) {
			if (mScrollProfilingStarted) {
				Debug.stopMethodTracing();
				mScrollProfilingStarted = false;
			}
		}
		break;
	}
	case MotionEvent.ACTION_CANCEL: {
		mTouchMode = TOUCH_MODE_REST;
		setPressed(false);
		View motionView = this.getChildAt(mMotionPosition - mFirstPosition);
		if (motionView != null) {
			motionView.setPressed(false);
		}
		clearScrollingCache();
		final Handler handler = getHandler();
		if (handler != null) {
			handler.removeCallbacks(mPendingCheckForLongPress);
		}
		if (mVelocityTracker != null) {
			mVelocityTracker.recycle();
			mVelocityTracker = null;
		}
		mActivePointerId = INVALID_POINTER;
		break;
	}
	case MotionEvent.ACTION_POINTER_UP: {
		onSecondaryPointerUp(ev);
		final int x = mMotionX;
		final int y = mMotionY;
		final int motionPosition = pointToPosition(x, y);
		if (motionPosition >= 0) {
			// Remember where the motion event started
			v = getChildAt(motionPosition - mFirstPosition);
			mMotionViewOriginalTop = v.getTop();
			mMotionPosition = motionPosition;
		}
		mLastY = y;
		break;
	}
	}
	return true;
}
```
这个方法中的代码就非常多了，因为它所处理的逻辑也非常多，要监听各种各样的触屏事件。但是我们目前所关心的就只有手指在屏幕上滑动这一个事件而已，对应的是ACTION_MOVE这个动作，那么我们就只看这部分代码就可以了。

可以看到，ACTION_MOVE这个case里面又嵌套了一个switch语句，是根据当前的TouchMode来选择的。那这里我可以直接告诉大家，当手指在屏幕上滑动时，TouchMode是等于TOUCH_MODE_SCROLL这个值的，至于为什么那又要牵扯到另外的好几个方法，这里限于篇幅原因就不再展开讲解了，喜欢寻根究底的朋友们可以自己去源码里找一找原因。

这样的话，代码就应该会走到第78行的这个case里面去了，在这个case当中并没有什么太多需要注意的东西，唯一一点非常重要的就是第92行调用的trackMotionScroll()方法，相当于我们手指只要在屏幕上稍微有一点点移动，这个方法就会被调用，而如果是正常在屏幕上滑动的话，那么这个方法就会被调用很多次。那么我们进入到这个方法中瞧一瞧，代码如下所示：
```java
boolean trackMotionScroll(int deltaY, int incrementalDeltaY) {
	final int childCount = getChildCount();
	if (childCount == 0) {
		return true;
	}
	final int firstTop = getChildAt(0).getTop();
	final int lastBottom = getChildAt(childCount - 1).getBottom();
	final Rect listPadding = mListPadding;
	final int spaceAbove = listPadding.top - firstTop;
	final int end = getHeight() - listPadding.bottom;
	final int spaceBelow = lastBottom - end;
	final int height = getHeight() - getPaddingBottom() - getPaddingTop();
	if (deltaY < 0) {
		deltaY = Math.max(-(height - 1), deltaY);
	} else {
		deltaY = Math.min(height - 1, deltaY);
	}
	if (incrementalDeltaY < 0) {
		incrementalDeltaY = Math.max(-(height - 1), incrementalDeltaY);
	} else {
		incrementalDeltaY = Math.min(height - 1, incrementalDeltaY);
	}
	final int firstPosition = mFirstPosition;
	if (firstPosition == 0 && firstTop >= listPadding.top && deltaY >= 0) {
		// Don't need to move views down if the top of the first position
		// is already visible
		return true;
	}
	if (firstPosition + childCount == mItemCount && lastBottom <= end && deltaY <= 0) {
		// Don't need to move views up if the bottom of the last position
		// is already visible
		return true;
	}
	final boolean down = incrementalDeltaY < 0;
	final boolean inTouchMode = isInTouchMode();
	if (inTouchMode) {
		hideSelector();
	}
	final int headerViewsCount = getHeaderViewsCount();
	final int footerViewsStart = mItemCount - getFooterViewsCount();
	int start = 0;
	int count = 0;
	if (down) {
		final int top = listPadding.top - incrementalDeltaY;
		for (int i = 0; i < childCount; i++) {
			final View child = getChildAt(i);
			if (child.getBottom() >= top) {
				break;
			} else {
				count++;
				int position = firstPosition + i;
				if (position >= headerViewsCount && position < footerViewsStart) {
					mRecycler.addScrapView(child);
				}
			}
		}
	} else {
		final int bottom = getHeight() - listPadding.bottom - incrementalDeltaY;
		for (int i = childCount - 1; i >= 0; i--) {
			final View child = getChildAt(i);
			if (child.getTop() <= bottom) {
				break;
			} else {
				start = i;
				count++;
				int position = firstPosition + i;
				if (position >= headerViewsCount && position < footerViewsStart) {
					mRecycler.addScrapView(child);
				}
			}
		}
	}
	mMotionViewNewTop = mMotionViewOriginalTop + deltaY;
	mBlockLayoutRequests = true;
	if (count > 0) {
		detachViewsFromParent(start, count);
	}
	offsetChildrenTopAndBottom(incrementalDeltaY);
	if (down) {
		mFirstPosition += count;
	}
	invalidate();
	final int absIncrementalDeltaY = Math.abs(incrementalDeltaY);
	if (spaceAbove < absIncrementalDeltaY || spaceBelow < absIncrementalDeltaY) {
		fillGap(down);
	}
	if (!inTouchMode && mSelectedPosition != INVALID_POSITION) {
		final int childIndex = mSelectedPosition - mFirstPosition;
		if (childIndex >= 0 && childIndex < getChildCount()) {
			positionSelector(getChildAt(childIndex));
		}
	}
	mBlockLayoutRequests = false;
	invokeOnItemScrollListener();
	awakenScrollBars();
	return false;
}
```
这个方法接收两个参数，deltaY表示从手指按下时的位置到当前手指位置的距离，incrementalDeltaY则表示据上次触发event事件手指在Y方向上位置的改变量，那么其实我们就可以通过incrementalDeltaY的正负值情况来判断用户是向上还是向下滑动的了。如第34行代码所示，如果incrementalDeltaY小于0，说明是向下滑动，否则就是向上滑动。

下面将会进行一个边界值检测的过程，可以看到，从第43行开始，当ListView向下滑动的时候，就会进入一个for循环当中，从上往下依次获取子View，第47行当中，如果该子View的bottom值已经小于top值了，就说明这个子View已经移出屏幕了，所以会调用RecycleBin的addScrapView()方法将这个View加入到废弃缓存当中，并将count计数器加1，计数器用于记录有多少个子View被移出了屏幕。那么如果是ListView向上滑动的话，其实过程是基本相同的，只不过变成了从下往上依次获取子View，然后判断该子View的top值是不是大于bottom值了，如果大于的话说明子View已经移出了屏幕，同样把它加入到废弃缓存中，并将计数器加1。

接下来在第76行，会根据当前计数器的值来进行一个detach操作，它的作用就是把所有移出屏幕的子View全部detach掉，在ListView的概念当中，所有看不到的View就没有必要为它进行保存，因为屏幕外还有成百上千条数据等着显示呢，一个好的回收策略才能保证ListView的高性能和高效率。紧接着在第78行调用了offsetChildrenTopAndBottom()方法，并将incrementalDeltaY作为参数传入，这个方法的作用是让ListView中所有的子View都按照传入的参数值进行相应的偏移，这样就实现了随着手指的拖动，ListView的内容也会随着滚动的效果。

然后在第84行会进行判断，如果ListView中最后一个View的底部已经移入了屏幕，或者ListView中第一个View的顶部移入了屏幕，就会调用fillGap()方法，那么因此我们就可以猜出fillGap()方法是用来加载屏幕外数据的，进入到这个方法中瞧一瞧，如下所示：
```java
/**
 * Fills the gap left open by a touch-scroll. During a touch scroll,
 * children that remain on screen are shifted and the other ones are
 * discarded. The role of this method is to fill the gap thus created by
 * performing a partial layout in the empty space.
 * 
 * @param down
 *            true if the scroll is going down, false if it is going up
 */
abstract void fillGap(boolean down);
OK，AbsListView中的fillGap()是一个抽象方法，那么我们立刻就能够想到，它的具体实现肯定是在ListView中完成的了。回到ListView当中，fillGap()方法的代码如下所示：
void fillGap(boolean down) {
    final int count = getChildCount();
    if (down) {
        final int startOffset = count > 0 ? getChildAt(count - 1).getBottom() + mDividerHeight :
                getListPaddingTop();
        fillDown(mFirstPosition + count, startOffset);
        correctTooHigh(getChildCount());
    } else {
        final int startOffset = count > 0 ? getChildAt(0).getTop() - mDividerHeight :
                getHeight() - getListPaddingBottom();
        fillUp(mFirstPosition - 1, startOffset);
        correctTooLow(getChildCount());
    }
}
```
down参数用于表示ListView是向下滑动还是向上滑动的，可以看到，如果是向下滑动的话就会调用fillDown()方法，而如果是向上滑动的话就会调用fillUp()方法。那么这两个方法我们都已经非常熟悉了，内部都是通过一个循环来去对ListView进行填充，所以这两个方法我们就不看了，但是填充ListView会通过调用makeAndAddView()方法来完成，又是makeAndAddView()方法，但这次的逻辑再次不同了，所以我们还是回到这个方法瞧一瞧：
```java
/**
 * Obtain the view and add it to our list of children. The view can be made
 * fresh, converted from an unused view, or used as is if it was in the
 * recycle bin.
 *
 * @param position Logical position in the list
 * @param y Top or bottom edge of the view to add
 * @param flow If flow is true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @return View that was added
 */
private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
        boolean selected) {
    View child;
    if (!mDataChanged) {
        // Try to use an exsiting view for this position
        child = mRecycler.getActiveView(position);
        if (child != null) {
            // Found it -- we're using an existing child
            // This just needs to be positioned
            setupChild(child, position, y, flow, childrenLeft, selected, true);
            return child;
        }
    }
    // Make a new view for this position, or convert an unused view if possible
    child = obtainView(position, mIsScrap);
    // This needs to be positioned and measured
    setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);
    return child;
}
```
不管怎么说，这里首先仍然是会尝试调用RecycleBin的getActiveView()方法来获取子布局，只不过肯定是获取不到的了，因为在第二次Layout过程中我们已经从mActiveViews中获取过了数据，而根据RecycleBin的机制，mActiveViews是不能够重复利用的，因此这里返回的值肯定是null。

既然getActiveView()方法返回的值是null，那么就还是会走到第28行的obtainView()方法当中，代码如下所示：
```java
/**
 * Get a view and have it show the data associated with the specified
 * position. This is called when we have already discovered that the view is
 * not available for reuse in the recycle bin. The only choices left are
 * converting an old view or making a new one.
 * 
 * @param position
 *            The position to display
 * @param isScrap
 *            Array of at least 1 boolean, the first entry will become true
 *            if the returned view was taken from the scrap heap, false if
 *            otherwise.
 * 
 * @return A view displaying the data associated with the specified position
 */
View obtainView(int position, boolean[] isScrap) {
	isScrap[0] = false;
	View scrapView;
	scrapView = mRecycler.getScrapView(position);
	View child;
	if (scrapView != null) {
		child = mAdapter.getView(position, scrapView, this);
		if (child != scrapView) {
			mRecycler.addScrapView(scrapView);
			if (mCacheColorHint != 0) {
				child.setDrawingCacheBackgroundColor(mCacheColorHint);
			}
		} else {
			isScrap[0] = true;
			dispatchFinishTemporaryDetach(child);
		}
	} else {
		child = mAdapter.getView(position, null, this);
		if (mCacheColorHint != 0) {
			child.setDrawingCacheBackgroundColor(mCacheColorHint);
		}
	}
	return child;
}
```
这里在第19行会调用RecyleBin的getScrapView()方法来尝试从废弃缓存中获取一个View，那么废弃缓存有没有View呢？当然有，因为刚才在trackMotionScroll()方法中我们就已经看到了，一旦有任何子View被移出了屏幕，就会将它加入到废弃缓存中，而从obtainView()方法中的逻辑来看，一旦有新的数据需要显示到屏幕上，就会尝试从废弃缓存中获取View。所以它们之间就形成了一个生产者和消费者的模式，那么ListView神奇的地方也就在这里体现出来了，不管你有任意多条数据需要显示，ListView中的子View其实来来回回就那么几个，移出屏幕的子View会很快被移入屏幕的数据重新利用起来，因而不管我们加载多少数据都不会出现OOM的情况，甚至内存都不会有所增加。

那么另外还有一点是需要大家留意的，这里获取到了一个scrapView，然后我们在第22行将它作为第二个参数传入到了Adapter的getView()方法当中。那么第二个参数是什么意思呢？我们再次看一下一个简单的getView()方法示例：
```java
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	Fruit fruit = getItem(position);
	View view;
	if (convertView == null) {
		view = LayoutInflater.from(getContext()).inflate(resourceId, null);
	} else {
		view = convertView;
	}
	ImageView fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
	TextView fruitName = (TextView) view.findViewById(R.id.fruit_name);
	fruitImage.setImageResource(fruit.getImageId());
	fruitName.setText(fruit.getName());
	return view;
}
```
第二个参数就是我们最熟悉的convertView呀，难怪平时我们在写getView()方法是要判断一下convertView是不是等于null，如果等于null才调用inflate()方法来加载布局，不等于null就可以直接利用convertView，因为convertView就是我们之间利用过的View，只不过被移出屏幕后进入到了废弃缓存中，现在又重新拿出来使用而已。然后我们只需要把convertView中的数据更新成当前位置上应该显示的数据，那么看起来就好像是全新加载出来的一个布局一样，这背后的道理你是不是已经完全搞明白了？

之后的代码又都是我们熟悉的流程了，从缓存中拿到子View之后再调用setupChild()方法将它重新attach到ListView当中，因为缓存中的View也是之前从ListView中detach掉的，这部分代码就不再重复进行分析了。

为了方便大家理解，这里我再附上一张图解说明：

![img](/images/24/3.png)

那么到目前为止，我们就把ListView的整个工作流程代码基本分析结束了