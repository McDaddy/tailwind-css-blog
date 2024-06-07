---
title: 开发机环境设置
date: 2024-05-28
tags:
 - 前端工程化
lastmod: 2024-05-28
draft: false
summary: '主要针对开发机GO环境设置'

---



# 开发机初始化设置

开发机配置

- CPU:8 核

- 内存:16 GB

- 操作系统:Debian 10 linux 5.4



## Go环境配置

### 手动安装

以安装1.17版本为例，在[官方网站](https://go.dev/dl/)找到对应的下载链接

```shell
wget https://go.dev/dl/go1.17.13.linux-amd64.tar.gz
```

下载完成后解压，把Go解压到`/usr/local`目录下

```shell
sudo tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz
```

设置环境变量

```shell
echo export PATH=$PATH:/usr/local/go/bin | sudo tee -a /etc/profile #所有用户
echo export PATH=$PATH:/usr/local/go/bin >> ~/.profile #当前用户
```

使用source命令让环境变量生效

```shell
source ~/.profile
```

最后用go命令验证效果

```shell
go version
# go version go1.17.13 linux/amd64
```

### 自动安装

使用gvm进行安装

```shell
apt-get install bison # 可能需要的一个前置
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
export GOROOT_BOOTSTRAP=$GOROOT
source /root/.gvm/scripts/gvm
```

完成后就可以用命令来验证是不是装好

![image-20240314181044171](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240314181044171.png)

需要先安装一个go 1.4

```shell
gvm install go1.4 -B
```

结束之后再装需要的版本

```shell
gvm install go1.18
```

结束之后就可以验证，并使用`gvm use` 来切换版本

![image-20240314181239753](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240314181239753.png)

最后用`go version` 来验证下结果

```
gvm use go1.18 --default
```



### 安装gopls

安装gopls以及在vscode debug都需要go1.18及以上版本

如果是通过gvm安装的go可能是无法被自动识别的，需要手动在`.vscode/settings.json`里添加goroot的路径

```
{
    "go.goroot": "/home/mcdaddy/.gvm/gos/go1.18"
}
```

如果出现vscode debug无法进入断点的情况，可能是因为debug的路径其实是软链接，需要将路径替换成真实路径

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach to Process",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}",
            "args": [],
              "dlvFlags": ["--check-go-version=false"],
              "substitutePath": [
                {
                  "from": "/home",
                  "to": "/data00/home"
                }
              ]
        }
    ]
}
```





## zsh & oh-my-zsh安装

### 安装zsh

```shell
sudo apt-get install zsh
```

安装完成后可以通过命令查看支持的shell列表

```shell
cat /etc/shells
```

![image-20240312163125589](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240312163125589.png)

### 安装 oh-my-zsh

可以通过curl脚本安装

```shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

如果遇到无法下载github的问题，可以在**本机**启动一个代理服务（或者查找厂内官方代理），然后在devbox设置

```shell
export http_proxy=http://本机ip:port
export https_proxy=http://本机ip:port
```

自动安装就能得到

![image-20240312164730835](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240312164730835.png)

把path也写到zsh的设置里面去，否则上面的go命令也是找不到的

```shell
echo export PATH=$PATH:/usr/local/go/bin >> ~/.zshrc 
```

#### 设置主题

在`~/.oh-my-zsh/themes`目录下可以看到预设的主题，比如用`agnoster`这个主题，就只需要在`.zshrc`里面修改

```
ZSH_THEME="agnoster"
```

#### 安装插件

先cd到插件目录

```shell
cd ~/.oh-my-zsh/custom/plugins
```

zsh-autosuggestions 自动提示插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

zsh-syntax-highlighting 高亮插件

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting 
```

最后修改`.zshrc`

```shell
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

`source ~/.zshrc` 使修改生效