# nginx 解决微信文章图片防盗链



解决图片防盗链的方法就是把请求头的`referer`去掉或者换成指定的内容

我们可以用`nginx`反向代理微信图片的链接，并把请求的`referer`进行替换

### 首先找到微信图片服务器的IP

```cmd
ping mmbiz.qpic.cn
```

### 配置微信图片反向代理

```nginx
    location ~ /mmbiz_(.*)/ {
        proxy_pass         http://59.57.18.143;

        proxy_set_header   Host             "mmbiz.qpic.cn";
        proxy_set_header   Referer          "";
    }
```

#### **移除微信图片的域名**

使用 `sub_filter` 移除微信图片的域名

```nginx
sub_filter "http://mmbiz.qpic.cn" "";
sub_filter_once off;
```

