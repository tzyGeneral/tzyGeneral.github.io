# 使用 jadx，apktool 反编译apk，签名打包apk



### jadx 是一个非常好用的android反编译gui工具，下面就来介绍jadx工具

mac 上安装

```brew install jadx```

解压后的文件夹目录：

assets：静态资源文件夹

lib：以来的库，c语言写的，so库，第三方库，系统库，自己写的库，都在这里面，也有的把加密token放到so文件中

res：就是resource，资源文件夹，和上面assets的区别就是上面的不回被编译，这个res会被编译，国外游戏汉化就是改这个文件

<font color='red'>**Androidmanifest.xml**</font>：这个文件很重要，app启动的时候会读取这个文件，获取什么权限，首次加载什么页面，都在这里

<font color='red'>**class.dex**</font>：这个里面就是java编译之后的可执行文件，这里面就是真正的代码了，安卓就是执行这个文件，启动app，我们java编译之后就是.class文件，安卓的源代码编译后就是.dex文件

<font color='red'>**resource.arsc**</font>：这个是资源索引文件，这个是对应关系，dex文件直接找不到res，需要靠这个文件来找到对应的资源



### apktoll，反编译apk

安装 ```brew install apktool```

执行 ``` apktool d xxxx.apk ```

假如要执行hock操作，写xposed

首先要读懂<font color='red'>**Androidmanifest.xml**</font>文件

```package``` ，包名，hock需要包名来过滤

```name```，这个name很重要，你找代码的入口就要通过这个来找，

因为你看到的代码都是混淆了，你搜索都搜不到，你要通过线索来找，

有些代码是不能混淆的，方法名字是不能混淆的，这个安卓应用理论的name就是不能混淆的，这个是入口，你混淆了之后，就加载不了了，找不到这个name了，

这就是你找代码的线索，而且他告诉你了文件夹的路径，com.taobao,tao,TaobaoApplication，

activity也是非常的重要，一个activity就是一个界面，这个更加的关键，这个里面也是不能混淆的，所以你可以通过这个来找，

而且这个里面也是有路径的，每一个页面的名字，路径都有，而且不能混淆，这就是一个重要的线索



### 重新打包

安装 ```keytool```, ```jarsigner```

第一步，生成签名

第二步，给apk签名，使用jarsigner工具



生成证书命令

```shell
keytool -genkey -keystore my-release-key.keystore -alias my_alias -keyalg RSA -keysize 4096 -validity 10000
```

最后一个10000是有效期，可以设置长一点，太短了，没多久就过期了，你还要重新签名，打包，

这一步之后会生成一个签名，

使用证书给apk签名

```shell
jarsigner -sigalg MD5withRSA -digestalg SHA1 -keystore my-release-key.keystore -signedjar com.yuanfuhou.test_sign.apk yunfuhou.1.12.apk my_alias
```



#### 注意

有的APP在启动时会对签名校验，要逆APP，跳过校验

他写死了签名是什么，你的签名和他的签名不一致，就不能，

你要找到验证签名的地方，然后注释掉，然后才可以，



#### smail文件的阅读

这个apktool反编译之后会产生smali文件，

这个smali文件，类似汇编语言，但是比汇编简单许多，只要你会java，了解android的相关知识，就可以轻松的看懂，

这个smali文件是可以直接编辑器打开的，

这个smali文件就是一一对应那个java文件，可以直接对比jadx反编译的java文件，对比看，

所以一定要对应java文件看，否则smali文件是接近汇编的语言，你看起来是很费劲的，

你看里面有一个行号，这个是一个重点，你对应的去看对应的java文件就可以了，

不懂的还可以百度搜索一下，

**读这个smali文件的目的：**

**就是看到加密的地方在哪里，然后通过改smali文件，来达到目的，**

**或者是绕过app的签名验证，**

**或者是抓包用了代理，这个也可以绕过**

**或者是打印log，**

**这些都要通过修改smali文件来实现，**