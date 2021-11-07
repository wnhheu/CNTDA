# Lab1 - Wireshark

<!--more-->

这个很呆，CNTDA的lab好像要注册才可以下下来，我注册了一半觉得好像太繁琐了，然后查了一下好像还要书的序列号。。中文版说是没有的（毕竟这么便宜），然后去github上面找到了一个repository，非常不错，在此分享

[Resource](https://github.com/vacanthu/Computer-Networking-A-Top-Down-Approach-NOTES)

---

大体的内容这个resource里面都涵盖了，事实上这第一个Lab就只是安装wireshark这个软件而已，我不重复造轮子了，这个实验最后的几个问题也很简单，不多赘述了（才不是我写了一下午第一章笔记太累了）

# Lab2-HTTP

### 第一部分：简单HTTP的抓包和分析

就像名字一样，很简单，就是访问一个网站，然后简单的抓包、分析

![wireshark_1](https://github.com/wnhheu/Photos/blob/main/wireshark_1.png)

但是可以看到，这里在第一个GET请求之后，没有返回的包。。但是我的浏览器加载出了内容。。这就很神奇了

不过后面再重新尝试又可以接受到返回的包，个人认为是wireshark把服务器的返回包漏掉了，因为如果接受到了的话，即便有问题，也会返回400等等报错的状态值（参见第二部分的**进一步分析**）

### 第二部分：HTTP 长文件/嵌入网页的对象/加密的网站

![wireshark_2](https://github.com/wnhheu/Photos/blob/main/wireshark_2.png)

第一个是长文件，可以看到，这里可以看到一个很奇怪的内容**TCP Previous segment not captured**（不得不说，做这个作业会做出来好多奇怪的内容，看参考答案和自己去做完全不一样，因为网络还是一个颇为复杂的结构）

这里给出一个[Explanation](https://blog.csdn.net/weixin_34319374/article/details/91961180)大致解释了这种情况（不过我看不太懂这篇文章），按照我的理解就是说这里出现了包丢失的情况，但是考虑到这是TCP协议，丢失之后也会再次确认，或者是wireshark抓包过程中丢包，所以网页是可以正常打开的

---

![wireshark_4](https://github.com/wnhheu/Photos/blob/main/wireshark_4.png)

第二个是嵌入网页的对象，我们可以看到，No23的get命令之后没有收到服务器的回答，个人推测是不是因为在No19的get之后服务器就已经把这幅图片一起传过来了（因为在浏览器上这幅图片是和文字同时出现的）

然后还有一个很有意思（或者说和预期不一样）的是No65的get命令return了一个**301 Moved Permanetly**,这里需要注意的一点，No465的请求已经和我们初始的请求没有关系了（也可以看到一个的No是116，一个是465）

那么就很奇怪，这个图片去哪里了？我们没有收到服务器返回的文件

![wireshark_6](https://github.com/wnhheu/Photos/blob/main/wireshark_6.png)

这是对No116服务器返回信息的截图，我们可以看到重定向的地址是https://kurose.cslash.net/8E_cover_small.jpg

一个很重要的点！这里用的是https协议，不是http，所以没有在wireshark抓包中出现，因为我们设置的过滤值是http

#### 进一步分析

##### No23

![wireshark_8](https://github.com/wnhheu/Photos/blob/main/wireshark_8.png)

这里是取消了过滤器之后对No23的请求以及之后的响应的截图

我们可以看到，服务器返回的信息用的是TCP，而非HTTP，原因是[wireshark抓取的http协议包在数据传输过程中显示的是TCP协议，只有建立连接和断开连接那几个包才显示为http](https://zhidao.baidu.com/question/549190588.html) 这样就解释了上文提到的关于No23没有返回包的疑问

##### No116

对于No116来说，因为采用了https协议，所以后面的内容都是加密的

需要注意的是，https是http + ssl，但是ssl已经是过去式了，现在用的是TLS（我一开始在数据流里找https没有找到）

![wireshark_9](https://github.com/wnhheu/Photos/blob/main/wireshark_9.png)

这中间，No119-No124采用的LLMNR协议是域名解析协议，就是DNS域名解析，然后No128可以看到是一个TLSv1.2协议，这个可能就是我们向服务器请求照片的那个包，但是我不确定，可以看出来，Source IP和Target IP都是加密了的。。

不过我好歹能够解释之前遇到的问题了，网上关于这个作业的答案好像有一点不太靠谱

---

第三个是一个加密的网站，实际上还ok，没有遇到什么意料之外的问题

---

---

噢，还有一个很精妙的点，如果用了vpn之后，就看不到http了，好像大部分流量都用的是ssl，有一点复杂

