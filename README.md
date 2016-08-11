# LRecyclerView 

LRecyclerView是支持addHeaderView、 addFooterView、下拉刷新、分页加载数据的RecyclerView。

新增功能：SwipeMenu系列功能，包括Item侧滑菜单、长按拖拽Item，滑动删除Item等。

**它对 RecyclerView 控件进行了拓展，给RecyclerView增加HeaderView、FooterView，并且不需要对你的Adapter做任何修改。**


##效果图
![这里写图片描述](https://raw.githubusercontent.com/cundong/HeaderAndFooterRecyclerView/master/art/art1.png)

##Gradle
--

Step 1. 在你的根build.gradle文件中增加JitPack仓库依赖。

```
allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
    }
}
```	

Step 2. 在你的model的build.gradle文件中增加LRecyclerView依赖。
```
compile 'com.github.jdsjlzx:LRecyclerView:1.0.3'
```

LRecyclerView requires at minimum Java 7 or Android 4.0.


##项目简述
1. 下拉刷新、滑动到底部自动加载下页数据； 
2. 可以方便添加Header和Footer；
3. 头部下拉样式可以自定义；
4. 具备item点击和长按事件。
5. 网络错误加载失败点击Footer重新请求数据；
6. 可以动态为FooterView赋予不同状态（加载中、加载失败、滑到最底等）；
7. SwipeMenu系列功能，包括Item侧滑菜单、长按拖拽Item，滑动删除Item等。

<br>注意：EndlessLinearLayoutActivity.java类里面有标准完整的使用方法，请尽量在这个界面看效果。</b>



##Demo下载
[点我下载](https://raw.githubusercontent.com/jdsjlzx/LRecyclerView/master/app/app-release.apk)

##功能介绍

### 添加HeaderView、FooterView
```
//add a HeaderView
RecyclerViewUtils.setHeaderView(mRecyclerView, new SampleHeader(this));

//add a FooterView
RecyclerViewUtils.setFooterView(mRecyclerView, new SampleFooter(this));
```
添加HeaderView还可以使用下面两种方式：

```
View header = LayoutInflater.from(this).inflate(R.layout.sample_header,(ViewGroup)findViewById(android.R.id.content), false);
RecyclerViewUtils.setHeaderView(mRecyclerView, header);


CommonHeader headerView = new CommonHeader(getActivity(), R.layout.layout_home_header);
RecyclerViewUtils.setHeaderView(mRecyclerView, headerView);
```

上面的方式同样适用于FooterView。

### LScrollListener-滑动监听事件接口

LScrollListener实现了nRefresh()、onScrollUp()、onScrollDown()、onBottom()、onScrolled五个事件，如下所示：

```
void onRefresh();//pull down to refresh

void onScrollUp();//scroll down to up

void onScrollDown();//scroll from up to down

void onBottom();//load next page

void onScrolled(int distanceX, int distanceY);// moving state,you can get the move distance
```
 - onRefresh()——RecyclerView下拉刷新事件；
 - onScrollUp()——RecyclerView向上滑动的监听事件；
 - onScrollDown()——RecyclerView向下滑动的监听事件；
 - onBottom()——RecyclerView滑动到底部的监听事件；
 - onScrollDown()——RecyclerView正在滚动的监听事件；
 
使用：
```
mRecyclerView.setLScrollListener(new LRecyclerView.LScrollListener() {
            @Override
            public void onRefresh() {
                //下拉刷新
            }

            @Override
            public void onScrollUp() {
            }

            @Override
            public void onScrollDown() {
            }

            @Override
            public void onBottom() {
                // 加载更多（加载下页数据）
            }

            @Override
            public void onScrolled(int distanceX, int distanceY) {
            }

        });
 
```

###下拉刷新和加载更多

详见LScrollListener接口介绍。

####设置下拉刷新样式

```
mRecyclerView.setRefreshProgressStyle(ProgressStyle.BallSpinFadeLoader); //设置下拉刷新Progress的样式
mRecyclerView.setArrowImageView(R.drawable.iconfont_downgrey);  //设置下拉刷新箭头
```

AVLoadingIndicatorView库有多少效果，LRecyclerView就支持多少下拉刷新效果，当然你也可以自定义下拉刷新效果。

效果图：

![这里写图片描述](http://img.blog.csdn.net/20160701173404897)

####设置加载更多样式

拷贝LRecyclerview_library中的文件文件layout_recyclerview_list_footer_loading.xml到你的工程中，修改后即可。


###开启和禁止下拉刷新功能

```
mRecyclerView.setPullRefreshEnabled(true);
```

or
```
mRecyclerView.setPullRefreshEnabled(false);
```

默认是开启。


###强制刷新

根据大家的反馈，增加了一个强制刷新的方法，使用如下：

```
mRecyclerView.forceToRefresh();
```

**无论是下拉刷新还是强制刷新，刷新完成后调用下面代码：**

```
mRecyclerView.refreshComplete();
mLRecyclerViewAdapter.notifyDataSetChanged();
```


### 加载网络异常处理
--------
加载数据时如果网络异常或者断网，LRecyclerView为你提供了重新加载的机制。

效果图：

![这里写图片描述](https://raw.githubusercontent.com/cundong/HeaderAndFooterRecyclerView/master/art/art5.png)

网络异常出错代码处理如下：

```
RecyclerViewStateUtils.setFooterViewState(getActivity(), mRecyclerView, getPageSize(), LoadingFooter.State.NetWorkError, mFooterClick);

private View.OnClickListener mFooterClick = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            RecyclerViewStateUtils.setFooterViewState(getActivity(), mRecyclerView, getPageSize(), LoadingFooter.State.Loading, null);
            requestData();
        }
    };
```

上面的mFooterClick就是我们点击底部的Footer时的逻辑处理事件，很显然我们还是在这里做重新请求数据操作。


### 点击事件和长按事件处理

先看下怎么使用：

```
mLRecyclerViewAdapter.setOnItemClickLitener(new OnItemClickLitener() {
            @Override
            public void onItemClick(View view, int position) {
                ItemModel item = mDataAdapter.getDataList().get(position);
                Toast.makeText(EndlessLinearLayoutActivity.this, item.title, Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onItemLongClick(View view, int position) {
                ItemModel item = mDataAdapter.getDataList().get(position);
                Toast.makeText(EndlessLinearLayoutActivity.this, "onItemLongClick - " + item.title, Toast.LENGTH_SHORT).show();
            }
        });
```

原理就是实现viewHolder.itemView的点击和长按事件。由于代码过多就不贴出来了。

viewHolder源码如下：

```
public static abstract class ViewHolder {
        public final View itemView;
        int mPosition = NO_POSITION;
        int mOldPosition = NO_POSITION;
        long mItemId = NO_ID;
        int mItemViewType = INVALID_TYPE;
        int mPreLayoutPosition = NO_POSITION;
```

###设置空白View（setEmptyView）

```
mRecyclerView.setEmptyView(view);
```




##打赏

觉得本框架对你有帮助，不妨打赏赞助我一下，让我有动力走的更远。

<img src="http://img.blog.csdn.net/20160524104553306" width=240 height=280 />

