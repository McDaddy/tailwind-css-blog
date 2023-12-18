---
title: 你需要知道的Cookie
date: 2023-11-07
tags:
 - 前端工程化
lastmod: 2023-11-07
draft: false
summary: 'Cookie是什么？它有什么作用？'
---

# 你需要知道的Cookie

我们在Web开发中，免不了会和浏览器中的Cookie打交道，今天我们就来彻底理解下Cookie



## Cookie介绍

在维基百科中Cookie的定义是

> Cookie（复数形态Cookies），类型为「小型文本文件」，指某些网站为了辨别用户身份而储存在用户本地终端上的数据。

一般情况下，浏览器存储cookie的容量上限是4KB，所以说它是个小型文件，而它的最主要作用就是用于识别客户端上的用户身份

### 为什么会有Cookie

一般我们都会说 “HTTP 是一个无状态的协议”，而所谓无状态协议，简单的理解就是即使同一个客户端连续两次发送请求给服务器，服务器也识别不出这是同一个客户端发送的请求。这样的话用户所有的操作状态都无法保持

所以为了解决这个问题，Cookie应运而生。服务端与客户端通过传递Cookie实现了通讯状态的保持（非HTTP协议的保持）



它的工作方式如下图

![image-20231107174033694](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107174033694.png)

最核心的优势在于它会随着HTTP请求自动附带到请求头中，免去开发者额外设置的工作量

### Cookie查看

可以通过打开浏览器开发者工具，选择`Application` Tab

![image-20231107175130252](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107175130252.png)

此时打开的是在当前域名下的所有Cookie，其实Cookie是存储在我们操作系统上的一个具体文件，比如Mac，Cookie文件就存储在`~/Library/Application Support/Google/Chrome/Default/Cookies`这个文件中，这是一个二进制数据库文件不能直接打开，但可以通过下载一个SQLite软件来打开

![image-20231107192914058](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107192914058.png)

这样就能看到本机上的所有Cookie信息了

### Cookie的属性

#### Name/Value

就是Cookie的键值对，唯一要注意的是这些字符串中不能包含分号，逗号等特殊符号，所以在设置前需要做一次编码处理

#### Expires/Max-Age

表示Cookie过期的时间

![image-20231107195618451](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107195618451.png)

如果在设置Cookie时不添加任何过期信息，这就是一个会话Cookie，在浏览器关闭后它也随之失效



**Expires** 用于设置 Cookie 的过期时间。

这里的时间表示是GMT即格林威治时间，比如我们可以在http请求的返回中用`set-cookie`头来设置客户端的cookie

```
Set-Cookie: mcdaddy=chenweitao; Expires=22 Dec 2024 07:28:00 GMT;
```

如果我们是在客户端操作，则可以直接用JavaScript进行操作，在控制台中输入（假设接下来的操作都在百度主页发生）

```javascript
document.cookie = 'mcdaddy=chenweitao; Expires=22 Dec 2024 07:28:00 GMT;'
```

就会得到这样一条Cookie

![image-20231107200653756](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107200653756.png)

上面的这个Cookie就会在时间到达GMT，2024年12月22日7点28分时失效



**Max-Age**也是设置过期时间的，它和Expires的区别是它的具体时间是在客户端计算，它的值是一个数字，单位是**秒**

```javascript
document.cookie = 'mcdaddy=chenweitao; Max-Age=120'
```

这里最终得到的结果就是一个120秒后会失效的cookie

![image-20231107201215383](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107201215383.png)

所以其实和Expire没有本质区别，好处是不会设置出一个过去时间的Cookie。

如果这个值设置为0或者负数，这个Cookie会立即失效

如果设置为负数，就是成为一个会话cookie



如果同时设置Expires和Max-Age，Max-Age的优先级更高



#### Domain

Domain 指定了 Cookie 可以送达的主机名。假如没有指定，那么默认值为当前文档访问地址中的主机部分（但是不包含子域名）。

如上面我们设置的Cookie，在不设置Domain的情况下，就是完整的域名`www.baidu.com`

但实际上在使用cookie的场景都会设置成根域名的形式，即`.baidu.com`，因为子域名是可以复用根域名的Cookie

即访问`tieba.baidu.com`会自动带上在`.baidu.com`的Cookie，但是反过来则不行

同时，Cookie是不能跨域设置的，下面的设置是不会产生效果的

