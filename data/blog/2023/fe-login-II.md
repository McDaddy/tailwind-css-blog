---
title: 前端登录知多少II
date: 2023-11-02
tags:
 - 前端工程化
lastmod: 2023-11-02
draft: false
summary: '当我们日常访问网页或者网站应用时，可以发现，除了日常的账号密码登录之外，往往还提供一种扫码登录的入口，例如淘宝、阿里云、飞书等等。那么本文就从原理层面来解析如何实现一个扫码登录的全流程'

---

# 前端登录知多少II

当我们日常访问网页或者网站应用时，可以发现，除了日常的账号密码登录之外，往往还提供一种扫码登录的入口，例如淘宝、阿里云、飞书等等。那么本文就从原理层面来解析如何实现一个扫码登录的全流程

![image-20231102165109037](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231102165109037.png)

![image-20231102165124116](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231102165124116.png)



## 扫码登录的概念

- 目的：免去用户去输入账号密码的流程，简化登录过程
- 原理：通过已经登录的手机端，来授权未登录的PC页面端直接登录
- 前提：扫码的手机端必须是已经登录的状态，否则手机端还是需要再登录一遍



## 场景

首先我们来分析一下扫码登录有哪些必要条件和必要步骤

- 二维码：登录页上的二维码，其背后的信息其实就是一个跳转的链接，让用户在手机扫码后可以自动跳转到一个登录确认页面

- 一般扫码动作都是发生在Web端产品对应的手机app上的，一般情况下不会出现用一个手机浏览器打开扫码连接，这样做有两个原因
  - 确保扫码的这一端是已经登录的，这样就免去了重新登录的步骤
  - 为了推广自己产品的手机客户端
- Web页面在被扫码之后，一般都会提示目前操作的状态，比如`已扫码/待确认`等状态信息，类似下图
  ![image-20231103121720736](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231103121720736.png)
- 在手机端扫码后，会跳转到一个**确认登录页**，类似下图
  <img src="https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/img_v3_024r_6a298aa8-d476-4522-bb55-163dded063eg.jpg" alt="img_v3_024r_6a298aa8-d476-4522-bb55-163dded063eg" style="zoom:25%;" />

- 点击**确认登录**之后，PC端页面就会跳转到登录成功的页面
- 此时PC端的登录态应该是持久化的，即使刷新页面也不会丢失这个登录状态

## 流程图

经过上面的分析，我们可以把主要的流程总结为下图

![image-20231103141127619](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231103141127619.png)

## 开发实现

### 生成二维码

1. 每次刷新二维码，都通过一个UUID来作为生成的key，并把它缓存在一个map中
2. 二维码中的信息，就是一个指向手机端确认页的链接，同时附带了uuid

```javascript
const map = new Map<string, QrCodeInfo>();

@Get('qrcode/generate')
async generate(@Req() request: Request) {
  const uuid = randomUUID();
  const dataUrl = await qrcode.toDataURL(
    `${request.protocol}://${request.headers.host}/pages/confirm.html?id=${uuid}`,
  );

  map.set(`qrcode_${uuid}`, {
    status: 'noscan',
  });

  return {
    qrcode_id: uuid,
    img: dataUrl,
  };
}
```

它的返回就是一个base64字符串可以用来显示图片以及uuid

![image-20231103145157955](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231103145157955.png)



### 二维码展示

写一个简单的页面，用来做PC端的登录页

1. 打开页面就请求`/qrcode/generate`接口，用来生成二维码
2. 得到接口返回渲染出二维码图片
3. 如果图片是未被扫码或扫码后还未确认的状态（即非**结束状态**），就需要定时轮询二维码当前的状态，并实时反馈到页面

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>扫码登录</title>
    <script src="https://unpkg.com/axios@1.5.0/dist/axios.min.js"></script>
  </head>
  <body>
    <div id="login">
      <img id="img" src="" alt="" />
      <div id="info" style="margin-left: 16px"></div>
    </div>
    <script>
      if (localStorage.getItem('xx-jwt-token')) {
        axios.get('/userInfo', {
          headers: { 'xx-jwt-token': localStorage.getItem('xx-jwt-token') },
        }).then(res => {
          if (res.data.username) {
            document.getElementById('content').style.visibility = 'visible';
            document.getElementById('login').style.display = 'none';
            document.getElementById('name').innerHTML = res.data.username;
          }
        });
      } else {
        // 打开页面就开始请求生成接口
        axios.get('/qrcode/generate').then((res) => {
          document.getElementById('img').src = res.data.img;
          queryStatus(res.data.qrcode_id);
        });
      }

      function queryStatus(id) {
        axios.get('/qrcode/check?id=' + id).then((res) => {
          const status = res.data.status;

          let content = '';
          switch (status) {
            case 'noscan':
              content = '未扫码';
              break;
            case 'scan-wait-confirm':
              content = '已扫码，等待确认';
              break;
            case 'scan-confirm':
              content = '已确认，即将跳转';
              break;
            case 'scan-cancel':
              content = '已取消';
              break;
          }
          document.getElementById('info').textContent = content;

          if (status === 'noscan' || status === 'scan-wait-confirm') {
            setTimeout(() => queryStatus(id), 1000);
          }
          if (status === 'scan-confirm') {
            window.localStorage.setItem('xx-jwt-token', res.data.token);
            // show app content
          }
        });
      }
    </script>
  </body>
</html>
```

