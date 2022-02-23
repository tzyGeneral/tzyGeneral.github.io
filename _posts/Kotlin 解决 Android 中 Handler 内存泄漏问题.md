# Kotlin 解决 Android 中 Handler 内存泄漏问题

如何在fragment中解决handler内存泄漏问题呢，直接看代码

```kotlin
private var handler: Handler = WithoutLeakHandler(this)

companion object {
        const val GET_LOCAL_VIDEOS: Int = 100
        /**
         * by moosphon on 2018/09/16
         * desc: 解决handler内存泄漏的问题，消息的处理需要放在内部类的{@link #Handler.handleMessage}
         */
        private class WithoutLeakHandler( fragment: VideoLocalFragment) : Handler(){
            private var mFragment: WeakReference<VideoLocalFragment> = WeakReference(fragment)

            override fun handleMessage(msg: Message) {
                super.handleMessage(msg)
                when(msg.what){
                    GET_LOCAL_VIDEOS -> {
                        val fragment = mFragment.get()

                        Log.e("VideoLocalFragment", "收到视频搜索完毕的消息了")
                        if (fragment != null){
                            fragment.adapter.setData(fragment.videoData!!)
                            fragment.fm_video_local_rv.adapter = fragment.adapter
                        }

                    }
                }
            }
        }
    }
    
    ......
    
     Thread(Runnable {
            videoData = MediaUtils.getLocalVideos(context)
            Log.e("VideoLocalFragment", "扫描本地视频的数量为->"+videoData?.size)
            val message= Message()
            message.what = GET_LOCAL_VIDEOS
            handler.sendMessage(message)
        }).start()
```

在kotlin中，我们需要在静态类 `WithoutLeakHandler` 中重写 `handleMessage` 方法，并在里面处理消息和刷新UI。