```javascript
document.cookie = 'mcdaddy=chenweitao; Domain=.taobao.com;'
```



#### Path

Path 指定了一个 URL 路径，这个路径必须出现在要请求的资源的路径中才可以发送 Cookie 首部。比如设置 `Path=/docs`，`/docs/Web/` 下的资源会带 Cookie ，`/test` 则不会携带 Cookie 。

这个在实际中应用比较少，一般都设置成`/`



#### Secure属性

标记为 Secure 的 Cookie 只应通过被HTTPS协议加密过的请求发送给服务端。使用 HTTPS 安全协议，可以保护 Cookie 在浏览器和 Web 服务器间的传输过程中不被窃取和篡改。



#### HTTPOnly

设置 HTTPOnly 属性可以防止客户端脚本通过 document.cookie 等方式访问 Cookie，防止攻击脚本读取敏感Cookie信息。



#### SameSite

这个属性我们重点展开，在说明这个属性前，我们先了解下什么是一方Cookie什么是三方Cookie



## 一方 vs 三方Cookie

假设我们访问A网站，通过对它的请求，在返回的header中`set-cookie`，这样产生的就是**一方Cookie**，这是我们最熟悉的cookie形式，它从属于当前访问的A网站，Domain也是A网站的域名

同时我们在访问A的网站中，访问到了B域名的一个接口，这个接口在它的返回中同样set cookie，这样就会出现一个从属于A网站，但是Domain是这个第三方的域名的Cookie。这就是**三方Cookie**

![image-20231107212359620](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107212359620.png)

那么三方Cookie有什么作用呢？目前最主要的使用场景就是**广告推荐**

假设你这在逛优酷，在搜索栏搜索`手机`，观看相关的视频，过一段时间后，打开天猫或者淘宝，会惊喜的发现首页就会推给你手机的广告。这是怎么做到的呢？

主要分以下几部分：

1. 当访问优酷时，会发现网站的Cookie里有一些属于`mmstat.com`的三方Cookie，这是因为运行了mmstat的脚本

脚本主动访问了mmstat的接口服务，同时在返回中被set了Cookie

![image-20231108101529603](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231108101529603.png)

![image-20231107213234265](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107213234265.png)

2. 这个脚本会无时无刻得记录我们的操作，同时上报到mmstat这个域名，然后被服务端分析我们的购物行为倾向

![image-20231107213658844](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231107213658844.png)

3. 像淘宝这类应用应该会定期执行大数据任务（个人猜想），分析这些用户行为，并产生相应的推荐内容
4. 当我们打开天猫或者淘宝，会发现在这些域名下，发现依然有mmstat的Cookie存在，并且通过对比发现，在两个网站中，mmstat的K-V是完全一样的

![image-20231108101946411](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231108101946411.png)

4. 这就使得淘宝知道一个信息，即刚才在优酷上进行搜索`手机`操作的人和当前打开淘宝的人是同一个用户，这时候淘宝就会从预先大数据任务中产生结果里依据cookie信息得到推荐反馈
   至于淘宝是怎么做关联的，个人猜想是，打开淘宝后，mmstat向淘宝后端发送了请求，把cookie的信息发送了过去，然后在用户请求淘宝的接口时，淘宝把相同的cookie注到自己的域名下，证据就是两个domain有一个相同kv的cookie
   ![image-20231108115959294](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231108115959294.png)
5. 同时这个信息是多方共享的，推荐也是双向的，即我们在淘宝搜索`手机`再去优酷，也可能会看到手机相关的推荐视频

除此之外，基于三方Cookie的特性我们还能实现

- 前端行为埋点日志上报
- 两个网站间的免登录，比如在淘宝登录后打开天猫就是直接登录的状态



## SameSite

好了，我们说回SameSite，首先SameSite直译就是**同站**，那同站是什么意思呢？

具有相同 eTLD+1 的网站被视为 “同站”。具有不同 eTLD+1 的网站是 “**跨站**”。其中eTLD是`effective top-level domain`的缩写

![image-20231108145309174](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231108145309174.png)

具体的例子参考下图

![image-20231108145442073](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231108145442073.png)

这里我们所说的跨站，就对应于我们上面提到的**三方**



#### PSL

但这里会有一些奇怪的例子，比如`a.github.io`和`b.github.io`是跨站的，这里就是要涉及上面`eTLD`中的e了。一般在我们概念中的顶级域名比如`.com`,`.net`等等也就几十个，这里面肯定是不包含`.github.com`这种商业域名的。