样子差不多就是这样

![image-20231103150306074](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231103150306074.png)

### 实现轮询

这里需要定义好二维码的几个状态

- noscan 未扫描

- scan-wait-confirm -已扫描，等待用户确认

- scan-confirm 已扫描，用户同意授权

- scan-cancel 已扫描，用户取消授权

- expired 已过期

轮询仅在`未扫描`和`已扫描，等待用户确认`两个状态下发生，因为其他三个状态都是状态不可逆的状态，不需要再轮询

此接口接受一个uuid作为参数，返回它的当前状态

```javascript
@Get('qrcode/check') // 面向pc
async check(@Query('id') id: string) {
  const info = map.get(`qrcode_${id}`);
  if (info.status === 'scan-confirm') {
    return {
      token: await this.jwtService.sign({
        userId: info.userInfo.userId,
      }),
      ...info,
    };
  }
  return info;
}
```



### 登录确认页

当用户扫了上面的二维码之后，手机端就会跳转到一个确认登录的手机页。

它的地址是`http://localhost/pages/confirm.html?id={uuid}`

这个页面主要就是为了展示一个**确认登录**的按钮（假设目前是一个手机端已经登录的状态，就不去校验手机端是否登录了）

- 一旦打开这个页面，第一件事就是通知服务端，这个码已经被扫了，从而改变二维码的显示状态
- 如果点击了确认登录按钮，就通知服务端，这个码的登录已经被确认。此时手机端把自己的`jwt token`返回给服务端，即等于告诉服务端，是哪个登录的用户扫了码
- 如果点击了取消按钮，就通知服务端，这个扫码已取消

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>扫码登录确认</title>
    <script src="https://unpkg.com/axios@1.5.0/dist/axios.min.js"></script>
  </head>
  <body>
    <div id="login">
      <div id="info">是否确认登录 McDaddy 的假网站？</div>
      <button id="confirm">确认登录</button>
      <button id="cancel">取消</button>
    </div>

    <script>
      const params = new URLSearchParams(window.location.search.slice(1));

      const id = params.get('id'); // 就是uuid

      axios.get('/qrcode/scan?id=' + id).catch((e) => {
        alert('二维码已过期');
      });

      document.getElementById('confirm').addEventListener('click', async () => {
        const res = await axios.get('/login?username=admin') // 前端mock一个用户
        await axios.get('/qrcode/confirm?id=' + id, { headers: { 'xx-jwt-token': res.data.token }}).catch((e) => {
          alert('二维码已过期');
        });
      });

      document.getElementById('cancel').addEventListener('click', () => {
        axios.get('/qrcode/cancel?id=' + id).catch((e) => {
          alert('二维码已过期');
        });
      });
    </script>
  </body>
</html>
```



### 扫码动作

当手机端打开确认页自动调用，把二维码的状态置为`待确认`

```javascript
@Get('qrcode/scan')
async scan(@Query('id') id: string) {
  const info = map.get(`qrcode_${id}`);
  if (!info) {
    throw new BadRequestException('二维码已过期');
  }
  info.status = 'scan-wait-confirm';
  return 'success';
}
```

此时因为PC端会轮询，所以会变化状态

![image-20231103170414833](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20231103170414833.png)

### 确认动作

这里假设会在点击确认时，带上手机端的jwt token，即等于告诉服务端这个uuid对应的二维码被xx确认了，当然实际实现除了传jwt token肯定还有别的方式

- 这里会先验证下jwt token的有效性
- 更新二维码状态为`已确认`
- 把这个二维码的userInfo设置为当前登录用户

```javascript
@Get('qrcode/confirm') // 面向手机端
async confirm(
  @Query('id') id: string,
  @Headers('xx-jwt-token') auth: string,
) {
  let user;
  try {
    const info = await this.jwtService.verify(auth);

    user = this.users.find((item) => item.id == info.userId);
  } catch (e) {
    throw new UnauthorizedException('token 过期，请重新登录');
  }

  const info = map.get(`qrcode_${id}`);
  if (!info) {
    throw new BadRequestException('二维码已过期');
  }
  info.status = 'scan-confirm';
  info.userInfo = user;
  return 'success';
}
```



### 客户端跳转

这里就需要涉及到刚才轮询接口的状态判断功能，如果当前二维码已确认，就从map中把二维码对应的user信息拿出来，同时重新签名一个新的token，返回给PC端

```javascript
if (info.status === 'scan-confirm') {
    return {
      token: await this.jwtService.sign({
        userId: info.userInfo.userId,
      }),
      ...info,
    };
  }
```

此时的PC端，会在收到的轮询返回中得到一个额外的token字段，然后把它存到`localstorage`里面去，同时完成页面的跳转

```javascript
// index.html
if (status === 'scan-confirm') {
  window.localStorage.setItem('xx-jwt-token', res.data.token);
  // show app content
}
```



### 状态保持

此时再去刷新PC端页面时，因为本地已经存了token，此时会调用一个校验接口来校验jwt token是否有效，如果有效就正常显示页面。所以此时只有在本地删除这个token或者等待它过期，否则登录状态将一直保持！

```javascript
if (localStorage.getItem('xx-jwt-token')) {
    axios.get('/userInfo', {
      headers: { 'xx-jwt-token': localStorage.getItem('xx-jwt-token') },
    }).then(res => {
      if (res.data.username) {
        // show page
      }
    });
  }
```

