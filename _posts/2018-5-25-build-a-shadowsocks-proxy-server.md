---
layout: post
title: 搭建 Shadowsocks 代理服务器
image: '/images/搭建 Shadowsocks 代理服务器.jpg'
---

本文将在 Windows 上以 Shadowsocks 为例（SS / SSR / SSRR / V2Ray 等其他翻墙方式同理），并尽可能详细的图文结合的方式介绍如何自建翻墙服务器（科学上网）。

搭建服务器其实只需要两步：

1. 购买 VPS (Virtual Private Server，虚拟专用服务器，也就是 Shadowsocks 的服务端，可以视作提供翻墙流量的来源）
2. 部署 VPS（也就是设置 Shadowsocks 客户端中的 IP，端口号，密码，加密方式等等）

<br/>

## 购买 VPS

[**Vultr**](https://www.vultr.com/) ，这是一家 VPS 服务器提供商（VPS 的提供商还有很多，朋友可以根据自己的需求去寻找），可以用支付宝付款。下面我也将以 Vultr 为例，介绍如合自建翻墙服务器。

### 1. 创建账户
下图是 Vultr 官网的首页，在图中红色方框内分别填入邮箱地址和密码（密码必须包含数字，和大、小写字母，以及位数须在十位以上），在点击 **Create Account**（创建账户），账户就创建完成了。

![1](/images/build-a-server-for-cross-the-china-great-firewall/1.png)

### 2. 选择支付方式
点击下图中红色方框中的 **Alipay**（支付宝）以选择使用支付宝支付。

![2](/images/build-a-server-for-cross-the-china-great-firewall/2.png)

第一次支付的最低金额是 10 美元，支付宝会自动换算成人民币进行支付。 这里先讲讲 Vultr 的收费方式，当我们一次支付 10 美元之后，我们的 Vultr 账户里便会有 10 美元的存款。

![3](/images/build-a-server-for-cross-the-china-great-firewall/3.png)

我们在 Vultr 上购买服务器时，Vultr 会告诉我们这个服务器每月需要多少钱。比如我们购买洛杉矶服务器，2.5 美元 / 月，当我们开始使用我们所购买的服务器时，Vultr 不会从我们账户里直接扣掉 2.5 美元，而是会将 2.5 美元 / 月转换成 0.003472 美元 / 小时，这样的话方便我们随时更换或停止服务器，而不会造成浪费钱。

### 3. 购买服务器
点图下图中红色方框内的加号按钮开始选择购买服务器。

![4](/images/build-a-server-for-cross-the-china-great-firewall/4.jpeg)

首选选择服务器的位置，这里我们以 **Los Angeles**（洛杉矶）为例。服务器位置在哪里也就代表 IP 地址在哪里。

![5](/images/build-a-server-for-cross-the-china-great-firewall/5.jpeg)

然后在选择服务器的系统型号，这里我们以 **Ubuntu 16.04 x64** 为例。服务器信号跟接下来部署 VPS 有关，不同型号的服务器系统会使用不同的命令语句。

![6](/images/build-a-server-for-cross-the-china-great-firewall/6.jpeg)

最后选择我们想要的套餐。如下图所示，洛杉矶的 2.5 美元 / 月已经卖完了，这里我们只好选择 5 美元 / 月，也就是 0.007 美元 / 小时，同时也代表着有更多的流量可以给我们使用，对于 5 美元 / 月的这种类型，图上已经说明每月会有 1000G 的流量。

剩下的几项现在不用在意，最后点击如下图所示的红色方框中的 **Deploy Now**（部署）就购买成功了。

![7](/images/build-a-server-for-cross-the-china-great-firewall/7.png)

### 4. 查看服务器信息
购买成功之后就可以看到在自己服务器列表里，有了我们刚刚购买的服务器，点击刚刚购买服务器进入到服务器的详情信息。

![8](/images/build-a-server-for-cross-the-china-great-firewall/8.png)

在服务器的详情信息中，我们需要用到 **IP Address**（IP 地址）和 **Password**（密码），这两项都可以点击右边的复制按钮进行复制，下面就开始部署 VPS 。

![9](/images/build-a-server-for-cross-the-china-great-firewall/9.png)

<br/>

## 部署 VPS
下面我将以 Windows 平台为例，介绍如合部署虚拟服务器。

若是在 Mac 或是 Linux 平台上，可跳过接下来的 1，2 两步，直接在系统终端中输入 `ssh root@xxx.xxx.xxx.xxx（此为你服务器的 IP 地址）`，然后等待提示密码输入，在输入密码后，即可从接下来的第 3 步开始署虚拟服务器。

### 1. 下载用于部署 VPS 的软件
在这里我们使用 [**Xshell**](https://www.netsarang.com/download/main.html) 这款软件，当然也可以使用其他软件进行部署。首先来到 Xshell 的官方下载页面。点击如下图所示的红色方框中的 **Download**（下载）。

![10](/images/build-a-server-for-cross-the-china-great-firewall/10.jpeg)

这时候会进入资料填写页面，因为我们没有买过这款软件，所以我们以学者的身份获取这款软件。如图分别选择 Home and school user（家庭和学校用户），并填写 **First Name**（名字），**Last Name**（姓氏），与 **Email**（邮箱），然后点击 **Submit**（提交）。提交成功后可以在邮箱里收到这款软件的下载地址。

![11](/images/build-a-server-for-cross-the-china-great-firewall/11.jpeg)

### 2. 使用 Xshell 连接服务器
安装好后打开 Xshell ，点击如下图所示的红色方框中的 **New**（快捷键 Alt + N ） 按钮，会打开如同所示的界面。红色方框中的 **Name**（名称）选项可以任意填。现在将我们所购买的服务器的详情信息里的 IP 地址复制到 **Host** 的输入框中。

![12](/images/build-a-server-for-cross-the-china-great-firewall/12.jpeg)

接下来再点击如下图所示的红色方框中的 **Authentication**（认证），**User Name**（用户名）只输入 **root** ，再将所购买的服务器的详情信息里的密码复制粘贴到 **Password**（密码）输入框中在，再点击 **OK** 按钮。

![13](/images/build-a-server-for-cross-the-china-great-firewall/13.jpeg)

接下来会出现我们即将开始部署的 VPS 的列表，选择我们刚才所建的 **New Session**，在点击如下图所示的红色方框中的 **Connect**（连接）。

![14](/images/build-a-server-for-cross-the-china-great-firewall/14.jpeg)

第一次连接时，可能出现如下图所示的情况，这时只需要点击 **Accept and Save**（接受并且保存）即可。

![15](/images/build-a-server-for-cross-the-china-great-firewall/15.jpeg)

成功连接之后会如下图所示。如果不成功请检查前期 Xshell 的配置是否做好，或者有可能是因为我们所购买的服务器的 IP 被屏蔽，这时需要在 Vultr 中卸载掉我们所买的服务器，再重新购买其他地址的服务器。

![16](/images/build-a-server-for-cross-the-china-great-firewall/16.jpeg)

### 3. 使用命令行部署 VPS
在这里我们使用秋水逸冰大神的 [Shadowsocks 一键安装脚本](https://teddysun.com/486.html)，将下列三条语句分别复制粘贴到（在命令行中点击鼠标右键，选择 **Paste** 进行粘贴） Xshell 的命令行中，再按下键盘上的 **Enter** （回车键）按钮，一条语句执行完后在复制另一条进去。

{% highlight %}
wget — no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
{% endhighlight %}

{% highlight %}
chmod +x shadowsocks-all.sh
{% endhighlight %}

{% highlight %}
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
{% endhighlight %}

执行完后会出现如图所示的代码，这时候以从键盘输入数字 1（选择安装 Python 版）为例。

![17](/images/build-a-server-for-cross-the-china-great-firewall/17.jpeg)

接着会出现如图所示的代码，这里要求我们自定义输入连接 Shadowsocks 的密码，有可能你输入密码时会看不见密码，但密码仍然是输入了的，所以当你输入完密码之后再按 **Enter** 键，继续执行代码。

![18](/images/build-a-server-for-cross-the-china-great-firewall/18.jpeg)

接着会出现如图所示的代码，这里要求我们从 **1–65535** 之间选择一个端口。

![19](/images/build-a-server-for-cross-the-china-great-firewall/19.jpeg)

接着会出现如图所示的代码，这里要求我们选择加密方式，我们以 **ase — 256- gcm** 为例，从键盘输入 1 ，按下回车键即可（若选择其他加密方式，可能还会选择是否开启混淆，如需开启，再输入 y，按下回车即可）。这时候会再次确认是否安装 Shadowsocks 服务端，再次按下回车键即可。

![20](/images/build-a-server-for-cross-the-china-great-firewall/20.jpeg)

执行完以上几部后，VPS 就已经在开始部署了，等待部署完毕，会出现以下画面，就代表部署完成了。这里可以先不关闭 Xshell，因为后面我们可以继续安装 **Google BBR** 算法提升 Shadowsocks 服务端速度。

![21](/images/build-a-server-for-cross-the-china-great-firewall/21.jpeg)

### 4. 重启服务器以启用 Shadowsocks 服务端
回到 Vultr 的服务器页面，选中我们刚刚所部署好的服务器，点击 **Restart**（重启）按钮重启我们所购买的服务器，现在我们可以将刚才在 Xshell 中所填写的 IP 、端口、密码、加密方式填入 Shadowsocks 客户端中开始使用。

![22](/images/build-a-server-for-cross-the-china-great-firewall/22.png)

<br/>

## 推荐: 安装 Google BBR 加速算法

安装 **Google BBR**（BBR 官方项目地址：<https://github.com/google/bbr> ）加速算法可以大幅提升使用 Shadowsocks 时的速度，具体操作和开始部署 VPS 时一样，用 Xshell 连接上服务器，在命令行中分别执行以下三条语句：

{% highlight %}
Wget–no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
{% endhighlight %}

{% highlight %}
chmod +x bbr.sh
{% endhighlight %}

{% highlight %}
./bbr.sh
{% endhighlight %}

安装成功后执行：

{% highlight %}
lsmod | grep bbr
{% endhighlight %}

如果返回值为 **tcp_bbr** ，即说明 bbr 已安装成功，并且启动正常。

