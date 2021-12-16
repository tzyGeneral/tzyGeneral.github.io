# nginx 部署 Let's encrypt 免费ssl证书流程



网站域名为www.2kju.com, 介绍部署 Let's Encrypt 免费ssl证书的方法

#### 安装nginx

首先确保安装了nginx（我的是宝塔安装的nginx），没有装的话执行命令

```shell
# 支持中文
yum install -y langpacks-zh_CN
yum install -y nginx

# 保证以后重启系统会自动启动nginx服务
systemctl enable nginx
```

#### 安装certbot

我们采用 `certbot` 脚本方式申请 `let's encript` 证书，依次执行如下命令安装该工具：

```shell
yum install -y epel-release
yum install -y certbot
```

执行命令

```shell
certbot certonly -d *.2kju.com --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

一般来说都要输入邮箱啥的，一路点过去就完事了

假如出错显示

```python
certbot.errors.AuthorizationError: Some challenges have failed.
```

这就得按照提示来去进行txt验证

```shell
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.YOURDOMAIN with the following value:

Fhx3AXM****************e4TchYU

Once this is deployed,
-------------------------------------------------------------------------------
Press Enter to Continue
```

此时登录域名管理后台,添加`_acme-challenge.YOURDOMAIN`域名的TXT记录,值为`Fhx3AXM****************e4TchYU`,保存后输入以下命令进行确认已经正常解析:
`dig -t txt _acme-challenge.YOURDOMAIN`
如果返回结果中有上面填写的值说明已经添加并解析成功,此时返回证书更新界面按回车继续.正常情况下会返回如下结果.

```shell
Waiting for verification...
Cleaning up challenges
Generating key (2048 bits): /etc/letsencrypt/keys/0001_key-certbot.pem
Creating CSR: /etc/letsencrypt/csr/0001_csr-certbot.pem
```

#### 证书续签

宝塔后台任务，添加

```shell
certbot renew --quiet --renew-hook "/etc/init.d/nginx reload"
```

保证每天执行一次即可

#### 取消证书

```shell
certbot revoke --cert-path /etc/letsencrypt/live/2kju.com/cert.pem
certbot delete --cert-name 2kju.com
```



#### niginx 配置

```nginx
server{
    listen 443 ssl;
    server_name  www.2kju.com;
    ssl_certificate "/etc/letsencrypt/live/2kju.com/fullchain.pem";
    ssl_certificate_key "/etc/letsencrypt/live/2kju.com/privkey.pem";
    
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;
}
```

