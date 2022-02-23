# Squid 代理服务器（高匿名）

1. 安装 Squid

   ```shell
   yum install squid
   ```

   

2. 配置Squid，放通所有流量

   + 打开配置文件

     ```shell
     vim /etc/squid/squid.conf
     ```

   + 在文件末尾处 添加以下配置

     ```SHELL
     http_access allow all
     
     forwarded_for delete
     follow_x_forwarded_for deny all
     
     request_header_access Via deny all
     request_header_access X-Forwarded-For deny all
     request_header_access Referer deny all
     request_header_access From deny all
     request_header_access X-Cache deny all
     
     reply_header_access Via deny all
     reply_header_access X-Cache deny all
     reply_header_access Server deny all
     reply_header_access WWW-Authenticate deny all
     reply_header_access Link deny all
     
     dns_v4_first on
     ```

     

3. 使用`squid -k parse`分析是否设置有错误

4. `squid -z`进行初始化（这个命令神奇，要自己按回车键才会退出）。
5. 使用`service squid start`启动服务。