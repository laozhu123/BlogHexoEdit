---
title: Android中当item数量超过一定大小RecyclerView高度固定
date: 2018-04-08 10:27:30
categories: "java基础学习"
tags:
     - Android
     - 基础
---

### Android中当item数量超过一定大小时，将RecyclerView高度固定

#### 方法1

> 直接通过LayoutParams来设定相应高度

    ViewGroup.LayoutParams lp = rv.getLayoutParams();
    if (list.size() > 4) {
        lp.height = DensityUtil.dip2px(mContext,32 * 4);
    } else {
        lp.height = DensityUtil.dip2px(mContext,34 * list.size());
    }
    rv.setLayoutParams(lp);


该方法只适用于item高度固定，在本例中使用34dp来设置相应的item高度，故而可以通过乘上相应的item数来计算RecyclerView的高度。


#### 方法2
> 重写LayoutManger的onMeasure方法，这种方式可以获取到各个item的不同高度，从而可以设置变动的高度。

**在使用这种方式时，有一点需要注意的是，不要将RecyclerView的android:layout_height属性设置为wrap_content，不然是不会成功的。**

    rv.setLayoutManager(new LinearLayoutManager(mContext) {
                    @Override
                    public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state, int widthSpec, int heightSpec) {
                        int count = state.getItemCount();
                        if (count > 0) {
	                        int realHeight = 0;
	                        int realWidth = 0;
	                        for(int i = 0;i < count; i++){
		                        View view = recycler.getViewForPosition(0);
		                        if (view != null) {
	                                measureChild(view, widthSpec, heightSpec);
	                                int measuredWidth = View.MeasureSpec.getSize(widthSpec);
	                                int measuredHeight = view.getMeasuredHeight();    
	                                realWidth = realWidth > measuredWidth ? realWidth : measureWidth;
	                                realHeight += measureHeight;            
	                            }
	                            setMeasuredDimension(realWidth, realHeight); 
	                        }                           
                        } else {
                        super.onMeasure(recycler, state, widthSpec, heightSpec);
                   }
               }
            });



方法中的recycler是一个item的循环使用器，起到对item管理的作用。