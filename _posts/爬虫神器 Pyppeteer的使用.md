# 爬虫神器 Pyppeteer的使用



### Pyppteer介绍

Puppeteer 是 Google 基于 Node.js 开发的一个工具，有了它我们可以通过 JavaScript 来控制 Chrome 浏览器的一些操作，当然也可以用作网络爬虫上，其 API 极其完善，功能非常强大，Selenium 当然同样可以做到。

而 Pyppeteer 又是什么呢？它实际上是 Puppeteer 的 Python 版本的实现，但它不是 Google 开发的，是一位来自于日本的工程师依据 Puppeteer 的一些功能开发出来的非官方版本。

在 Pyppetter 中，实际上它背后也是有一个类似 Chrome 浏览器的 Chromium 浏览器在执行一些动作进行网页渲染，首先说下 Chrome 浏览器和 Chromium 浏览器的渊源。



> Chromium 是谷歌为了研发 Chrome 而启动的项目，是完全开源的。二者基于相同的源代码构建，Chrome 所有的新功能都会先在
> Chromium 上实现，待验证稳定后才会移植，因此 Chromium
> 的版本更新频率更高，也会包含很多新的功能，但作为一款独立的浏览器，Chromium
> 的用户群体要小众得多。两款浏览器“同根同源”，它们有着同样的 Logo，但配色不同，Chrome 由蓝红绿黄四种颜色组成，而
> Chromium 由不同深度的蓝色构成。



Pyppeteer 就是依赖于 Chromium 这个浏览器来运行的。那么有了 Pyppeteer 之后，我们就可以免去那些烦琐的环境配置等问题。如果第一次运行的时候，Chromium 浏览器没有安装，那么程序会帮我们自动安装和配置，就免去了烦琐的环境配置等工作。另外 Pyppeteer 是基于 Python 的新特性 async 实现的，所以它的一些执行也支持异步操作，效率相对于 Selenium 来说也提高了。

那么下面就让我们来一起了解下 Pyppeteer 的相关用法吧。



### 2. 安装

首先就是安装问题了，由于 Pyppeteer 采用了 Python 的 async 机制，所以其运行要求的 Python 版本为 `3.5` 及以上。

```python
pip3 install pyppeteer
```

测试

```python
import pyppeteer
```



#### 快速上手

```python
import asyncio
from pyppeteer import launch
from pyquery import PyQuery as pq
async def main():
   browser = await launch()
   page = await browser.newPage()
   await page.goto('https://dynamic2.scrape.cuiqingcai.com/')
   await page.waitForSelector('.item .name')
   doc = pq(await page.content())
   names = [item.text() for item in doc('.item .name').items()]
   print('Names:', names)
   await browser.close()
asyncio.get_event_loop().run_until_complete(main())
```

先初步看下代码，大体意思是访问了这个网站，然后等待 .item .name 的节点加载出来，随后通过 pyquery 从网页源码中提取了电影的名称并输出，最后关闭 Pyppeteer。

看运行结果，和之前的 Selenium 一样，我们成功模拟加载出来了页面，然后提取到了首页所有电影的名称。

那么这里面的具体过程发生了什么？我们来逐行看下。

+ launch 方法会新建一个 Browser 对象，其执行后最终会得到一个 Browser 对象，然后赋值给 browser。这一步就相当于启动了浏览器
+ 然后 browser 调用 newPage 方法相当于浏览器中新建了一个选项卡，同时新建了一个 Page 对象，这时候新启动了一个选项卡，但是还未访问任何页面，浏览器依然是空白。
+ 随后 Page 对象调用了 goto 方法就相当于在浏览器中输入了这个 URL，浏览器跳转到了对应的页面进行加载。
+ Page 对象调用 waitForSelector 方法，传入选择器，那么页面就会等待选择器所-对应的节点信息加载出来，如果加载出来了，立即返回，否则会持续等待直到超时。此时如果顺利的话，页面会成功加载出来。
+ 页面加载完成之后再调用 content 方法，可以获得当前浏览器页面的源代码，这就是 JavaScript 渲染后的结果。
+ 然后进一步的，我们用 pyquery 进行解析并提取页面的电影名称，就得到最终结果了。
  另外其他的一些方法如调用 asyncio 的 get_event_loop 等方法的相关操作则属于 Python 异步 async 相关的内容了，你如果不熟悉可以了解下前面所讲的异步相关知识。



