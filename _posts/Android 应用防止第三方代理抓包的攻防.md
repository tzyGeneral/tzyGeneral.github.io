# Android 应用防止第三方代理抓包的攻防

**如何使请求不走默认的系统代理**

例如mitmproxy来抓取网络流量是需要设置系统代理的（一般来说）。普通的就是在手机wifi设置 -> 高级设置 -> 代理

一般网络请求库都提供了不走默认代理的功能，这里以常见的 HttpURLConnection 和 OkHttp为例子

HttpURLConnection：

```java
URL url = new URL(urlStr);
urlConnection = (HttpURLConnection) url.openConnection(Proxy.NO_PROXY);
```

OkHttp：

```java
OkHttpClient client = 
        new OkHttpClient().newBuilder().proxy(Proxy.NO_PROXY).build（）;
```

可见其关键在于 Proxy.NO_PROXY，我们来看看它是何方神圣。

```java
public final static Proxy NO_PROXY = new Proxy();
private Proxy() {
  type = Type.DIRECT;
  sa = null;
}
```

大概意思是告诉协议处理器不使用任何的代理，直连互联网。一般用于绕过全局代理。

此时，我们设置代理之后，使用 mitmproxy 确实抓不到请求包了。

**此时又该如何抓包呢**

这个时候开发表示很开心，可是做安全测试的小伙伴则有点不高兴了。不过没关系，还是有一些办法可以尝试的。

**Xposed hook 大法**

可以通过 Xposed 来 hook 掉设置代理的地方，修改为 burp 的代理。不过需要一些前置条件：手机需要完整 root，且仅限于 android 9 之前。

```java
if (lpparam.packageName.equals("ddns.android.vuls")) { 
        findAndHookMethod("java.net.URL", lpparam.classLoader, "openConnection", Proxy.class, new XC_MethodHook() {         
	   @Override 
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable { 
                Proxy proxy = (Proxy) param.args[0]; 
                if (proxy.type().equals(Proxy.Type.DIRECT)) { 
                    param.args[0] = new Proxy( Proxy.Type.HTTP, new InetSocketAddress("192.168.8.233",8080) ); 
                    XposedBridge.log("proxy " + (Proxy)param.args[0]); 
            }
        }); 

        findAndHookMethod("okhttp3.OkHttpClient.Builder", lpparam.classLoader, "proxy", Proxy.class, new XC_MethodHook() { 
            @Override 
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable { 
                Proxy proxy = (Proxy) param.args[0]; 
                if (proxy.type().equals(Proxy.Type.DIRECT)) { 
                    param.args[0] = new Proxy( Proxy.Type.HTTP, new InetSocketAddress("192.168.8.233",8080) ); 
                    XposedBridge.log("proxy " + (Proxy)param.args[0]); 
            }
        }); 
}
```

正常安装插件之后，就可以抓取数据包了.

**Frida hook 大法**

也可以通过 Frida hook 来去掉设置代理的地方。同样需要一些前置条件：frida-server 方式，手机需要 adb root；frida-gadget 方式，虽无需 root，仅适用于非系统应用。

```javascript
Java.perform(function() { 
    try { 
        var URL = Java.use("java.net.URL"); 
        URL.openConnection.overload('java.net.Proxy').implementation = function(arg1){ 
            return this.openConnection(); 
        } 
    } catch(e) { 
        console.log("" + e); 
    } 

    try { 
        var Builder = Java.use("okhttp3.OkHttpClient$Builder"); 
        var mybuilder = Builder.$new(); 
        Builder.proxy.overload('java.net.Proxy').implementation = function(arg1){ 
            return mybuilder; 
        } 
    } catch(e) { 
        console.log("" + e); 
    } 
});
```

运行 frida 脚本之后，就可以抓取数据包了

**流量转发**

还可以通过 iptables 将应用的流量转发到 burp 代理。同样需要一些前置条件：手机需要 adb root，且安装了 iptables（部分机型上可能无效）。

```shell
iptables -t nat -A OUTPUT -p tcp -m owner --uid-owner 应用uid  -j DNAT --to-destination ip:端口
```

通过 `ps -A | grep vuls` 命令来获取应用的uid

