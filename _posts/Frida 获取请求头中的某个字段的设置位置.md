# Frida 获取请求头中的某个字段的设置位置

在ios的网络请求中，设置请求头信息经常会用到

`-[NSMutableURLRequest setValue:forHTTPHeaderField:]`，所以可以通过追踪此函数，打印堆栈信息，从而快速定位到网络请求发送到位置。

1. 首先打开应用，然后在终端执行下面命令：

   frida -U -m “-[NSMutableURLRequest setValue:forHTTPHeaderField:]” 应用名称

2. 在handlers文件夹中修改 setValue_forHTTPHeaderField_.js 内容，然后重现执行上面的命令

```js
  onEnter(log, args, state) {
    log(`-[NSMutableURLRequest setValue:${args[2]} forHTTPHeaderField:${args[3]}]`);

    log(`args 2:` + ObjC.Object(args[2]) + "]");
    log(`args 3:` + ObjC.Object(args[3]) + "]");

    log('\n\n\tBacktrace:\n\t' + Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n\t'));  //打印堆栈
  },
  onLeave(log, retval, state) {
  }
}
```

