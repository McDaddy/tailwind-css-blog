---
title: Debian包
date: 2025-01-18
tags:
 - OS
lastmod: 2025-08-28
draft: false
summary: '如何打一个debian包'

---



主要针对如何打一个符合官方标准的deb包，如果不是官方的会简单很多



## 命令

### dpkg

#### 安装：

```
apt install dpkg
```

#### 使用：

- 查看某个命令是由哪个软件包安装得来的

```shell
# 如想找下libtool是从哪里安装的
which libtool
# 得到/usr/bin/libtool

dpkg -S /usr/bin/libtool
# 得到 libtool-bin: /usr/bin/libtool
# 代表这个命令就是从libtool-bin来的
```

- 安装包

```shell
dpkg -i vco_1.0.0_amd64.deb
```





### apt

apt是`apt-get`和`apt-cache`等工具的结合体，优先使用

#### 使用：

- 查看一个包的信息：只列出这个包的信息，包括以下信息，但与有没有安装没关系

```shell
$ apt show libvterm-dev 
Package: libvterm-dev
Version: 0~bzr718-1
Priority: optional
Section: libdevel
Source: libvterm
Maintainer: James McCoy <jamessan@debian.org>
Installed-Size: 118 kB
Depends: libvterm0 (= 0~bzr718-1)
Homepage: http://www.leonerd.org.uk/code/libvterm/
Tag: devel::library, role::devel-lib
Download-Size: 33.2 kB
APT-Sources: http://mirrors.debian.org/debian buster/main amd64 Packages
Description: abstract terminal library (development files)
```

- 查看一个包是否被安装，同时是从哪个源来

```shell
$ apt policy libtool-bin # 或者apt-cache policy libtool-bin
libtool-bin:
  Installed: 2.4.6-9   # 表示已经安装的版本
  Candidate: 2.4.6-9
  Version table:
 *** 2.4.6-9 500
        500 http://mirrors.debian.org/debian buster/main amd64 Packages  # 500/100表示优先级，优先会从500下载
        100 /var/lib/dpkg/status
```

- 自动安装编译当前目录下软件源码所需的所有依赖包：`./` 表示当前目录，通常这个目录下应该包含软件的源码包，特别是需要有 `debian/control` 文件（该文件定义了编译此软件所需的依赖项）

```shell
apt build-dep ./       

# 在control文件中依赖写成这样
Build-Depends: debhelper (>=10), git, cmake, build-essential, dh-golang, golang-any (>= 2:1.23~), libtool-bin
```

- 移除包

```shell
apt remove libvterm # 只删除软件包本身，保留配置文件
sudo apt purge libvterm0 libvterm-dev # 会删除软件包及所有相关配置文件，适合彻底清理不再需要的软件
```

- 列出包

```shell
apt list --installed # 只列出已安装的包
apt list --upgradable # 只列出可升级的包
apt list <包名> # 搜索特定包（支持通配符，如 apt list python*）
```



### dpkg-buildpackage 

打包工具，是通过安装`dpkg-dev`这个包来的

```
dpkg-buildpackage -rfakeroot -us -uc -b
```

这个命令用来打包，其中-b表示只打二进制.deb文件，不打源代码包





## 打包准备与规范

### 目录要求

最重要的就是要在工程根目录下有一个debian目录，和几个必须的文件

```bash
# 和project同级目录获取编译结果
project/
##此目录执行apt build-dep ./
# 此目录执行dpkg-buildpackage -us -uc
├── a.c
├── b.c
├── debian
│   ├── changelog
│   ├── compat
│   ├── control
│   └── rules
└── updboot.sh

```

注意：最终打包出来的结果是在工程目录的上一层

### control

核心文件之一，记录这个包的**元信息**，类似包的身份证

```bash
Source: vco   # 包名和除非有多个包，否则和下面的Package一致
Section: utils  # 类型，代表包的分类
Priority: optional # 是不是系统必须装的包
Maintainer: mcdaddy <mcdaddychen@126.com>
Build-Depends: debhelper (>=10), git, cmake, build-essential, golang-any (>= 2:1.23~), libtool-bin # 编译依赖
Standards-Version: 4.6.2 # 写死

Package: vco
Architecture: any # 支持多架构
Depends: ${misc:Depends}
Description: what is it
 Long description starts here.
 This line is also part of long description.

```



### rules

可以理解为给debian工具读的Makefile，它里面包含了打包的整个生命周期，我们要做的就是在需要的节点写入自定义的步骤即可

```bash
#!/usr/bin/make -f

# 覆盖 dh_golang 的包检查步骤，跳过 dpkg-query 验证
override_dh_golang: # 仅执行必要的模块处理，不检查系统包
	true

%:
		dh $@ --with=golang  # 关键：通过 --with=golang 启用 dh-golang 工具链


# （可选）自定义编译参数（如指定输出文件名、LDFLAGS 等）
override_dh_auto_build:
		# 打印当前构建目录，确认是否在临时目录
		echo "===== 当前构建目录: $(shell pwd) ====="
		# 显式执行go build并输出详细错误
		echo "===== 开始编译 ====="
		go build -mod=mod -o vco

# 关键：将二进制文件复制到 debian 包临时目录
override_dh_auto_install:
		# 创建目标目录（对应系统的 /usr/bin/）
		install -d debian/vco/usr/bin/
		# 复制根目录的 vco 到临时目录（权限 755 确保可执行）
		install -m 755 vco debian/vco/usr/bin/

# 清理临时文件
override_dh_clean:
		dh_clean


override_dh_builddeb:
		# 执行你的最后操作（如删除临时依赖）
		rm -rf  vco  # 同时清理根目录的 vco
		# 执行默认的打包步骤
		dh_builddeb
```