接下来我们再看看另外一个例子，这个例子设定了浏览器窗口大小，然后模拟了网页截图，另外还可以执行自定义的 JavaScript 获得特定的内容，代码如下：

```python
import asyncio
from pyppeteer import launch
width, height = 1366, 768
async def main():
   browser = await launch()
   page = await browser.newPage()
   await page.setViewport({'width': width, 'height': height})
   await page.goto('https://dynamic2.scrape.cuiqingcai.com/')
   await page.waitForSelector('.item .name')
   await asyncio.sleep(2)
   await page.screenshot(path='example.png')
   dimensions = await page.evaluate('''() => {
       return {
           width: document.documentElement.clientWidth,
           height: document.documentElement.clientHeight,
           deviceScaleFactor: window.devicePixelRatio,
       }
   }''')

   print(dimensions)
   await browser.close()
asyncio.get_event_loop().run_until_complete(main())
```

这里我们又用到了几个新的 API，完成了页面窗口大小设置、网页截图保存、执行 JavaScript 并返回对应数据。

首先 screenshot 方法可以传入保存的图片路径，另外还可以指定保存格式 type、清晰度 quality、是否全屏 fullPage、裁切 clip 等各个参数实现截图。



### 3.详细用法

了解了基本的实例之后，我们再来梳理一下 Pyppeteer 的一些基本和常用操作。Pyppeteer 的几乎所有功能都能在其官方文档的 `API Reference` 里面找到，链接为：https://miyakogi.github.io/pyppeteer/reference.html，用到哪个方法就来这里查询就好了，参数不必死记硬背，即用即查就好。



#### 3.1 launch

使用 Pyppeteer 的第一步便是启动浏览器，首先我们看下怎样启动一个浏览器，其实就相当于我们点击桌面上的浏览器图标一样，把它运行起来。用 Pyppeteer 完成同样的操作，只需要调用 launch 方法即可。

我们先看下 launch 方法的 API，链接为：https://miyakogi.github.io/pyppeteer/reference.html#pyppeteer.launcher.launch，其方法定义如下：



- ignoreHTTPSErrors (bool)：是否要忽略 HTTPS 的错误，默认是 False。
- headless (bool)：是否启用 Headless 模式，即无界面模式，如果 devtools 这个参数是 True 的话，那么该参数就会被设置为 False，否则为 True，即默认是开启无界面模式的。
- executablePath (str)：可执行文件的路径，如果指定之后就不需要使用默认的 Chromium 了，可以指定为已有的 Chrome 或 Chromium。
- slowMo (int|float)：通过传入指定的时间，可以减缓 Pyppeteer 的一些模拟操作。
  args (List[str])：在执行过程中可以传入的额外参数。
- ignoreDefaultArgs (bool)：不使用 Pyppeteer 的默认参数，如果使用了这个参数，那么最好通过 args 参数来设定一些参数，否则可能会出现一些意想不到的问题。这个参数相对比较危险，慎用。
- handleSIGINT (bool)：是否响应 SIGINT 信号，也就是可以使用 Ctrl + C 来终止浏览器程序，默认是 True。
- handleSIGTERM (bool)：是否响应 SIGTERM 信号，一般是 kill 命令，默认是 True。
- handleSIGHUP (bool)：是否响应 SIGHUP 信号，即挂起信号，比如终端退出操作，默认是 True。
- dumpio (bool)：是否将 Pyppeteer 的输出内容传给 process.stdout 和 process.stderr 对象，默认是 False。
- userDataDir (str)：即用户数据文件夹，即可以保留一些个性化配置和操作记录。
  env (dict)：环境变量，可以通过字典形式传入。
