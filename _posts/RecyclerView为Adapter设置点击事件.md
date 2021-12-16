# RecyclerView为Adapter设置点击事件



### 方法一：在Adapter单独设置

在`Adapter`的`onCreateViewHolder`或者`onBindViewHolder`单独为每个组件设置点击事件，两处都可以，推荐在`onBindViewHolder`中实现.

**缺点**

> 如果点击的逻辑需要在Activity或者Fragment中使用就会变得麻烦，比如长按弹出一个Dialog然后进行逻辑处理，这种情况下把逻辑放到Adapter中绝对不是明智之举

### 方法二：提供自定义接口

**实现思路：在Adapter提供接口，由外部实现**

1. 首先在Adapter申明一个接口，里面提供等待实现的回调函数
2. 申明一个延迟初始化接口变量
3. 设置一个函数用以设置接口
4. 通过view自带的**setOnClickListener**、**setOnLongClickListener**的回调函数调用接口的函数，实现回调的效果。实现哪种按照需求，这里我同时实现了两个。
5. 最后一步，调用自定义的**setOnItemClickListener**并实现自己的回调逻辑

```kotlin
/*为节省篇幅，删去了无关代码*/
/*adapter类*/{
    /*步骤2的接口变量*/
    private lateinit var onItemClickListener: OnItemClickListener

    /*调用接口回调*/
    override fun onBindViewHolder(holder: mViewHolder, position: Int) {
        /*步骤4，实现接口回调*/
        if (this::onItemClickListener.isInitialized)/*判断lateinit var是已经初始化*/
            onItemClickListener.let {
                holder.cell.apply {
                    setOnClickListener {
                        onItemClickListener.onClick(it, position)
                    }
                    setOnLongClickListener {
                        onItemClickListener.onLongClick(it, position)
                        true
                    }
                }
            }
    }

    /*步骤3的函数*/
    fun setOnItemClickListener(onItemClickListener: OnItemClickListener){
        this.onItemClickListener=onItemClickListener
    }

    /*步骤1的接口*/
    interface OnItemClickListener {
        fun onClick(view: View, position: Int)
        fun onLongClick(view: View, position: Int)
    }
}
```

```kotlin
/*adapter外实现接口*/
madapter.setOnItemClickListener(object :ChannelAdapter.OnItemClickListener{
            override fun onClick(view: View, position: Int) {
            }
            override fun onLongClick(view: View, position: Int) {
            }
        })
```

### 方法三：使用回调

可以使用RxJava、LiveData等……思路和上一种方法相同