---
layout: post
title: 手动进行 Nexus 和 Pixel 系列手机的 OTA 更新
image: '/images/手动进行 Nexus 和 Pixel 系列手机的 OTA 更新.jpg'
---

本文将介绍谷歌 OTA 更新的推送机制，如何快速获取最新的 OTA 更新推送，以及如何手动刷入 OTA 更新文件（不会清除数据，效果和正常 OTA 更新一样，未解锁的设备也可以这样做）。

OTA 全称 Over-the-air programming，意为空中编程，是一种为设备分发新软件、配置等的方法。OTA 的一项特征是，一个中心位置可以向所有用户发送更新，该更新立即应用到频道上的每个人。

谷歌会以 OTA 的形式对 Nexus 和 Pixel 系列手机每月会推送一次系统安全补丁更新，每年推送一次大的系统版本更新。但按照谷歌的推送机制，每次推送的第一批对象只占所有可更新用户的 1%，第二批推送对象占可更新用户的 11%，一直这样直到全部推送完毕。这样做是为了，如果新推送的更新出现问题，可以把坏的影响控制在最小范围内。

<br/>

## 快速获取 OTA 更新推送

[**根据谷歌软件工程师 Elliott Hughes 所说**](https://9to5google.com/2017/09/22/google-android-ota-check-for-update/)，要想最快获取 OTA 更新的推送，需要主动去点击**设置 > 关于手机 > 系统更新 > 检查更新（Check for update）**，这样相当于主动请求更新，可以跳过推送队列，尽早收到更新。 

但总有人会抢在您前面点击检查更新，所以您仍然没有收到更新推送，这时候我们可以手动刷入 OTA 更新文件。

<br/>

## 手动刷入 OTA 更新文件

**注意：**在应用更新之前，最好要先备份照片等个人数据。

### 前期准备

首先从发布 OTA 更新文件的[**官方页面**](https://developers.google.com/android/ota)，找到并下载跟自己设备对应的最新的 OTA 更新文件。

使用 OTA 更新文件更新设备，需要用到 **ADB**（Android Debug Bridge，安卓调试工具）。您可以从这里[**下载**](https://developer.android.com/studio/releases/platform-tools.html)它。还需确定您已在电脑上安装有原始设备制造商 USB 驱动程序（OEM USB Drivers），以及已开启设备中的 USB 调试（USB debugging）。

然后将下载好的 ADB 压缩文件解压（推荐解压至在 D 盘根目录下，方便后续操作），会得到一个名为 **platform-tools** 的文件夹，再将刚才下载的 OTA 更新文件放入这个文件夹。为了方便，我已将 OTA 更新文件**重命名为 ota_file.zip**。

为了能用 ADB 调试设备，我们需要将 ADB 添加进系统环境变量。**鼠标右键点击我的电脑 > 属性（Properties）> 高级系统设置（Advanced system settings）> 环境变量（Environment Variables）> 选中Path> 修改（Edit）**。

![img](/images/manually-update-nexus-and-pixel-series-phone-by-ota/1.jpeg)

点击**新建（New）**，在这里将我们的 platform-tools 文件夹的路径输入进去。然后保存我们刚刚所做的各种改变。

![img](/images/manually-update-nexus-and-pixel-series-phone-by-ota/2.jpeg)

现在我们需要打开命令行刷入 OTA 更新文件。按住 **Win + R 键**，在弹出的对话框中输入 **cmd** 以打开命令行窗口。输入 **d:** 并运行（意为前往 platform-tools 文件夹所在的盘符），再输入 **cd D:\platform-tools** 并运行（意为前往 platform-tools 文件夹所在的目录）。

![img](/images/manually-update-nexus-and-pixel-series-phone-by-ota/3.jpeg)

### 正式开始

连接上设备，下面开始手动刷入 OTA 更新文件：

1. 确认设备的确没有收到 OTA 更新推送，通过转到**设置 > 关于手机 > 系统更新**，应该显示“您的系统已是最新（Your system is up to date）”。

2. 在命令行中运行这条语句：

```
adb reboot recovery
```

现在设备会处于恢复模式（Recovery mode），屏幕上应该显示带有红色感叹号的 Android 图标或机器人。

3. 长按**电源键**，然后按一下**音量 + 键**，将出现一个菜单。使用音量按键上下选择，电源键确认，选择**从 ADB 应用更新（ Apply update from ADB）**。

4. 在命令行中运行这条语句：

```
adb devices
```

然后在命令行中，您的设备名称旁边会显示“ sideload ”。

5. 在命令行中运行这条语句：
{% highlight js %}
adb sideload ota_file.zip
{% endhighlight %}
更新完成后，选择**立即重启（Reboot system now）**。

为了安全起见，当设备不需要更新时，应该禁用 USB 调试。
