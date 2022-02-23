# MacOs 安装 OpenResty 踩坑记录



### OpenResty

openResty 是基于 Nginx的一个“强化包”，除了包含Nginx的功能模块，它的功能更加丰富，集成了脚本语言 Lua 简化 Nginx二次开发，更加方便快速的搭建动态网关

安装命令

```shell
brew tap openresty/brew //  添加仓库
brew install openresty  //  安装

```

#### 问题一：连不上raw.githubusercontent.com port 443

```shell
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused Error: Failed to download resource "openresty-openssl--patch" Download failed: https://raw.githubusercontent.com/openresty/openresty/master/patches/openssl-1.1.0d-sess\_set\_get\_cb\_yield.patch
```

原因：域名被DNS污染了，需要重新设置一下本地hosts，域名解析的读取顺序是：浏览器缓存->本地hosts文件->外部DNS

解决：在本地hosts文件加上：199.232.4.133 raw.githubusercontent.com

#### 问题二：找不到GeoIP包

直接brew安装

```shell
brew install geoip
```

继续报错：

```shell
Warning: geoip 1.6.12 is already installed, it's just not linked
You can use `brew link geoip` to link this version.
```

按照提示

```shell
brew link geoip
```

继续报错：

```shell
Linking /usr/local/Cellar/geoip/1.6.12... 
Error: Could not symlink include/GeoIP.h
/usr/local/include is not writable.
```

这里涉及了一些写入的权限问题。解决办法主要是修改文件权限为当前用户。若路径下没有include文件夹还需要创建一下。

解决：

```shell
cd /usr/local
sudo mkdir include
sudo chown -R $(whoami) $(brew --prefix)/include
```

执行

```shell
brew link geoip
brew install openresty
```

