---
title: PC实用工具记录
date: 2020-03-04 09:50:19
categories: 实用工具
tags:
---

### 实用工具记录（Windows)

在这里记录一下自己在用的一些十分好用的工具，方便以后换电脑继续用，同时也先向各位看到的小伙伴做一个推荐，下面开始：

---

#### Chrome浏览器
![chrome](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020chrome.jpg)



大声喊出来，换新电脑开机后第一步干什么：**从浏览器下载另一个浏览器**

虽然windows10已经绑定MicroSoft Edge浏览器，而且最近已经更新为使用Chromium内核（就可以使用Chrome商店上的应用），但就个人观点，还是Chrome更为好用，强大的功能，丰富的插件，更快的速度，稳定的性能；60%以上的市场份额不是说说的，同时这也决定了几乎所有的浏览器相关应用都有Chrome版本，不会出现兼容性问题

#### Typora

![Typora](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020img2020typora-icon.png)

MarkDown文档书写工具，没错，这篇文档就是用它写的

完美支持所有md语法，可切换多种主题，具有多种编辑模式（源代码模式，专注模式，打字机模式），快捷键实现文本样式

支持目录[toc]，自动生成目录

支持picGo（后面会讲）图片上传，将图片复制到文档即可实现图片自动上传，同时自动获取链接并插入文档

#### picGo

![picGo](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020img2020image-20200304103159537.png)

图床上传工具，GitHub上开源

在本地md文档中插入图片时，链接一般是本地连接（图片的相对或觉得路径），线上编辑时链接一般为相关的网站提供（例如简书上编辑插入图片，获取的链接由简书提供）

所以将本地文档上传时，文档中的本地连接会失效，线上查看文档时图片无法加载，这时就需要使用图床，常用的图床有SM.MS，腾讯云COS，微博图床，GitHub图床，七牛图床，Imgur图床，阿里云OSS，又拍云图床

picGo支持上述所有图床（微博图床不再支持），用于将本地图片上传，并获取图片链接，同时嵌入了一个小的server，默认运行在本地36677端口上，方便在其他应用中直接上传图片，例如上面提到的Typora

自己现在用的是GitHub图床，唉，毕竟免费，就是图片加载速度慢，又是会加载不出来，可以使用jsDeliver提供的CDN服务（没错，又是免费:smirk:），提高加载速度

#### Ditto

![Ditto Icon](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020img2020icon.png)

剪切板增强工具

Windows自带的剪切板每次只能复制一条记录，无论记录大小，复制下一条，上一条就会从剪切板清除，再次使用就需要重新复制，如果由出现多个记录需要重复使用就会十分麻烦

Ditto解决了这个问题，它会自动获取每次复制的记录，存入数据库中，使用以前记录时只需要选择即可；可以设置存储记录最大条数，且记录不会因为电脑关机而清除；可以设置记录过期时间；也可以手动清除数据库记录；支持一次选择复制多个记录；

#### Free Download Manager

![Free Download Manager Lib](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020img2020logo.png)

免费的下载工具

可以代替迅雷的良心下载工具，支持普通下载，磁力链接，种子等下载方式，支持IE，Edge，Firefox，Chrome集成，接管浏览器的下载

Chrome有Free Download Manager extension插件，可以在Chrome应用商店中下载，其他浏览器不知道，Firefox应该也有相关插件

可以下载GitHub上的release版本发布（当初自己就是为了下载这个而安装的，github上下载东西是真滴慢）

#### PotPlayer

![PotPlayer](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020img2020potplayer-PotPlayer_logo_(2017).png)

视频播放器

内置解码器，支持多种格式的视频播放，无广告，界面清爽干净

和上面的Free Download Manager搭配使用

![img](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img20206a6a5eef1f3996b3)

#### Everything

文件搜索工具

![everything](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020everything-6863__everything_icon_converted.png)

文件搜索神器，体积小，速度快，支持正则语法，不过一般都是搜文件名

#### 火绒安全软件

![img](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020logo-1583302884776.png)

安全软件

这个没啥说的，和360相比，功能感觉不出来有啥差别，也可能是自己没好好用，但是胜在干净，没有广告，没有弹窗，还可以拦截其他应用的弹窗（自己感觉最好用的功能），至于360？它连自己的弹窗都拦不完

原本想着不装安全软件的，自己感觉系统自带的防火墙绝对够用，系统防火墙都拦不了的漏洞安全问题我不觉得安全软件就可以拦截得了（可能是安全软件病毒库，补丁什么的更新比官方及时吧），但老感觉电脑“裸奔”怪怪的

![img](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020e18d20c94006dfe0-0381536966d1161a-3a665d27faa0ebb89387bb25481a888a.jpg)

目前就到这儿了                                                                                  2020/3/4

---

#### Snipaste

微软提供的一个截图工具，压缩包，免安装，功能强大，截图+贴图

截图功能就不说了，快捷键`F1`

贴图功能，快捷键`F3`，可以把剪贴板的内容转化为图片置顶显示

- 剪贴板中的是图片，则贴出图像
- 剪贴板中的是文字，将文字转化为图像贴出
- 剪贴板中的是文件路径
  - 路径指示的文件为图片，则贴出图片；贴出图片后再次作贴图操作，则会贴出路径文本
  - 不是图片，则将路径按文本贴出

还有大量的高级操作，用不着，也没记住

![](https://gitee.com/yanzixian/picBed/raw/master/img202003/嘿嘿.png)

系统自带的截图工具也不错，但没有贴图功能，也没有历史截图的记录，关键是没怎么用过

2020/3/9

---

#### geek uninstaller

![](https://gitee.com/yanzixian/picBed/raw/master/img202003/20200327221842.png)

软件卸载程序

体积小（6M），免安装（就是一个.exe可执行文件），免费（也有收费版，功能多一点）；

卸载软件的同时，可以快速完全的清理相关的文件和注册表垃圾，卸载得十分干净

#### 7-zip

![](https://gitee.com/yanzixian/picBed/raw/master/img202003/20200327222511.png)

开源压缩、解压缩软件

体积小（1M），免安装（exe），支持多种格式（zip，rar，7z）

2020/3/27 更新

---

#### TranslucentTB

win10任务栏透明软件

可以设置使任务栏透明，Github开源，微软软件商店上也可以免费下载

安装后任务栏出现图标，单击图标出现如下设置窗口：

![image-20200330150327138](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200330150327138.png)

有Normal，Clear，Opaque，Blur，Fluent多种模式，透明，半透明等，Clear为全透明，其他可以自己试；

选择Open at boot开启开机自启

#### TranfficMonotor

> 这是一个用于显示当前网速、CPU及内存利用率的桌面悬浮窗软件，并支持任务栏显示，支持更换皮肤

Github上开源

![](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200330150944433.png)

2020/3/30

---

