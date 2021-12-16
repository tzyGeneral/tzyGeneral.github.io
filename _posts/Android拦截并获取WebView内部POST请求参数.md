# Android拦截并获取WebView内部POST请求参数

## 实现过程

### 方案一

在`shouldInterceptRequest`中拦截所有请求

```java
webView.setWebViewClient(new WebViewClient() {
    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
        try {
            URL url = new URL(request.getUrl());
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }
        Log.e("InternetActivity", request + "");
        return super.shouldInterceptRequest(view, request);
    }

});
```

但是通过此方法只能获取get请求的参数（因为参数直接拼在了url链接中），对于post请求的参数无可奈何。



### 方案二

后来参考了[request_data_webviewclient](https://github.com/KonstantinSchubert/request_data_webviewclient)，有了新的实现方式，具体原理为：给H5注入一段js代码，目的是在每次Ajax请求都会调用Android原生的方法，将请求参数传给客户端。

**js注入代码：**如下：

```js
<script language="JavaScript">

    function generateRandom() {
      return Math.floor((1 + Math.random()) * 0x10000)
        .toString(16)
        .substring(1);
    }


    // This only works if `open` and `send` are called in a synchronous way
    // That is, after calling `open`, there must be no other call to `open` or
    // `send` from another place of the code until the matching `send` is called.
    requestID = null;
    XMLHttpRequest.prototype.reallyOpen = XMLHttpRequest.prototype.open;
    XMLHttpRequest.prototype.open = function(method, url, async, user, password) {
        requestID = generateRandom()
        var signed_url = url + "AJAXINTERCEPT" + requestID;
        this.reallyOpen(method, signed_url , async, user, password);
    };
    XMLHttpRequest.prototype.reallySend = XMLHttpRequest.prototype.send;
    XMLHttpRequest.prototype.send = function(body) {
        interception.customAjax(requestID, body);
        this.reallySend(body);
    };

</script>
```



**客户端拦截请求：**

```java

@Override
public final WebResourceResponse shouldInterceptRequest(final WebView view, WebResourceRequest request) {
    String requestBody = null;
    Uri uri = request.getUrl();

    // 判断是否为Ajax请求（只要链接中包含AJAXINTERCEPT即是）
    if (isAjaxRequest(request)) {
        // 获取post请求参数
        requestBody = getRequestBody(request);
        // 获取原链接
        uri = getOriginalRequestUri(request, MARKER);
    }

    // 重新构造请求，并获取response
    WebResourceResponse webResourceResponse = shouldInterceptRequest(view, new WriteHandlingWebResourceRequest(request, requestBody, uri));
    if (webResourceResponse == null) {
        return webResourceResponse;
    } else {
        return injectIntercept(webResourceResponse, view.getContext());
    }
}
```

