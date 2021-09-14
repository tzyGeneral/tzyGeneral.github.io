# Friada Hook与Fiddler抓包手机淘宝

> 工具：python3.9  frida Frida-server mumu模拟器 Fiddler

首先安装python3的frida模块

```python
pip3 install frida==12.7.5
pip3 install frida-tools
```

**mumu模拟器打开root权限**

```shell
adb connect 127.0.0.1:7555
```

 使用 `adb devices` 查看是否有检测到模拟器

使用adb命令查看模拟器的cpu类型，输入

```shell
adb shell
getprop ro.product.cpu.abi
```

此时会输入这台手机的cpu信息，如 `x86`



安装这个型号去下载相应的frida-server安装包

frida-server下载地址：[https://github.com/frida/frida/releases?after=12.7.5](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffrida%2Ffrida%2Freleases%3Fafter%3D12.7.5)

下载完成后，把文件解压后推送到模拟器上

```shell
adb push frida-server-12.7.5-android-x86 /data/local/tmp/frida-server
```

**通过命令转发模拟器tcp端口**：

```shell
adb forward tcp:27043 tcp:27043 
adb forward tcp:27042 tcp:27042
```

然后adb shell进入虚拟机：

```shell
cd /data/local/tmp/    //进入frida-server所在目录
chmod 777 frida-server    //赋予权限
./frida-server    //启动运行
```

运行启动后，不要关闭cmd窗口，让他一直运行就好了。

重新打开一个cmd 本机执行 frida-ps -U 应该能看到模拟器上启动的包名。

接着用模拟器打开app，将代码中替换淘宝包名“com.taobao.taobao”，执行以下python hook 代码：

```python

import frida, sys

jscode = """

Java.perform(function () {

var SwitchConfig = Java.use('mtopsdk.mtop.global.SwitchConfig');

    SwitchConfig.isGlobalSpdySwitchOpen.overload().implementation = function(){

        var ret = this.isGlobalSpdySwitchOpen.apply(this, arguments);

        console.log("isGlobalSpdySwitchOpenl "+ret)

        return false

    }

})

"""

def on_message(message, data):

    if message['type'] == 'send':

        print("[*] {0}".format(message['payload']))

    else:

        print(message)

process = frida.get_usb_device().attach('APP包名')

script = process.create_script(jscode)

script.on('message', on_message)

script.load()

sys.stdin.read()

```

最后，打开fiddler，配置https，模拟器安装fiddler证书，配置好代理