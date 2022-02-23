# Moviepy 自动化视频处理：添加音频 背景音乐 实现多轨音频



**本文将讲述的内容**：

为视频文件添加背景音乐

支持视频原声音量调节

支持背景音乐循环播放，覆盖整个视频时长

**用到的函数**

+ `audio_loop`

```python
audio_loop(audioclip, nloops=None, duration=None)
***
audioclip: 音频文件
nloops：循环次数
duration：循环持续时长
***
```

作用：

循环播放视频音频剪辑，返回播放给定剪辑音频nloop次数或在持续时间秒内。

+ `CompositeAudioClip`

```python
CompositeAudioClip(audio_clip_lists)
***
audio_clip_lists: 音频文件列表，eg：[audio1, audio2]
***
```

作用：

通过组合多个AudioClips制作的剪辑，通过将多个音频片段放在一起而制作成的音频片段

示例：

```python
from moviepy.editor import *
***
为视频添加一个背景音乐
多轨音频合成
***

# 需要添加背景音乐的视频
video_clip = VideoFileClip(r'./video.mp4')
# 提取视频对应的音频，并调节音量
video_audio_clip = video_clip.audio.volumex(0.8)
# 背景音乐
audio_clip = AudioFileClip(r'./test.mp3').volumex(0.5)
# 设置背景音乐循环，时间和视频时间一致
audio = afx.audio_loop(audio_clip, duration=video_clip.duration)
# 设置背景音乐和背景音乐，音频叠加
audio_clip_add = CompositeAudioClip([video_audio_clip, audio])
# 视频写入背景音
final_video = video_clip.set_audio(audio_clip_add)
# 将处理完成的视频保存
final_video.write_videofile("video_result.mp4")
```

