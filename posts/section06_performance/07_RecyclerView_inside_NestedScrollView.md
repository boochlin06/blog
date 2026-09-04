# How to use RecyclerView inside NestedScrollView?

> 原文網址：https://boochlin.com/?p=169
> 發布日期：2016-08-17
> 文章編號：169

---

![correct_device](../../assets/images/correct_device.gif)

[https://boochlin.com/wp-content/uploads/2016/08/correct_device.gif](https://boochlin.com/wp-content/uploads/2016/08/correct_device.gif)

### 如何在 NestedScrollView 內遷入 RecyclerView ？

上面的圖，是最終完成的畫面。

至於這個問題就是沒辦法直接隨便在 xml 檔案裡隨便寫寫就可以搞定，如下。主要就是 RecycleView 和 NestedScrollView 的滑動會產生衝突。並非 spec 所要求。

以下面的 code 來跑。

![螢幕快照 2016-08-16 下午6.54.54](../../assets/images/螢幕快照-2016-08-16-下午6.54.54.png)

[https://boochlin.com/wp-content/uploads/2016/08/螢幕快照-2016-08-16-下午6.54.54.png](https://boochlin.com/wp-content/uploads/2016/08/螢幕快照-2016-08-16-下午6.54.54.png)

就會變成底下這樣，

![error_recycle](../../assets/images/error_recycle.gif)

[https://boochlin.com/wp-content/uploads/2016/08/error_recycle.gif](https://boochlin.com/wp-content/uploads/2016/08/error_recycle.gif)

主要就是高度的判定會重疊到，底下的 RecycleView 被 AppBarLayout 蓋住，在 AppBarLayout 展開情況下 ， RecycleView 不管怎麼滑動，AppBarLayout 都不為所動，只有當 AppBarLayout 擇疊起來，RecycleView 往下拉到底，才會判定 AppBarLayout 連動到。

因此這裡可以合理推斷 RecycleView 的高度判定是有問題的。從畫面上來看發現 RecycleView 的顯示範圍是在從 AppBarLayout 底部開始算，然後加上 RecycleView 的高度，而 RecycleView 的高度則是 match_parent ，這裡看起來是整個螢幕高度，那就會變成 RecycleView 的底部會超出手機螢幕，這也可以從畫面驗證，AppBarLayout 展開時， RecycleView 無論如何滑動都無法看到 item 9 , 10 ，但是當 AppBarLayout 擇疊時，RecycleView 的範圍判定就是正常的。

因此要修正的點就是重新 measure RecycleView 的範圍，進而修正 NestedScrollView 的行為。(有關 [NestedScrollView](https://segmentfault.com/a/1190000002873657) 這篇寫得不錯，NestedScrollView 出現主要是想要解決何種問題，主要是獨立出 class to handle parent and child touch event ，常搭配 CoordinatorLayout 的 Behavior 完成酷炫功能)

有關 RecycleView 的 wrap_content ，在 measure child 時，會出現 size = 0 的情況。這就照成後續的問題（[大神原始問題](http://RecyclerView cannot size itself based on the measured dimensions of its children)，已經回報給 google），因此這裡提出重寫 onMeasure 的方法，主要改成用 View.MeasureSpec.UNSPECIFIED 去讓 child 自己決定高度，最後加總得到 RecycleView 的高度。

ps. 上述問題會發生在 android support library 23.2 之前。如果是使用 23.2 (包含）之後的 RecycleView 之後，就可以用另外一套修正方法(跳過下面這段程式碼)

ref：

[http://stackoverflow.com/questions/31000081/how-to-use-recyclerview-inside-nestedscrollview](http://stackoverflow.com/questions/31000081/how-to-use-recyclerview-inside-nestedscrollview)

[https://github.com/emanuelet/LayoutManagers/blob/master/ExpansiveLayoutManager.java](https://github.com/emanuelet/LayoutManagers/blob/master/ExpansiveLayoutManager.java)

[http://blog.csdn.net/yaochangliang159/article/details/50540276](http://blog.csdn.net/yaochangliang159/article/details/50540276)

```
public class ExpandBarLinearLayoutManager extends LinearLayoutManager {

    public ExpandBarLinearLayoutManager(Context context, int orientation, boolean reverseLayout) {
        super(context, orientation, reverseLayout);
    }

    private int[] mMeasuredDimension = new int[2];

    @Override
    public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state,
                          int widthSpec, int heightSpec) {
        final int widthMode = View.MeasureSpec.getMode(widthSpec);
        final int heightMode = View.MeasureSpec.getMode(heightSpec);
        final int widthSize = View.MeasureSpec.getSize(widthSpec);
        final int heightSize = View.MeasureSpec.getSize(heightSpec);
        int width = 0;
        int height = 0;
        for (int i = 0; i < getItemCount(); i++) {

            if (getOrientation() == HORIZONTAL) {

                measureScrapChild(recycler, i,
                        View.MeasureSpec.makeMeasureSpec(i, View.MeasureSpec.UNSPECIFIED),
                        heightSpec,
                        mMeasuredDimension);

                width = width + mMeasuredDimension[0];
                if (i == 0) {
                    height = mMeasuredDimension[1];
                }
            } else {
                measureScrapChild(recycler, i,
                        widthSpec,
                        View.MeasureSpec.makeMeasureSpec(i, View.MeasureSpec.UNSPECIFIED),
                        mMeasuredDimension);
                height = height + mMeasuredDimension[1];
                if (i == 0) {
                    width = mMeasuredDimension[0];
                }
            }
        }
        switch (widthMode) {
            case View.MeasureSpec.EXACTLY:
                width = widthSize;
            case View.MeasureSpec.AT_MOST:
            case View.MeasureSpec.UNSPECIFIED:
        }

        switch (heightMode) {
            case View.MeasureSpec.EXACTLY:
                height = heightSize;
            case View.MeasureSpec.AT_MOST:
            case View.MeasureSpec.UNSPECIFIED:
        }

        setMeasuredDimension(width, height);
    }

    private void measureScrapChild(RecyclerView.Recycler recycler, int position, int widthSpec,
                                   int heightSpec, int[] measuredDimension) {
        View view = recycler.getViewForPosition(position);
        recycler.bindViewToPosition(view, position);
        if (view != null) {
            RecyclerView.LayoutParams p = (RecyclerView.LayoutParams) view.getLayoutParams();
            int childWidthSpec = ViewGroup.getChildMeasureSpec(widthSpec,
                    getPaddingLeft() + getPaddingRight(), p.width);
            int childHeightSpec = ViewGroup.getChildMeasureSpec(heightSpec,
                    getPaddingTop() + getPaddingBottom(), p.height);
            view.measure(childWidthSpec, childHeightSpec);
            measuredDimension[0] = view.getMeasuredWidth() + p.leftMargin + p.rightMargin;
            measuredDimension[1] = view.getMeasuredHeight() + p.bottomMargin + p.topMargin;
            recycler.recycleView(view);
        }
    }
}
```

然後一定要幫 RecycleView 設定 setNestedScrollingEnabled(false)

```
recycleView.setLayoutManager(new ExpandBarLinearLayoutManager(getActivity(), LinearLayoutManager.VERTICAL, false));
recycleView.setNestedScrollingEnabled(false);
```

然後如果直接使用 23.2 之後的 RecycleView 的人， google 已經修正了，只需要

```
LinearLayoutManager linearLayoutManager = new LinearLayoutManager(getActivity());
// 其實默認 auto measure is true.
linearLayoutManager.setAutoMeasureEnabled(true);
recycleView.setLayoutManager(linearLayoutManager);
// 這行不能少
recycleView.setNestedScrollingEnabled(false);
```

這裡我看原始碼，這一段就是修正 code ，還蠻直覺，並非直接強制使用 View.MeasureSpec.UNSPECIFIED（這問題還蠻明顯的，這會使得 child view measure 風險變很大，因為無法控制 child view 是適合哪一種），而是採用兩階段 layout ，藉此得到正確的 child size.

```
if (mLayout.mAutoMeasure) {
            final int widthMode = MeasureSpec.getMode(widthSpec);
            final int heightMode = MeasureSpec.getMode(heightSpec);
            final boolean skipMeasure = widthMode == MeasureSpec.EXACTLY
                    && heightMode == MeasureSpec.EXACTLY;
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
            if (skipMeasure || mAdapter == null) {
                return;
            }
            if (mState.mLayoutStep == State.STEP_START) {
                dispatchLayoutStep1();
            }
            // set dimensions in 2nd step. Pre-layout should happen with old dimensions for
            // consistency
            mLayout.setMeasureSpecs(widthSpec, heightSpec);
            mState.mIsMeasuring = true;
            dispatchLayoutStep2();

            // now we can get the width and height from the children.
            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);

            // if RecyclerView has non-exact width and height and if there is at least one child
            // which also has non-exact width & height, we have to re-measure.
            if (mLayout.shouldMeasureTwice()) {
                mLayout.setMeasureSpecs(
                        MeasureSpec.makeMeasureSpec(getMeasuredWidth(), MeasureSpec.EXACTLY),
                        MeasureSpec.makeMeasureSpec(getMeasuredHeight(), MeasureSpec.EXACTLY));
                mState.mIsMeasuring = true;
                dispatchLayoutStep2();
                // now we can get the width and height from the children.
                mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
            }
        }
```

有興趣的人，可以繼續 trace code ，但我就到此一鞠躬啦。

other reference:

[http://www.jianshu.com/p/4535442d568f](http://www.jianshu.com/p/4535442d568f)

[http://www.jianshu.com/p/06c0ae8d9a96](http://www.jianshu.com/p/06c0ae8d9a96)

[http://blog.csdn.net/feiduclear_up/article/details/46514791](http://blog.csdn.net/feiduclear_up/article/details/46514791)

[http://stackoverflow.com/questions/25702884/add-viewpagerindicator-to-android-studio](http://stackoverflow.com/questions/25702884/add-viewpagerindicator-to-android-studio)
