---
title: 后端也需要了解的浏览器技巧
date: 2023-11-16
tags:
 - 前端工程化
 - Chrome
lastmod: 2023-11-16
draft: false
summary: '浏览器的调试使用技巧主要还是针对前端的，但是有没有哪些技巧对后端同样有提效的作用？'

---

# 后端也需要了解的浏览器技巧

浏览器的调试使用技巧主要还是针对前端的，但是有没有哪些技巧对后端同样有提效的作用？这里就列举一些调试与使用的技巧，希望可以同样对后端同学提供帮助



## 调试技巧篇

### 获取完整请求信息

在我们通过前端页面调试后端接口时，一旦不符合预期，自然就想如何获得整个API请求的信息，用来自己后端调试

如果是Get请求，就比较简单，只需要把请求的链接复制出来即可

但如果我们遇到的是POST/PUT 等需要传递Body的请求，就比较麻烦，一种方式是把请求的Payload中的数据直接拷贝出来
![image-20231116172032410](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231116172032410.png)

但这样也仅仅只能拿到Body信息，如果请求还涉及Header/Cookie/Jwt/Content-Type等信息的话，获取这么点信息是完全不够的。

这时候，我们可以通过在API上右键菜单 => Copy => Copy as cURL来获取到一条curl语句

![image-20231116172401959](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231116172401959.png)

内容类似如下

```
curl 'https://qrcode-login.vercel.app/userInfo' \
  -H 'authority: qrcode-login.vercel.app' \
  -H 'accept: application/json, text/plain, */*' \
  -H 'accept-language: zh-CN,zh;q=0.9,en;q=0.8,ja;q=0.7' \
  -H 'if-none-match: W/"18-LVOPt/634EkE7QdMl1uVWEV8s5M"' \
  -H 'referer: https://qrcode-login.vercel.app/pages/index.html' \
  -H 'sec-ch-ua: "Google Chrome";v="119", "Chromium";v="119", "Not?A_Brand";v="24"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'sec-ch-ua-platform: "macOS"' \
  -H 'sec-fetch-dest: empty' \
  -H 'sec-fetch-mode: cors' \
  -H 'sec-fetch-site: same-origin' \
  -H 'user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36' \
  -H 'xx-jwt-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IumZiOmfoemfrCIsImlhdCI6MTY5OTI2MzQ5OX0.Nb0J_1kzM835_GgsvrhlC1Thuhlweg-PujtZ7bYt7nI' \
  --data-raw '[{"events":[]}]' \
  --compressed
```

我们可以直接把它粘贴到我们自己的Terminal里执行，效果和在浏览器重新发起一样。同时如果需要的话可以直接在上面修改各种参数，已实现用例的测试。这样做最重要的一点是我们不会遗漏请求中的任何细节信息，不会出现同样的入参在浏览器不正常，但在我本地是好的那种玄学情况



### 重试接口

那么如果我们在不想改参数的前提下，能不能直接在浏览器里面直接重新发起呢？

可以通过点击api上右键菜单 => Replay XHR来实现

![image-20231116174801564](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231116174801564.png)

点击之后，就会立刻发起一个相同请求体的API请求。这个场景特别适合花了很长时间配置了一个复杂的表单，点击提交结果报错了，修改后端接口后，又不想重新配置一次一样的表单，就可以用这个功能来简化操作



### 状态清除

我们在调试一些和状态有关的场景，例如用户登录状态、租户空间、多语言等，我们会希望把自己模拟成一个从未登录过的新用户。这种情况有一个非常实用的方法就是打开浏览器的**无痕模式**，在MAC下快捷键`cmd + shift + N`

![image-20231117105749161](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117105749161.png)

在这个模式下，浏览器不会去访问存在的Cookie，同样也不会去写Cookie等本地存储，可以真正做到无状态下使用应用

除此之外，再推荐一种更快捷的方式，不需要切换浏览器Tab，直接打开开发者工具 => Application

![image-20231117110122893](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117110122893.png)

这里列出了所有可能的浏览器存储（仅针对本域名），勾选之后点击`Clear site data`就可以清除所有相关的数据，这样就同样可以模拟一个新用户的行为



### 模拟弱网

有的时候我们需要测试一下应用的一些边界性能，比如在弱网环境的响应。其实我们就可以利用浏览器来直接模拟

打开Network => 点击这个WiFi的图标 => Network throttling

![image-20231117110712605](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117110712605.png)

通过选择预设的弱网条件`Fast 3G/Slow 3G/Offline`来实现实时模拟，如果这些还不符合我们的需要还可以进行进一步自定义，设置网络的上下行限制，以及延迟

![image-20231117111243818](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117111243818.png)



### Network使用技巧

#### 聚焦目标请求

首先要确保Network中这个漏斗图标是打开状态的，这样才能看到过滤功能的工具栏

![image-20231117111551706](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117111551706.png)

可以看到浏览器自动帮我们把网络请求进行了分类