- devtools (bool)：是否为每一个页面自动开启调试工具，默认是 False。如果这个参数设置为 True，那么 headless 参数就会无效，会被强制设置为 False。
- logLevel (int|str)：日志级别，默认和 root logger 对象的级别相同。
- autoClose (bool)：当一些命令执行完之后，是否自动关闭浏览器，默认是 True。
- loop (asyncio.AbstractEventLoop)：事件循环对象。



#### 3.2 无头模式

首先可以试用下最常用的参数 headless，如果我们将它设置为 True 或者默认不设置它，在启动的时候我们是看不到任何界面的，如果把它设置为 False，那么在启动的时候就可以看到界面了，一般我们在调试的时候会把它设置为 False，在生产环境上就可以设置为 True，我们先尝试一下关闭 headless 模式：

```python
import asyncio
from pyppeteer import launch
async def main():
   await launch(headless=False)
   await asyncio.sleep(100)
asyncio.get_event_loop().run_until_complete(main())
```



#### 3.3 调试模式

另外我们还可以开启调试模式，比如在写爬虫的时候会经常需要分析网页结构还有网络请求，所以开启调试工具还是很有必要的，我们可以将 devtools 参数设置为 True，这样每开启一个界面就会弹出一个调试窗口，非常方便，示例如下：

```python
mport asyncio
from pyppeteer import launch
 
async def main():
   browser = await launch(devtools=True)
   page = await browser.newPage()
   await page.goto('https://www.baidu.com')
   await asyncio.sleep(100)
 
asyncio.get_event_loop().run_until_complete(main())
```



#### 3.4 禁用提示条

这时候我们可以看到上面的一条提示：“Chrome 正受到自动测试软件的控制”，这个提示条有点烦，那该怎样关闭呢？这时候就需要用到 args 参数了，禁用操作如下：

```python
browser = await launch(headless=False, args=['--disable-infobars'])
```



#### 3.5 防止检测

如何规避呢？Pyppeteer 的 Page 对象有一个方法叫作 `evaluateOnNewDocument`，意思就是在每次加载网页的时候执行某个语句，所以这里我们可以执行一下将 WebDriver 隐藏的命令，改写如下：

```python
import asyncio
from pyppeteer import launch
 
async def main():
   browser = await launch(headless=False, args=['--disable-infobars'])
   page = await browser.newPage()
   await page.evaluateOnNewDocument('Object.defineProperty(navigator, "webdriver", {get: () => undefined})')
   await page.goto('https://antispider1.scrape.cuiqingcai.com/')
   await asyncio.sleep(100)
 
asyncio.get_event_loop().run_until_complete(main())
```



#### 3.7 用户数据持久化

刚才我们可以看到，每次我们打开 Pyppeteer 的时候都是一个新的空白的浏览器。而且如果遇到了需要登录的网页之后，如果我们这次登录上了，下一次再启动又是空白了，又得登录一次，这的确是一个问题。

比如以淘宝举例，平时我们逛淘宝的时候，在很多情况下关闭了浏览器再打开，淘宝依然还是登录状态。这是因为淘宝的一些关键 Cookies 已经保存到本地了，下次登录的时候可以直接读取并保持登录状态。

那么这些信息保存在哪里了呢？其实就是保存在用户目录下了，里面不仅包含了浏览器的基本配置信息，还有一些 Cache、Cookies 等各种信息都在里面，如果我们能在浏览器启动的时候读取这些信息，那么启动的时候就可以恢复一些历史记录甚至一些登录状态信息了。

这也就解决了一个问题：很多时候你在每次启动 Selenium 或 Pyppeteer 的时候总是一个全新的浏览器，那这究其原因就是没有设置用户目录，如果设置了它，每次打开就不再是一个全新的浏览器了，它可以恢复之前的历史记录，也可以恢复很多网站的登录信息。

那么这个怎么来做呢？很简单，在启动的时候设置 userDataDir 就好了，示例如下：

```python
import asyncio
from pyppeteer import launch
 
async def main():
   browser = await launch(headless=False, userDataDir='./userdata', args=['--disable-infobars'])
   page = await browser.newPage()
   await page.goto('https://www.taobao.com')
   await asyncio.sleep(100)
 
asyncio.get_event_loop().run_until_complete(main())
```

