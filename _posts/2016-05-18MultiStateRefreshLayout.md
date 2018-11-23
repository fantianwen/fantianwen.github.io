title: MultiStateRefreshLayout
date: 2016-05-18 11:25:00
tags: android
---

#### android官方的[SwipeRefreshlayout](https://developer.android.com/reference/android/support/v4/widget/SwipeRefreshLayout.html)控件不支持上拉加载，于是修改了[MultiStateView](https://github.com/Kennyc1012/MultiStateView)的部分代码，做为listView的footview，封装成了这个可以加载更多的控件，且footview的状态可以改变。

- 支持ListView
- 支持footView改变状态


<!-- more -->

## 使用方法

```
<van.tian.wen.multirefreshlayout.MultiStateRefreshLayout
        android:id="@+id/multiRefreshLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        van:emptyView="@layout/emptylayout"
        van:errorView="@layout/errorlayout"
        van:listView="@layout/lv"
        van:loadingView="@layout/loadinglayout"
        van:successView="@layout/successlayout"
        van:unknownView="@layout/unknownlayout">

    </van.tian.wen.multirefreshlayout.MultiStateRefreshLayout>
```

代码中：

```
 multiRefreshLayout = (MultiStateRefreshLayout) findViewById(R.id.multiRefreshLayout);
        multiRefreshLayout.setColorSchemeColors(Color.GREEN);

        mListView = multiRefreshLayout.getListView();
        multiRefreshLayout.setListView(mListView);

        myAdapter = new MyAdapter(this, mLists);
        mListView.setAdapter(myAdapter);

        multiRefreshLayout.setOnLoadingListener(new MultiStateRefreshLayout.OnLoadingListener() {
            @Override
            public void onLoadMore() {
                // 加载更多
            }
        });
        multiRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
               // 刷新
            }
        });
        multiRefreshLayout.setOnSucessFootClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 点击footView的加载加载更多
            }
        });

```

## 实现原理

关键代码：

```
public void setListView(final ListView mListView) {
        this.mListView = mListView;

        mListView.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {
                switch (scrollState) {
                    case SCROLL_STATE_FLING:
                        break;
                    case SCROLL_STATE_IDLE:
                        //监听是都能够上拉刷新
                        if (canRefresh()) {
                            setEnabled(true);
                        } else {
                            setEnabled(false);
                        }
                        //监听能够上拉加载更多
                        if (canLoadMore(scrollState)) {
                            loadData();
                        }
                        break;
                }
            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {

            }
        });

    }

```

其实主要解决的问题就是SwipeRefreshLayout的下拉操作会和ListView的下拉操作相冲突，我们只要在ListView的scrollListener中进行判断：

```
 case SCROLL_STATE_IDLE:
                        //监听是都能够上拉刷新
                        if (canRefresh()) {
                            setEnabled(true);
                        } else {
                            setEnabled(false);
                        }
                        //监听能够上拉加载更多
                        if (canLoadMore(scrollState)) {
                            loadData();
                        }
                        break;
```


其中`canRefresh()`代码如下：

```
private boolean canRefresh() {
        return isTop();
    }

    private boolean isTop() {
        if (mListView.getCount() > 0) {
            if (mListView.getFirstVisiblePosition() == 0
                    && mListView.getChildAt(0).getTop() >= mListView.getTop()) {
                return true;
            }
        }
        return false;
    }
```

## 不足

改变状态需要自主代码调用实现，因为`SwipeRefreshLayout`不能够判断已经刷新完毕，这点比较不人性化。


