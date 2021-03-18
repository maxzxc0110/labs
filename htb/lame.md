<div class="container">

<div class="post"># lame

<div class="show-content">**难易程度：容易** 昨天心血来潮在htb（hack the box）充了个会员，（htb注册需要自己hack一个邀请码，这个比较简单，网上也很多教程，这里就不说了），纸上得来终觉浅，感觉还是得多实践。 这是我在htb做的第一台 靶机，看评价比较简单（你，你，长得最矮的那个，出来！），先用nmap扫一下

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-2cdace9e713239fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<div class="image-caption">nmap 结果图</div>

</div>

结果分别21,22,139,445几个端口。vsftpd2.3.4有一个比较著名的ftp后门，几乎每个渗透测试的教程都会讲一遍这个后门，于是在msf里找到这个攻击负载，很快啊，一下子就出来了！

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-ae9522efb76666b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

加载负荷，设置目标IP：

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-d8723e6900c629b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

然而失败了！

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-61577b9740a680b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

这个让我纠结了蛮久，以为是我本地环境的问题导致攻击失败。因为我是在kali虚拟机上做的攻击，我一开始还以为是因为没有外网ip，所以没有办法反弹shell，于是又去前两年双十一趁便宜买的那台阿里云服务器上试了下，还是一样的结果。后来google了一下，发现payload设置的ip应该是htb分配的vpn里的那个：

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-d656654bbf75ceac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

这就坑了，我还专门去问了玩kali的一些朋友，有些让我搞一台外网的vps做攻击机，有点让我做一个内网转发。但其实这些都复杂了，玩htb本机payload的ip就填这个10.10.14.30就行了。当然这个ip每个人分配的都不一样，你得看一下你本机的。这个也怪我之前没有先看一下htb的搞机教程。。。 ftp这个后门搞不了，那就试试ssh有啥，搜了一下好像也没发现有啥合适的。那就next！ 剩下只有samba，谷歌了一下这个版本的漏洞：

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-e964962780da7854.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

选择第一个搜索结果：

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-31c120969f59d578.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

在msf里搜索这个漏洞，发现有现成的：

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-01be9e4cb7a896c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

选择攻击载荷，发动攻击：

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-94b2887736261ee7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

成功getshell，输入whoami，发现还是root权限。拿flag走人：

<div class="image-package">![](http://upload-images.jianshu.io/upload_images/9177635-8d17a65989db911f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)</div>

</div>

</div>

</div>