具体的介绍可以看官方的一些说明，如： https://chromium.googlesource.com/chromium/src/+/master/docs/user_data_dir.md，这里面介绍了 userdatadir 的相关内容。

再次运行上面的代码，这时候可以发现现在就已经是登录状态了，不需要再次登录了，这样就成功跳过了登录的流程。当然可能时间太久了，Cookies 都过期了，那还是需要登录的。

以上便是 launch 方法及其对应的参数的配置。



#### 3.8.1 开启无痕模式

我们知道 Chrome 浏览器是有一个无痕模式的，它的好处就是环境比较干净，不与其他的浏览器示例共享 Cache、Cookies 等内容，其开启方式可以通过 `createIncognitoBrowserContext` 方法，示例如下：

```python
import asyncio
from pyppeteer import launch
 
width, height = 1200, 768
 
async def main():
   browser = await launch(headless=False,
                          args=['--disable-infobars', f'--window-size={width},{height}'])
   context = await browser.createIncognitoBrowserContext()
   page = await context.newPage()
   await page.setViewport({'width': width, 'height': height})
   await page.goto('https://www.baidu.com')
   await asyncio.sleep(100)
 
asyncio.get_event_loop().run_until_complete(main())
```

这里关键的调用就是 createIncognitoBrowserContext 方法，其返回一个 context 对象，然后利用 context 对象我们可以新建选项卡。

运行之后，我们发现浏览器就进入了无痕模式



#### 3.9 选项卡操作

前面我们已经演示了多次新建选项卡的操作了，也就是 newPage 方法，那新建了之后怎样获取和切换呢，下面我们来看一个例子：

```python
import asyncio
from pyppeteer import launch
 
async def main():
   browser = await launch(headless=False)
   page = await browser.newPage()
   await page.goto('https://www.baidu.com')
   page = await browser.newPage()
   await page.goto('https://www.bing.com')
   pages = await browser.pages()
   print('Pages:', pages)
   page1 = pages[1]
   await page1.bringToFront()
   await asyncio.sleep(100)
 
asyncio.get_event_loop().run_until_complete(main())
```

在这里我们启动了 Pyppeteer，然后调用了 newPage 方法新建了两个选项卡并访问了两个网站。那么如果我们要切换选项卡的话，只需要调用 pages 方法即可获取所有的页面，然后选一个页面调用其 bringToFront 方法即可切换到该页面对应的选项卡。



#### 3.9.2 常见操作

作为一个页面，我们一定要有对应的方法来控制，如加载、前进、后退、关闭、保存等，示例如下：

```python
import asyncio
from pyppeteer import launch
from pyquery import PyQuery as pq
 
async def main():
   browser = await launch(headless=False)
   page = await browser.newPage()
   await page.goto('https://dynamic1.scrape.cuiqingcai.com/')
   await page.goto('https://dynamic2.scrape.cuiqingcai.com/')
   # 后退
   await page.goBack()
   # 前进
   await page.goForward()
   # 刷新
   await page.reload()
   # 保存 PDF
   await page.pdf()
   # 截图
   await page.screenshot()
   # 设置页面 HTML
   await page.setContent('<h2>Hello World</h2>')
   # 设置 User-Agent
   await page.setUserAgent('Python')
   # 设置 Headers
   await page.setExtraHTTPHeaders(headers={})
   # 关闭
   await page.close()
   await browser.close()
 
asyncio.get_event_loop().run_until_complete(main())
```



#### 3.9.4点击

Pyppeteer 同样可以模拟点击，调用其 click 方法即可。比如我们这里以 https://dynamic2.scrape.cuiqingcai.com/ 为例，等待节点加载出来之后，模拟右键点击一下，示例如下：