- All： 即全部，不进行过滤
- Doc：可以理解为下载HTML，但这个不是绝对的，网页的第一个请求不论它返回是什么类型的，都会归为Doc
- JS：就是下载我们的`.js`文件
- CSS：下载`.css`文件
- Font：下载`.woff2`等格式的字体文件
- Img：下载各种图片
- WS：websocket

这里重点介绍下`Fetch/XHR`， fetch和xhr简单理解就是前端实现Ajax的两种原生方式，虽然现在我们写前端接口请求基本都不太可能用这些原生api，而是使用各种三方封装的请求库，但底层的调用是跳不出这两样的。

我们常说的API调用都是在这个类型下，所以一般调试接口的时候可以直接把类型聚焦在`Fetch/XHR`这样可以过滤绝大多数的非接口请求。

但即使这样我们会发现还是有着大量自己不关心的请求存在，这里主要有两个原因

1. 可能会发现一些类似下载js的请求也在这个类别的，原因是这个类型记录的是fetch和xhr发起的请求，而它们除了能请求接口外也是可以请求静态资源的，所以只要是通过JavaScript发起的请求都会在这里
2. 我们的应用可能有配置埋点，就会有各种日志上报的接口，这个往往不可控

所以为了进一步过滤目标请求，我们可以利用Filter功能

**关键字过滤**

![image-20231117114523090](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117114523090.png)

**条件过滤**

- larger-than
- status-code
- mixed-content:all
- scheme:http
- domain

![image-20231117115030447](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117115030447.png)

![image-20231117115055287](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117115055287.png)

当然也可以把两种或多种条件混合过滤

![image-20231117115248517](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117115248517.png)

#### 查看请求信息

建议大家打开，Network面板设置中的`Big Request Row`选项。它可以显示更多的信息，下面是打开和未打开的对比

![image-20231117161051876](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117161051876.png)

![image-20231117161158131](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117161158131.png)

Network的每一行默认只展示了有限的信息，我们可以右键任意一个头，菜单里可以选择我们关注的信息，并直接显示在列表里

![image-20231117115646405](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117115646405.png)

这里推荐把**Time**这个列打开，这样我们可以直观地看到接口的响应速度

![image-20231117152824276](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117152824276.png)

同时可以配合Waterfall这列来分析接口的总体使用时间，这里会包括接口发起的排队时间，服务端响应时间以及下载时间等

![image-20231117152952983](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117152952983.png)