这里引出一个新的知识**PSL**，全称是**Public Suffix List**，它就是一个域名列表，维护在[这个地址](https://publicsuffix.org/list/public_suffix_list.dat)，目前里面已经有上万条记录且可能每天都在增长。这里面维护了两类域名

- ICANN提供的TLD：ICANN（The Internet Corporation for Assigned Names and Numbers，互联网名称与数字地址分配机构），这里提供的都是我们熟悉的那些顶级域名，例如最常见的 com、net、org 等
- PRIVATE列表：是由个人或机构自行添加的，比如我们个人注册了一个域名`abc.cn`，想要把二级域名开放给别人用，比如`x.abc.cn`和`y.abc.cn`，这两个域名是给两个“外人”使用的，所以就害怕这个外人在使用这个域名的时候在`.abc.cn`这个一级域名里面去注入自己的Cookie，这样就有整个域名的Cookie被篡改的风险，所以这个列表就支持个人或机构自行去注册，把自己的域名注册成一个`effective`的TLD，本质就是为了做到域名的用户隔离





SameSite有三种可选值

#### Strict

`Strict` 是最严格的防护，将阻止浏览器在所有跨站点浏览上下文中将 `Cookie` 发送到目标站点。因此这种设置可以阻止所有 `CSRF` 攻击。然而，它的用户友好性太差，即使是普通的 `GET` 请求它也不允许通过。

例如，对于一个普通的站点，这意味着如果一个已经登录的用户跟踪一个发布在公司讨论论坛或电子邮件上的网站链接，这个站点将不会收到 `Cookie` ，用户访问该站点还需要重新登陆。



#### Lax

也是SameSite目前的默认值，对于允许用户从外部链接到达本站并使用已有会话的网站站，默认的 `Lax` 值在安全性和可用性之间提供了合理的平衡。 `Lax` 属性只会在使用危险 `HTTP` 方法发送跨域 `Cookie` 的时候进行阻止，例如 `POST` 方式。同时，使用 `JavaScript` 脚本发起的请求也无法携带 `Cookie`。

例如，一个用户在 A 站点 点击了一个 B 站点（GET请求），而假如 B 站点 使用了`Samesite-cookies=Lax`，那么用户可以正常登录 B 站点。相对地，如果用户在 A 站点提交了一个表单到 B站点（POST请求），那么用户的请求将被阻止，因为浏览器不允许使用 `POST` 方式将 `Cookie` 从A域发送到Ｂ域。

#### 

#### None

就是没有限制



这个属性是从Chrome浏览器v80开始推的，Chrome 计划从 2024 年第一季度开始对 1% 的用户禁用第三方 Cookie。将来迟早会全面实施

![image-20231108171822564](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231108171822564.png)

目前我们打开世面上绝大多数的网站，打开开发者工具中的`Issues`面板，都会看到下面这样一个warning

![image-20231109105957867](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231109105957867.png)



所以现在我们在访问各种网站是经常看到如下的提示，问我们是否接受所有Cookies，如果我们选择`Accept All Cookies`那就意味着运行这个网站跟踪我们的浏览行为。事实上目前大多数网站都是没有这样的提示的，让我们点击接受并不是他们无法设置三方cookie而是为了规避法律风险

![image-20231108144243101](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231108144243101.png)

目前在Chrome浏览器的设置中，我们也可以自主设置针对三方Cookie的策略

![image-20231110151221085](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231110151221085.png)



总结一下，在接下来的时间里，个人隐私会越来越受到重视，各大厂商想自由无条件获取用户的行为信息会变得越来越难。至于如果绕过三方Cookie的方法，由于篇幅原因就不在这里展开了



## 总结

Cookie的本质作用，不论是一方还是三方，都是用来标记用户身份的，由于这个特性它可以帮助我们完成诸如

- 用户登录态的保持
- SSO
- 用户行为日志分析
- 广告推荐

但是浏览器厂商对三方Cookie的限制越来越严格，在不远的将来三方Cookie将逐渐淡出我们的视野



## 参考

[当浏览器全面禁用三方 Cookie](https://juejin.cn/post/6844904128557105166?searchId=20231107170715391155229F92950D6F66#heading-1)

[域名小知识：Public Suffix List](https://imququ.com/post/domain-public-suffix-list.html)