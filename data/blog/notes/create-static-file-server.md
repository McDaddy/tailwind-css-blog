---
title: 【笔记】- 如何搭建一个静态文件服务器
date: 2020-10-07
tags:
 - NodeJS
 - 网络
 - 笔记
lastmod: 2020-10-07
draft: false
summary: '如何搭建一个静态文件服务器'
---

# 如何搭建一个静态文件服务器



## 1. 创建一个http server

利用http模块，创建一个http server，接受外部传入的port参数用来启动监听

```javascript
  start() {
    const server = http.createServer(this.handleRequest.bind(this));
    server.listen(this.port, () => {
      console.log(
        `${chalk.yellow("Starting up zf-server:")} ./${path.relative(
          process.cwd(),
          this.directory
        )}`
      );
      console.log(`  http://localhost:${chalk.green(this.port)}`);
    });
  }
```


## 2. 编写handleRequest函数

核心点：

1. 接受request中的路径，拿出pathname，其实就是目标文件相对静态服务器根路径的相对路径
2. 使用`decodeURIComponent`解析url，防止url中有中文和特殊符号
3. 拼接得到目标文件在服务器中的的绝对路径
4. 用`fs.stat`判断文件路径是文件还是目录，如果不存在就返回404
5. 如果是文件进入发送文件的逻辑
6. 如果是目录，利用`fs.readdir`得到目录内容，然后遍历内容组成一个列表，将包含href和文件名的列表返回给ejs模板渲染，设置`Content-Type`为`text/html;charset=utf-8`表示返回的是一个HTML页面

```javascript
  async handleRequest(req, res) {
    let { pathname } = url.parse(req.url); // 获取路径
    pathname = decodeURIComponent(pathname); // 可能路径含有中文
    let filePath = path.join(this.directory, pathname); // 获取绝对路径
    try {
      // 判断是文件还是目录用state， 判断存不存在用access
      let statObj = await fs.stat(filePath);
      if (statObj.isFile()) {
        this.sendFile(req, res, statObj, filePath);
      } else {
        // 需要列出文件夹中内容
        let dirContents = await fs.readdir(filePath); // fs-extra
        // 文件访问的路径 采用绝对路径 尽量不要采用./ ../路径
        dirContents = dirContents.map((item) => ({
          // 当前根据文件名产生目录和href
          dir: item,
          href: path.join(pathname, item),
        }));
        let result = await ejs.render(
          this.template,
          { dirs: dirContents },
          { async: true }
        );
        res.setHeader("Content-Type", "text/html;charset=utf-8");
        res.end(result);
      }
    } catch (e) {
      this.sendError(req, res, e);
    }
  }
sendError(req, res, e) {
    res.statusCode = 404;
    res.end(`Not Found`);
 }
```



## 3. 编写返回文件内容的函数

核心点：

1. 判断是否有协商缓存，有则返回304
2. 通过mime三方模块判断文件尾缀来自动生成`Content-Type`
3. 通过管道pipe，将文件放在一个可读流然后流入response这个可写流

```javascript
  sendFile(req, res, statObj, filePath) {
    if (this.cache(req, res, statObj, filePath)) {
      res.statusCode = 304; // 协商缓存是包含首次访问的资源的
      return res.end();
    }
    // 发送文件 通过流的方式
    console.log("sending file...");

    res.setHeader("Content-Type", mime.getType(filePath) + ";charset=utf-8");
    createReadStream(filePath).pipe(res);
  }
