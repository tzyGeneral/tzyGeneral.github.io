# 抓包国外app，Mitmproxy配置



废话不多说，直接上代码

```python
from mitmproxy import http
from typing import Any
import json

import mitmproxy.log
from config import Config

from redis_tool import RedisQueue
from redis_key import RedisKey


class Encode(json.JSONEncoder):
    def default(self, o: Any) -> Any:
        if isinstance(o, bytes):
            return str(o, encoding='utf-8')
        return json.JSONEncoder.default(self, o)


class Events:
    def log(self, entry: mitmproxy.log.LogEntry):
        print(25, entry.msg)


class Trans:
    redisHandle = None

    def __init__(self):
        self.redisHandle = RedisQueue()

    def request(self, flow: http.HTTPFlow) -> None:
        """The full HTTP request has been read."""
        # print(flow.request.method, flow.request.url)
        address = ('127.0.0.1', 7890)
        if flow.live:
            flow.live.change_upstream_proxy_server(address)

    # def response(self, flow: http.HTTPFlow):
    #     print(flow.response.text)


addons = [Trans()]

# "mitmdump --mode upstream:http://127.0.0.1:7890/ -s xxx.py"

```

首先给把手机连接上mitmdump的代理端口，正常操作流程添加证书，我是使用的8080默认端口

然后在命令行输入开启抓包命令

```shell
mitmdump --mode upstream:http://127.0.0.1:7890/ -s xxx.py
```

注意，我电脑上开的是clash代理软件，端口是7890，假如你使用的是别的代理软件，就改成相应的端口。



此时手机既可以抓包，又能访问外网了