>- Queueing
>
>  . The browser queues requests before connection start and when:
>
>  - There are higher priority requests.
>  - There are already six TCP connections open for this origin, which is the limit. Applies to HTTP/1.0 and HTTP/1.1 only.
>  - The browser is briefly allocating space in the disk cache.
>
>- **Stalled**. The request could be stalled after connection start for any of the reasons described in **Queueing**.
>
>- **DNS Lookup**. The browser is resolving the request's IP address.
>
>- **Initial connection**. The browser is establishing a connection, including TCP handshakes/retries and negotiating an SSL.
>
>- **Proxy negotiation**. The browser is negotiating the request with a [proxy server](https://en.wikipedia.org/wiki/Proxy_server).
>
>- **Request sent**. The request is being sent.
>
>- **ServiceWorker Preparation**. The browser is starting up the service worker.
>
>- **Request to ServiceWorker**. The request is being sent to the service worker.
>
>- **Waiting (TTFB)**. The browser is waiting for the first byte of a response. TTFB stands for Time To First Byte. This timing includes 1 round trip of latency and the time the server took to prepare the response.
>
>- **Content Download**. The browser is receiving the response, either directly from the network or from a service worker. This value is the total amount of time spent reading the response body. Larger than expected values could indicate a slow network, or that the browser is busy performing other work which delays the response from being read.
>
>- **Receiving Push**. The browser is receiving data for this response via HTTP/2 Server Push.
>
>- **Reading Push**. The browser is reading the local data previously received.



### 模拟返回

考虑以下场景

- 有时候想要测试前端的一些边界情况，比如空值，正负值，数据类型等
- 验证前端崩溃原因但因为是线上环境不能随便改数据。

这是如果通过改后端代码（或者找符合条件的数据）来测试代价就有点大。我们可以利用浏览器开发工具自带的功能来做模拟

打开开发者工具 => Network => 选择想要模拟的记录右键 => Override content

![Choosing override content from the right-click menu of a request.](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/qOixESNMKApJdBaw0Kcy.png)

如果是第一次使用此功能，需要设置一个本地目录来存储覆盖的数据

![DevTools prompts you to select a folder.](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/oEWmbPqoEmv7uOF1KwAg.png)

设置目录完成后，页面的顶端会弹出一个确认框，询问是否允许覆盖，点击Allow

![DevTools requests access.](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/RkMW3bOZAroc83TujxBq.png)

这时候再重复第一步的Override content，就会引导到一个编辑的界面。具体效果如下

![20231117154700_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20231117154700_rec_.gif)

通过这种方法，我们不仅可以任意修改返回结果，还能修改Response Header信息。在纯前端的应用上甚至可以把本地编译好的代码覆盖上去验证Bug修复效果

如果想关闭这个功能，把下图的选择框反选即可

![image-20231117155058848](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117155058848.png)



## 插件篇

就是Chrome浏览器的扩展程序，虽然有很大的数量级，但特别有用的不是特别多，这里我推荐几个常用的，欢迎补充

[广告终结者](https://chromewebstore.google.com/detail/%E5%B9%BF%E5%91%8A%E7%BB%88%E7%BB%93%E8%80%85/fpdnjdlbdmifoocedhkighhlbchbiikl?pli=1)

[Google翻译](https://chrome.google.com/webstore/detail/aapbdbdomjkkjkaonfhkkikfgjllcleb)

[FeHelper(前端助手)](https://chrome.google.com/webstore/detail/pkgccpejnmalmdinmhkkfafefagiiiad)：功能强大的前端助手

[篡改猴](https://chrome.google.com/webstore/detail/dhdgffkkebhmkfjojejmpbldmpobfkfo)：也称为油猴，通过它可以引申出另一个插件系统。

这里也推荐几个油猴脚本，欢迎补充

![image-20231117173045664](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231117173045664.png)



### SwitchyOmega

[SwitchyOmega](https://chrome.google.com/webstore/detail/padekgcemlokbadohgkifijomclgjgif)：为浏览器设置代理服务器的一个插件，大多数人的使用场景都是用来设置科学上网，但就开发人员来说，可以通过搭配[whistle](https://wproxy.org/whistle/)实现比浏览器开发工具更强大的网络调试功能

whistle是一个基于Node实现的跨平台web调试工具，有点类似Fiddler，最终呈现为一个可以劫持请求的HTTP代理服务器，我们需要手动在本地打开这个服务，默认启动在8899这个端口

![image-20231204103256008](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204103256008.png)

打开SwitchyOmega的设置页，添加一个场景，设置代理服务器的地址就是我们的127.0.0.1:8899

![image-20231204121914581](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204121914581.png)

当需要代理时，选择whistle这个代理，不需要时就选择**系统代理**

![image-20231204122041723](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204122041723.png)



假设遇到如下场景：

场景前提，前端代码是由本地服务器比如(localhost:8080)提供的，后端api是由具体环境的服务提供的（比如http://baidu.com）

#### 接口跨域

浏览器在A域名访问B域名，如果B服务的返回没有开启CORS的话，那会出现类似如下的报错

![image-20231204163301515](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204163301515.png)

这种情况一般发生在我们的本地开发环境，因为我们用的一般是localhost这样的域名，后端是不应该给这种域名开白的，而实际的线上环境域名往往是已经开了白名单的。所以这种情况就需要我们前端自己借助代理服务器来解决

打开`http://127.0.0.1:8899/`，配置whistle服务，在**Rules**这个tab增加一条规则`*.baidu.com resCors://enable`

![image-20231204163742118](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204163742118.png)

就这么简单的一句话，就能轻松解决这个跨域问题



#### 批量劫持请求/返回

我们在浏览器里其实也可以做到劫持接口，但是是有一些局限的

1. 只能劫持返回，不能劫持请求
2. 不能做到批量劫持，比如所有/api/xx的接口都自动加一个`xxx: 123`的header
3. 做一些精确劫持，比如把所有返回中文本中的`哈哈`字符串变成`嘻嘻`

在Rules中接入如下的规则，reqHeaders表示替换请求的Header，resHeaders表示替换返回的Header

![image-20231204164509916](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204164509916.png)

在**Values**Tab里添加一个叫fakeHeader的Item，内容如下

![image-20231204164714120](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204164714120.png)

此时刷新Baidu的页面，就会发现这三个Header已经在工具中显示了

![image-20231204164934185](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204164934185.png)

但我们仔细观察，发现这三个Header**并没**出现在Request中，难道是劫持没成功？

这里就要提到第一个Network Tab，它可以抓起到从浏览器发起的**所有经过代理的请求**。我们过滤下baidu.com，可以发现在这里不管请求还是返回都有这三个Header

![image-20231204165235468](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204165235468.png)

这说明，Chrome自带的Network工具，只能劫持到从自己这儿发出去的请求，而事实上，这个请求在透过浏览器本身后又经过了一层代理服务器，那时才把这个Header加上，所以这步浏览器是无法感知的

![image-20231204165654852](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231204165654852.png)



同理，whistle也能劫持改变各种返回内容，这里就不一一例举了，总的来说whistle可以帮助我们批量地完成对请求返回的劫持，在遇到一些复杂场景时会比浏览器更加高效



## 参考

[Network features reference](https://developer.chrome.com/docs/devtools/network/reference/?utm_source=devtools)

[用 Chrome 的人都需要知道的「神器」扩展：「油猴」使用详解](https://sspai.com/post/40485)

[whistle](https://wproxy.org/whistle/)