```



## 4. 编写缓存逻辑

核心点：

1. `Expires`头 和 `Cache-Control=max-age=1000` 都是强制缓存，如果生效那么浏览器就不会请求服务器，会直接返回200然后显示`from memory/disk cache`。 Cache-Control的优先级大于Expires
2. 强制缓存的优先级是大于协商缓存的，如果强制缓存生效那么协商缓存条件符合也不会有效果
3. `Cache-Control=no-cache`表示开启协商缓存，每次都会向服务器发起请求
4. 强制缓存和协商缓存是可以配合使用的，一定时间内是强制缓存，到时间后发送请求给服务器判断文件是否变化，没有变化的话返回304，接下来一段时间依然是强制缓存。如果变化了就返回最新的文件，接下去一段时间依然是强制缓存
5. `if-modified-since`和`Last-Modified`, 浏览器发了if-modified-since头，就跟文件的最后修改时间`ctime`对比，如果不同就表示缓存失效。 永远将ctime赋给返回的`Last-Modified`。缺点：不能识别秒级的变化
6. `if-none-match`和`Etag`， 浏览器发了if-none-match头，就计算下目标文件的Etag，通过`md5`编码实现，相当于得到了文件的指纹摘要。如果跟计算出来的Etag不同，即缓存失效。永远将新计算的Etag赋给返回的Etag头。缺点：大文件计算量太大耗时
7. Etag的优先级大于Last-Modified

```javascript
  cache(req, res, statObj, filePath) {
    // 设置缓存， 默认强制缓存10s  10s内部不在像服务器发起请求 （首页不会被强制缓存） 引用的资源可以被强制缓存
    // res.setHeader("Expires", new Date(Date.now() + 10 * 1000).toGMTString());
    // no-cache 表示每次都像服务器发请求
    // no-store 表示浏览器不进行缓存
    res.setHeader("Cache-Control", "no-cache"); // http1.1新浏览器都识别cache-control
    // 过了10s “文件还是没变” 可以不用返回文件 告诉浏览器找缓存 缓存里就是最新的
    // 协商缓存 商量一下 是否需要给最新的如果不需要返回内容 直接给304状态码 表示找缓存即可

    // 默认先走强制缓存，10s 内不会发送请求到服务器中采用浏览器缓存，但是10s后在次发送请求。后端要进行对比 1) 文件没有变化 直接返回304 即可，浏览器会去缓存中查找文件。之后的10s中还是会走缓存 2)文件变化了返回最新的，之后的10s中还是会走缓存 不停的循环
    // 看文件是否变化

    // 1. 根据修改时间来判断文件是否修改了  **304**服务端设置
    let ifModifiedSince = req.headers["if-modified-since"];
    let ctime = statObj.ctime.toGMTString();
    let ifNoneMatch = req.headers["if-none-match"];
    let etag = crypto
      .createHash("md5")
      .update(readFileSync(filePath))
      .digest("base64");

    // 服务器设置好的
    res.setHeader("Last-Modified", ctime); // 缺陷如果文件没变 修改时间改了呢
    res.setHeader("Etag", etag);

    if (ifModifiedSince != ctime) {
      // 如果前端传递过来的最后修改时间和我的 ctime时间一样 ，文件没有被更改过
      return false;
    }
    if (ifNoneMatch !== etag) {
      // 可以用开头 加上总字节大小生产etag
      return false;
    }
    // 采用指纹Etag  - 根据文件产生一个唯一的标识 md5

    return true;
  }
```

## 5. 作为npm包发布

需要在`package.json`中加入bin属性

```json
  "bin": {
    "mhs": "./bin/server.js"
  },
```

入口文件加入commander的输入提示，包括要配置的端口号，地址等

```javascript
#! /usr/bin/env node

const program = require("commander");
const config = require("./serverConfig");
const Server = require("../src/index");
const { forEachObj } = require("../util");

program.name = "mhs";
forEachObj(config, (val) => {
  program.option(val.option, val.descriptor);
});

program.on("--help", () => {
  console.log("\nExamples:");
  forEachObj(config, (val) => {
    console.log("  " + val.usage);
  });
});

// --port 3000  --directory d:  --cache
program.parse(process.argv); // 没有这一步 即使是--help也是没法接收到的

const finalConfig = {}
forEachObj(config, (value, key) => {
    finalConfig[key] = program[key] || value.default
});


const server = new Server(finalConfig);

server.start();
```

在本地调试时，可以使用`npm link`来链接本地



## 知识点

1. `Cache-Control`不仅仅是服务器返回可以带的头，浏览器的请求也可以带
2. 浏览器带`Cache-Control:max-age=0`则强缓存失效，开始走协商
3. 当刷新操作时，强制缓存是**不生效**的，反推一下如果当刷新时强制刷新有效，那就直接实现离线访问了
4. 强制缓存只有在浏览器前进后退时有效果
5. `Cache-Control:max-age=0`与`Cache-Control:no-cache`的区别，max-age是告诉浏览器**应该**去做下缓存校验，而no-cache是必须做校验，所以max-age是有可能拿到过期的缓存的，而no-cache不会。`Cache-Control: max-age=0, must-revalidate`等价于no-cache

