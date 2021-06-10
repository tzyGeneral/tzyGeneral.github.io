## python + selenium 获取session和token



### 背景

开门见山的说，最近使用selenium进行自动登陆相关业务，需要返回登陆后的cookie信息然后进行存储。但是突然遇到一个刺头，使用的是jwt，存储在sessionStrong里面，之前取cookie的方式无法获取到（当然，都不存在cookie :) ）



### 解决

直接上代码吧

```python
# 获取sessionid
    def get_sessionid(self):
        # 是要从localStorage中获取还是要从sessionStorage中获取，具体看目标系统存到哪个中
        # window.sessionStorage和直接写sessionStorage是等效的
        # 一定要使用return，不然获取到的一直是None
        # get的Item不一定就叫sessionId，得具体看目标系统把sessionid存到哪个变量中
        sessionid = self.browser.execute_script('return sessionStorage.getItem("sessionId");')

        # 另外sessionid一般都直接通过返回Set-Cookies头设置到Cookie中，所以也可以从Cookie读取
        # 获取浏览器所有Set-Cookie，返回对象是字典列表
        # cookies = self.browser.get_cookies()
        # 获取单项Cookie，是不是叫sessionId取决于系统存成什么变量，单项Cookie是字典
        # cookie = self.browser.get_cookie("sessionId")
        # cookie = cookie["value"]
        # print(f"{cookies}")
        return sessionid

    # 获取token
    def get_token(self):
        # 是要从localStorage中获取还是要从sessionStorage中获取，具体看目标系统存到哪个中
        # window.sessionStorage和直接写sessionStorage是等效的
        # 一定要使用return，不然获取到的一直是None
        # get的Item不一定就叫token，得具体看目标系统把token存到哪个变量中
        token = self.browser.execute_script('return sessionStorage.getItem("token");')
        # print(f"{token}")
        return token
```