```python
import asyncio
from pyppeteer import launch
from pyquery import PyQuery as pq
 
async def main():
   browser = await launch(headless=False)
   page = await browser.newPage()
   await page.goto('https://dynamic2.scrape.cuiqingcai.com/')
   await page.waitForSelector('.item .name')
   await page.click('.item .name', options={
       'button': 'right',
       'clickCount': 1,  # 1 or 2
       'delay': 3000,  # 毫秒
   })
   await browser.close()
 
asyncio.get_event_loop().run_until_complete(main())
```



这里 click 方法第一个参数就是选择器，即在哪里操作。第二个参数是几项配置：

- button：鼠标按钮，分为 left、middle、right。
- clickCount：点击次数，如双击、单击等。
- delay：延迟点击。
- 输入文本。

对于文本的输入，Pyppeteer 也不在话下，使用 type 方法即可，示例如下：

```python
import asyncio
from pyppeteer import launch
from pyquery import PyQuery as pq
 
async def main():
   browser = await launch(headless=False)
   page = await browser.newPage()
   await page.goto('https://www.taobao.com')
   # 输入内容
   await page.type('#q', 'iPad')
   # 关闭
   await asyncio.sleep(10)
   await browser.close()
 
asyncio.get_event_loop().run_until_complete(main())
```

这里我们打开淘宝网，使用 type 方法第一个参数传入选择器，第二个参数传入输入的内容，Pyppeteer 便可以帮我们完成输入了。



Page 获取源代码用 content 方法即可，Cookies 则可以用 cookies 方法获取，示例如下：

```python
mport asyncio
from pyppeteer import launch
from pyquery import PyQuery as pq
 
async def main():
   browser = await launch(headless=False)
   page = await browser.newPage()
   await page.goto('https://dynamic2.scrape.cuiqingcai.com/')
   print('HTML:', await page.content())
   print('Cookies:', await page.cookies())
   await browser.close()
 
asyncio.get_event_loop().run_until_complete(main())
```



Pyppeteer 可以支持 JavaScript 执行，使用 evaluate 方法即可，看之前的例子：



```python
import asyncio
from pyppeteer import launch
 
width, height = 1366, 768
 
async def main():
   browser = await launch()
   page = await browser.newPage()
   await page.setViewport({'width': width, 'height': height})
   await page.goto('https://dynamic2.scrape.cuiqingcai.com/')
   await page.waitForSelector('.item .name')
   await asyncio.sleep(2)
   await page.screenshot(path='example.png')
   dimensions = await page.evaluate('''() => {
       return {
           width: document.documentElement.clientWidth,
           height: document.documentElement.clientHeight,
           deviceScaleFactor: window.devicePixelRatio,
       }
   }''')

   print(dimensions)
   await browser.close()
 
asyncio.get_event_loop().run_until_complete(main())
```

这里我们通过 evaluate 方法执行了 JavaScript，并获取到了对应的结果。另外其还有 exposeFunction、evaluateOnNewDocument、evaluateHandle 方法可以做了解。



#### 3.9.6 延时等待

在本课时最开头的地方我们演示了 `waitForSelector` 的用法，它可以让页面等待某些符合条件的节点加载出来再返回。

在这里 waitForSelector 就是传入一个 CSS 选择器，如果找到了，立马返回结果，否则等待直到超时。

除了 waitForSelector 方法，还有很多其他的等待方法，介绍如下。



- waitForFunction：等待某个 JavaScript 方法执行完毕或返回结果。
- waitForNavigation：等待页面跳转，如果没加载出来就会报错。
- waitForRequest：等待某个特定的请求被发出。
- waitForResponse：等待某个特定的请求收到了回应。
- waitFor：通用的等待方法。
- waitForSelector：等待符合选择器的节点加载出来。
- waitForXPath：等待符合 XPath 的节点加载出来。



### 其他

另外 Pyppeteer 还有很多功能，如键盘事件、鼠标事件、对话框事件等等，在这里就不再一一赘述了。更多的内容可以参考官方文档的案例说明：https://miyakogi.github.io/pyppeteer/reference.html。

以上，我们就通过一些小的案例介绍了 Pyppeteer 的基本用法，下一课时，我们来使用 Pyppeteer 完成一个实战案例爬取。

本节代码：https://github.com/Python3WebSpider/PyppeteerTest。

