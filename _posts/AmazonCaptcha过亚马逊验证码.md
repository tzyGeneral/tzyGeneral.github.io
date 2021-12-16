# AmazonCaptcha过亚马逊验证码

### 背景

最近采集亚马逊商品信息，一直调验证码，发现了AmazonCaptcha这个第三方库，过亚马逊验证码是真的好用，使用也非常简单。

```python
#pip 安装
pip install amazoncaptcha
```

获取到验证码图片连接可以下载本地，然后直接传入保存的路径：

```python
from amazoncaptcha import AmazonCaptcha

captcha = AmazonCaptcha(文件保存路径)
solution = captcha.solve() #识别后返回的结果，字符型
```

也可以不下载下来，直接传入验证码图片连接：

```python
from amazoncaptcha import AmazonCaptcha

link = 获取到的验证码图片链接
# 测试示例
#link = 'https://images-na.ssl-images-amazon.com/captcha/usvmgloq/Captcha_kwrrnqwkph.jpg'

captcha = AmazonCaptcha.fromlink(link)
solution = captcha.solve()#识别后返回的结果，字符型
```

如果是用selenium采集的话，更方便。只要在验证码页面停留一会儿，调用AmazonCaptcha.fromdriver()方法 ，可以直接获取到验证码的识别后的结果。下面是示例：

```python
from amazoncaptcha import AmazonCaptcha
from selenium import webdriver

driver = webdriver.Chrome() 
driver.get('https://www.amazon.com/errors/validateCaptcha')

captcha = AmazonCaptcha.fromdriver(driver)
solution = captcha.solve()#识别后返回的结果，字符型
```

