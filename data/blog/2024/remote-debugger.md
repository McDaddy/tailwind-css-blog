---
title: Goland远程调试指南
date: 2024-06-07
tags:
 - 前端工程化
lastmod: 2024-04-08
draft: false
summary: '一步步实操如何在本地用GoLand利用开发机远程调试'
---



在后端开发中，有许多情况本地启动的程序是无法访问测试或预发环境的资源（因为有网络的隔离），如果需要验证代码就需要把代码部署到开发机上启动验证，但这样就无法像本地开发一样一步步debug代码。

这里就以Goland为工具，实现下如何远程调试

>  前提：开发机上已经有完备的GO环境



# 连接开发机

一般情况下有三种方式可以连接开发机

1. Kerberos鉴权连接
1. PubKey连接
1. 密码连接

## Kerberos鉴权连接

就是用kinit来实现登录，登录成功后就可以用命令

```shell
ssh user@devbox_ip
```

来直接ssh登录自己的开发机

- 打开Goland => Tools => Deployment => Configuration
- 点击+号 => 选择SFTP => 起个名字
- 点击SSH Configuration旁的三个点`...`，进行ssh配置
- 同样点击+号 => 输入Host => port写22 => 填入用户名
- Authentication type选择`OpenSSH config and authentication agent`

![image-20240607164913257](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607164913257.png)

- 点击Test Connection，得到`Successfully connected!`就表示成功
- 点击Apply

这种连接有一个缺点就是Kerberos的权限是有时效的，一般最大24小时，也就是第二天来需要重新kinit才能续上权限，比较麻烦

## PubKey连接

可能是最普世的连接方法，通过公私钥匙做到类似github在本地的鉴权

- 在本机`~/.ssh`目录下运行
  ```shell
  ssh-keygen -t ed25519 -b 2048 -C "mcdaddychen@126.com"
  ```

  其中`-t ed25519`是加密算法，最后加上邮箱。建议给生成的文件取一个名字（在交互式中填入，这里是devbox_ed2219）

  ![image-20240607170623453](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607170623453.png)
  结束后目录会新增两个文件
  ![image-20240607170823382](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607170823382.png)

- 将公钥发送到开发机
  ```shell
  scp devbox_ed2219.pub chenweitao.mcdaddy@xxx.xx.xx.xx:~/.ssh/devbox_ed2219.pub
  ```

- 登录到开发机，执行
  ```shell
  cat your_rsa_file  > authorized_keys
  ```

- 检查机器的`PubkeyAuthentication`是否开启
  ```shell
  grep Pubkey /etc/ssh/sshd_config
  ```

  如果显示yes则不需要做什么，否则需用用root账号（sudo -i）用vim改下这个文件

- 打开Goland，步骤同Kerberos，直接快进到SSH Configuration。Authentication type选择`Key pair`，在Private key file写上面生成的另一个私钥文件的本地路径
  ![image-20240607171502331](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607171502331.png)

- 同样点击测试，通过即成功

## 密码连接

这个推荐度不高，遇到再补充



# 同步代码

这一步需要把我们本地的代码同步到开发机上，最终做到在开发机上编译启动

在上一步配置好SSH之后，就可以回到Deployment页，得到这样的界面
![image-20240607172143885](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607172143885.png)

点击Test Connection，得到`Successfully connected to xxx`即成功

这里我们创建一个Type为SFTP的Deployment就是为了把代码通过SFTP的方式传过去

- 在开发机上，单独建一个文件夹，用来存放当前项目的代码，比如在home目录下创建一个Project目录
- 在Deployment的配置界面，找到第二个Tab Mappings，在Deployment path里填写相对于上面`Root path`的路径
  ![image-20240607172754830](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607172754830.png)

- 在项目的根文件上右键 => Deployment => Upload to xxx
  ![image-20240607173109128](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607173109128.png)
  这步操作，就会把我们本地的代码直接都上传到开发机的指定目录下
- 结束后，在开发机的目录ls下验证下
- Tools => Deployment => 选择Automatic Upload   这样本地的所有编辑都会自动同步到开发机
  ![image-20240607173319072](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607173319072.png)

- 如果切换了分支，Goland是**不会**自动上传改动的代码的，需要重复上面的步骤，在根目录右键手动上传



# 安装dlv

开发机之所以可以在启动程序后被本地远程调试，就是依赖了dlv这个工具，类似于cdp之于Chrome。

在开发机上找一个目录
```shell
git clone https://github.com/go-delve/delve
cd delve
make install
```

用`dlv version`验证下安装成功

回到Goland，添加一个Run/Debug Configuration，选择**Go Remote**

填写远程机地址，port写3452（随意）

![image-20240607173957401](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240607173957401.png)





# 开始调试

## 开发机侧

cd到那个上传好代码的目录下，执行下面的命令

```
dlv debug --headless --api-version=2 --listen=:3452  /home/chenweitao.mcdaddy/projectA/*.go -- -args1=1 -args2=2
```

其中listen就是上面配置port，保持一致

`--`后面可以传启动参数

如果想设置启动时的**环境变量**，可以用这种写法

```
ENV_VAR1=123 \
ENV_VAR2=456 \
dlv debug --headless --api-version=2 --listen=:3452  /home/chenweitao.mcdaddy/projectA/*.go -- -args1=1 -args2=2
```

这样开发机就相当于用debug模式完成了go mod安装和**编译**，如果打印出如下内容，则表示启动成功，等待IDE的连接

```
API server listening at: [::]:3452
2024-06-07T17:45:42+08:00 warning layer=rpc Listening for remote connections (connections are not authenticated nor encrypted)
```

## IDE侧

直接选择刚才创建的那个Go remote作为目标，点击Debug（虫子）

观察下开发机的终端，如果有启动日志打印出来，则表示整个链路是通的

在任意代码中，可以像本地运行一样，打上断点

如果要测试接口，那就要在postman类似工具上，用开发机ip + 应用port来访问