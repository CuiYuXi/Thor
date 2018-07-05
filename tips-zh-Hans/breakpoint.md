### 利用 Thor HTTP 抓包的断点调试功能调试 WebView

##### 0x1、要解决的问题

工作中时常遇到需要对自己已上线 app 中的 WebView 网页进行一些调试验证，以排除 bug，解决问题。

##### 0x2、解决方案

利用 Thor HTTP 抓包的『断点调试』功能向 WebView 注入调试脚本。

##### 0x3、技术原理

向目标网页的 HTML 响应中注入调试脚本（文本替换）。

##### 0x4、WebView 调试示例 app

[PPHub For Github](https://itunes.apple.com/app/id1314212521)

##### 0x5、用到的工具

- [Thor HTTP Sniffer](https://itunes.apple.com/app/id1210562295): 抓包及断点调试

- [vConsole](https://github.com/Tencent/vConsole): A lightweight, extendable front-end developer tool for mobile web page.

- [Eruda](https://github.com/liriliri/eruda): Console for mobile browsers.


#### 第一步：在 Thor 中设置过滤器断点

##### 0x1、打开 Thor, 创建一个过滤器，取名为："WebView 注入调试"，并添加一个断点进入编辑


##### 0x2、因为需要对 WebView 的响应 HTML 内容进行注入，所以选择在 `请求阶段 > 响应消息体回传前 > 新建匹配规则`

![](bp_res/4.jpg) ![](bp_res/8.jpg)

##### 0x3、在匹配规则中加入表达式进行替换，以注入 vConsole 工具

**因为对 HTML body 标签注入 js 脚本后可能不会总是生效，所以这里选择优先对 title 标签进行替换**

加入判断条件：判断是否包含 title 标签
``` javascript
@rsp.bodyText CONTAINS[cd] "</title>"
```

当有 title 标签时，执行以下表达式

``` javascript
^@rsp.bodyText "<\/title>" "</title><script type='text/javascript' src='https://coding.net/u/Tumblr/p/thor-lib/git/raw/master/vconsole/3.2.0/vconsole.min.js'></script><script>new VConsole();</script>"
```

没有 title 标签，则找 body 标签替换

``` javascript
^@rsp.bodyText "<\/body>" "<script type='text/javascript' src='https://coding.net/u/Tumblr/p/thor-lib/git/raw/master/vconsole/3.2.0/vconsole.min.js'></script><script>new VConsole();</script></body>"
```

- *考虑加载速度的原因，没有直接使用 github 上的 vConsole 原地址，而是 dump 了一份到 coding.net 仓库里，所以上面用的是 codeing.net 的脚本地址*

![](bp_res/7.jpg)


同理，再新建一个断点，完成 Eruda 工具的注入（其实可以不用两个调试脚本都注入，这里只是演示效果）


#### 第二步：在 PPHub 中找一个 WebView 界面进行尝试

##### 0x1、打开 Thor > 选择 "WebView 注入调试" 过滤器，并启动

![](bp_res/9.jpg) ![](bp_res/3.jpg) 

##### 0x2、打开 PPHub 并找到一个 WebView, 等待加载完成

![](bp_res/2.jpg) ![](bp_res/1.jpg) 
![](bp_res/6.jpg) ![](bp_res/5.jpg) 


#### 下面是 Thor 的 20 天免费试用 TestFlight 申请地址和详细介绍

0x1、[Thor 的详细介绍](https://github.com/PixelCyber/Thor/blob/master/README-zh-Hans.md)

0x2、[TestFlight 申请问卷填写](https://wj.qq.com/s/1607760/e57d)

0x3、过滤器 "WebView 注入调试" 后期优化版 [下载](bp_res/WebView_debug.f4thor)（下载后直接用 Thor 打开，安装）
