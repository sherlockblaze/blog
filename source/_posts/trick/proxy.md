---
title: 如何让你的电脑使用感官如身在国外(for Mainland China)
tags:
  - trick
  - ss
date: 2019-02-27
---

可以访问到这篇文章的同学，首先一定具备了科学上网的能力，所以对于如何科学上网不赘述，本篇文章内容中所使用的翻墙方法是ss。但文章基本上可以帮助到通过socks网络连接来访问外网的同学。

## 系统环境

本文主要建立在 MacOs 系统环境下，顺便提到了 Linux-like 类系统的配置。至于 Windows 用帅气的[小飞机](https://github.com/shadowsocks/shadowsocks-windows/releases)就行了。

## 配置步骤

步骤其实很简单。

### 搭建 ss

首先让你的电脑能访问外网，建立起ss连接。执行以下命令进行安装：

```shell
pip install shadowsocks
```

你也可以使用 brew 进行安装

```shell
brew install shadowsocks-libv
```

我上面 brew install 后面跟的名称可能有误，请根据情况进行修改。不建议使用brew安装，经过个人的使用测试，使用 brew 安装的 shadowsocks 速度竟然是偏慢的，原理暂为探究。

使用第一种方式安装好以后，可能会有启动报错。这是因为openssl的更新，所以需要更改一下所安装的ss的源代码，以保证启动时不会报错。进入你的python modules的安装目录，例：`/usr/lib/python3.6/site-packages/shadowsocks/crypto/openssl.py`，然后做如下修改：

- 将`libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)`改为`libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)`
- 将`libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx)`改为`libcrypto.EVP_CIPHER_CTX_reset(self._ctx)`。

### ss 配置文件

写配置文件，可以命名为`ss-conf.json`，放置在你想放置的地方。然后通过以下命令启动：

```shell
sslocal -c /yourpath/ss-conf.json -d start
```

为了方便你需要开机执行这条命令，于是，可以添加该命令到你的 `/usr/local/bin` 目录下：在该目录下新建文件 named ss-start，然后编写，`echo "root's pwd" | sudo -S sslocal -c /yourpath/ss-conf.json -d start`，其中，sslocal命令可能需要使用绝对路径来描述，建议先用命令行的方式执行正确后再写此文件，完成编辑后，使用命令`sudo chmod 755 /usr/local/bin/ss-start` 赋予执行权限，使用命令 `ss-start` 来测试是否成功，最后添加到快捷启动中即可。


### Pac文件下载

下载[pac](https://sherlockblaze.com/resources/file/pac/selfproxy.pac)文件，我已经帮忙生成好了，直接下载就完了。注意其中的一行`SOCKS5 127.0.0.1:1080`，其中 `1080` 为你本地启动ss用的端口号，参数名称是 `local_port`，默认是 `1080`，注意修改成你实际使用的数值。

### 安装&配置 Privoxy

安装privoxy，我在mac上进行的操作，直接 `brew install privoxy`，找到这样的一行 `forward-socks5t   /   127.0.0.1:9050 .`，注意哈，空格后面有个 `.` 号，将其中的端口号 `9050` 修改成你 `local_port` 对应的实际参数值即可，然后启动 privoxy。如果你是在linux-like系统下，用你使用系统的常用安装方式安装privoxy即可，比如 ubuntu，执行命令 `sudo apt-get install privoxy`，配置文件位置在你安装完成后可以看到。

### Mac 环境开启本地 apache 服务

mac下，执行命令 `sudo apachectl start`，会提示你成功，如果不成功，自己搜一下报错并解决，然后把你下载好的文件放到 `/Library/WebServer/Documents` 文件夹下，打开浏览器，地址栏输入 `localhost/selfproxy.pac`，会看到下载的pac文件的内容，接下来按照图片中的内容配置一下。

![auto](https://sherlockblaze.com/resources/img/trick/automatic-proxy-configuration.png)

![socks](https://sherlockblaze.com/resources/img/trick/socks-proxy-configuration.png)

**注**：Ubuntu或Linux下，进入对应的网络配置里，配置代理，选择自动，然后添加pac文件路径即可，例：`file:///home/dashuaibi/selfproxy.pac`，如果不行，说明现在的要求高了，本地pac不让用了，你可以使用nginx搭建一个简单的本地文件服务，然后 `localhost/selfproxy.pac`即可。

## 完成后总览

- 享受飞一般的github clone速度

```shell
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

- 可使用的代理有三个，`http_proxy=http://127.0.0.1:8118`、 `https_proxy=https://127.0.0.1:8118`、 `socks5://127.0.0.1:1080`，你可以在任何软件分情况进行配置，舒服的走代理。

- Mac下软件只要按第二章图片配置好，除浏览器选择使用pac外，其他软件都可以舒服的走代理，比如 Mail，可以登陆 Google 邮箱了。

- 如果你想新访问什么外网网站，在pac文件中，找到两个数组的交界处，在红线往下，按格式写好即可。

![modify pac](https://sherlockblaze.com/resources/img/trick/modify-pac.png)

完成后在网络配置，自动发现代理中，删除掉配置确认后重新填入一下，因为需要重新缓存。记得不要删除 `sherlockblaze.com`，从国外视角看本博客，看到中文里不一样的内容。