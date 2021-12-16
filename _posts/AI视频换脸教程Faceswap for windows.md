# AI视频换脸教程Faceswap for windows



## 1. 安装

官方下载地址：https://github.com/deepfakes/faceswap/releases



![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/7e49378d98d84dcbbcf4e8688321108e.jpg)

下载后双击exe安装，如果没有独显，就选 Setup for CPU

有英伟达显卡，就选 Setup for NVIDIA GPU

有AMD显卡，就选 Setup for AMD GPU



最好选择用GPU计算，因为CPU计算起来超级慢，就算是i9, 也不如一个普通GPU的计算能力

一个普通的短视频，就算是强如RTX-2070，可能都要不间断计算一个星期



![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/1e1ac955698341d79261ef8ffeb3f14f.jpg)



## 2. 获取视频序列帧

准备两段包含人脸的视频，最好是正脸

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/bc862ee6aa23485cac82d3959e1d212a.jpg)

然后我们需要把视频转化为序列帧，随便用什么方法、什么软件都行，在此，我们使用opencv-python去处理

新建两个文件夹p1_Input和p2_Input （命名随意） p1_Input用来存第一个视频的序列帧，

p2_Input用来存第二个视频的序列帧，然后运行python脚本



事先 pip install opencv-python

```python
import cv2

cap=cv2.VideoCapture(r'E:\PyCharmProjects\FaceswapFiles\燃向热血高校_小栗旬帅炸.mp4')
index=0
 
while True:
    _,frame=cap.read()
    if not _:
        break
    cv2.imshow('frame',frame)
    cv2.imwrite(r'E:\PyCharmProjects\FaceswapFiles\p1_Input\\'+'p'+str(index)+'.jpg',frame)
    index += 1
    c=cv2.waitKey(30)
    if c==27:
        break
```

另一个视频同理，改下路径即可

处理完后，对应的序列帧会存在对应的文件夹下

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/b2a30dfc66524ce78b5f52648c908ee8.jpg)

## 3. 提取序列帧中的人脸

首先新建两个空文件夹，用来存放提取的人脸图，例如p1_Output和p2_Output

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/80c9c262af7d46c89c46dd1d01bd5547.jpg)

然后打开faceswap软件

选中Extract

Input Dir 选为上文的p1_Input文件夹

Output Dir 选为上文的p1_Output文件夹

点击下方的Extract开始提取人脸

如果是第一次运行，点击Extract后会安装各种库，会比较慢

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/3cff5d5339624515bb257f2307ba3ebf.jpg)

执行完后，

会在p1_Output文件夹下生成人脸图，然后检查一下，类似p0044_0.jpg和p0045_0.jpg这种有问题的图片，手动删掉

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/6dc8ef60e12f4eb88a14c658761ed77b.jpg)

然后另一个视频同理

更改一下路径，输入和输出路径分别改为p2_Input和p2_Output ，重复上述操作

## 4. 训练

新建一个空文件夹，取名models，用来存放训练后的模型

此时选择Train一栏

Input A 选择 p1_Output

Input B 选择 p2_Output

Model Dir 选择 刚才新建的models文件夹

然后点击下方的Train按钮开始训练

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/274a2a0d9a0f4e289a430b88e363123e.jpg)

训练是个漫长的过程，根据处理的内容的复杂度，官方提示需要24小时至一个星期或者更长，要有耐心

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/10ee056811764bce8f1a71f0de63fe81.jpg)

根据电脑的性能，每隔一段时间，下方控制台会刷新一下损失函数的值，训练可以根据需求随时中断，官方建议，当值小于0.02效果会比较理想

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/270ea116dc2c45c082340d5f2a665a4c.jpg)

随时可中断训练，此时模型的精度，就是你中断时控制台上显示的损失函数的值，中断后，训练的模型会存在上文新建的models文件夹下

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/85607a649ace4c1ea7d4a8052600dbd1.jpg)

## 5. 换脸

新建个空文件夹，取名Final_Output，用来存放换脸后的最终的序列帧

选择Convert一栏

假如想把赵四的脸换到换到泷谷源治上

则：

Input Dir 需要选择 p1_Input路径，即视频1的原始序列帧

Output Dir 选择刚才创建的Final_Output文件夹路径

Model Dir 选择第4步保存模型的models文件夹路径

最后点击下方的Convert按钮开始换脸

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/19b60c76135141409b03714d403908ba.jpg)

## 6. 序列帧转为视频

第5步执行完后，可在Final_Output文件夹中找到最终换脸后的序列帧

此时我们再把序列帧转化为视频，用什么方式转换都行

在此，我们同样用opencv-python去处理

```python
import cv2
import os
 
pathDir = os.listdir(r'E:\PyCharmProjects\FaceswapFiles\Final_Output')
fourcc = cv2.VideoWriter_fourcc('M', 'J', 'P', 'G')
fps=20
videoWriter = cv2.VideoWriter(r'E:\PyCharmProjects\FaceswapFiles\output.avi', fourcc, fps, (1920,1080))
 
 
 
for x in pathDir:
    path='E:\\PyCharmProjects\\FaceswapFiles\\Final_Output\\'+x
    print(path)
    frame=cv2.imread(path)
    cv2.waitKey(30)
    videoWriter.write(frame)
```

![Faceswap：AI视频换脸教程, 换脸软件使用教程 Faceswap for windows](https://justcode.ikeepstudying.com/wp-content/uploads/2020/10/ef29626e818b499492a5e139c913fd7d.jpg)