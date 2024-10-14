## 网络相关



dig



### ping

![image-20240923133105561](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240923133105561.png)

- Icmp_seq: 使用的是icmp协议，这里就是序号。 ping本质就是icmp的echo的请求和响应
- time：表示从发送到返回的总时间
- ttl: 总共过了多少跳， linux初始64，这里说明总共过了 64 - 48 = 16 跳
- 64 bytes： 表示一个icmp数据包大小就是64字节， 其中56 data bytes表示内容是56字节， 8字节是icmp数据包的头





### curl 

```bash
# 非GET请求
curl -X -POST url
curl -XPOST url

# 添加Body
curl -X -POST url -d dataContent

# 添加Header
curl url -H 'lang: cn' -H 'cookie: xxx' # 可以添加多个

# 下载结果
curl -O url
curl -o a.txt url # 指定名称

# 查看返回码
curl -I url

# 跟随重定向
curl url -L

# 打印连接信息
curl -v url
```







ifconfig

### tcpdump

就是抓包命令，可以配合wireshark做分析



### netstat

需要安装net-tools这个软件包才能用

```
netstat -ntp | grep 8765
```

用来看本地网络端口的状态

参数：

- n:  以数字形式显示地址和端口 （如果不加那么可能显示的是主机名而不是IP）
- a:  显示所有
- t:  仅展示tcp连接
- p:  显示进程和进程号

```
home# netstat -ntp | grep 8766
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name 
tcp6       0      0 fdbd:dc61:9:463:9e:8766 fdbd:dc53:1:87::2:33226 ESTABLISHED 139/./shell_server  
tcp6       0      0 172.18.1.155:8766       10.121.74.137:46918     ESTABLISHED 139/./shell_server  
tcp6   22784      0 172.18.1.155:8766       10.213.37.69:43392      ESTABLISHED 139/./shell_server  
tcp6  241057      0 172.18.1.155:8766       10.189.51.208:56728     ESTABLISHED 139/./shell_server  
tcp6       0      0 172.18.1.155:8766       10.222.90.12:57186      ESTABLISHED 139/./shell_server  
tcp6  175939      0 fdbd:dc61:9:463:9e:8766 fdbd:dc53:22:812::33886 ESTABLISHED 139/./shell_server  
tcp6       0      0 172.18.1.155:8766       10.111.92.8:43360       ESTABLISHED 139/./shell_server  
```

其中

- Local Address 本地的地址和端口，可能是ipV4或者v6
- Foreign Address  远程地址
- State: 
  - ESTABLISHED表示已经建立连接
  - SYN-SENT 表示握手发出去了，但是对方没法返回，就是网络不通
  - LISTEN 表示这个这个端口正在监听传入的连接，就比如起了一个web服务等待调用
  - TIME-WAIT 表示连接关闭中，等待一段时间确保客户端断连





## 性能相关

### top

![image-20240920132052182](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240920132052182.png)

默认打开top就是这样的界面

逐行解析

![image-20240920132144619](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240920132144619.png)

第一行内容和我们输入另一个命令**uptime**效果是一样的

![image-20240920132330324](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240920132330324.png)

- 05:21:35表示机器当前的时间 
- up 87 days 表示这个系统运行了多少时间
- 0 user 表示当前登录用户数
- load average 表示系统负载，三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值 单位是%

![image-20240920132911714](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240920132911714.png)

第二行内容 表示进程信息

- 8 total 表示总进程数
- 1 running 表示正在运行数
- 7 sleeping 表示睡眠进程数
- 0 stopped 表示停止的进程数
- 0 zombie 表示僵尸进程数

![image-20240920133127375](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240920133127375.png)

第三行内容 表示CPU信息

- %Cpu(s): 35.6 us 表示**用户空间**占用CPU百分比
- 9.5 sy 表示**内核空间**占用CPU百分比 
- 0.1 ni 表示用户进程空间内改变过优先级的进程占用CPU百分比
- 51.1 id 表示空闲CPU百分比 
- 0.0 wa 表示等待输入输出的CPU时间百分比 

![image-20240920133529255](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240920133529255.png)

第四第五行内容 表示内存信息

- MiB Mem : 191810.2 total 表示物理内存总量
- 54540.3 free 表示空闲内存总量 
- 90993.6 used 表示已使用的物理内存总量 
- 46276.4 buff/cache 表示用作内核缓存的内存量



- MiB Swap: 0.0 total 表示交换区总量
- 0.0 free 表示 空闲交换区总量
- 0.0 used 表示已使用的交换区总量

![image-20240920134609137](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/image-20240920134609137.png)

后面就是进程信息

- PID－－>进程ID
- USER－－>用户
- PR－－>优先级
- NI－－>nice值。负值表示高优先级，正值表示低优先级
- VIRT－－>进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
- RES－－>进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
- SHR－－>共享内存大小，单位kb
- S－－>进程状态。 
  - D=不可中断的睡眠状态
  - R=运行 
  - S=睡眠
  - T=跟踪/停止
  - Z=僵尸进程
- %CPU－－>上次更新到现在的CPU时间占用百分比
- %MEM－－>进程使用的物理内存百分比
- TIME+－－>进程使用的CPU时间总计，单位1/100秒
- COMMAND－－>命令名/命令行

### 快捷键

- **m** 可以切换看内存的模式
  ![20240920135402_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240920135402_rec_.gif)
- **z** 可以彩色显示
- **e** 可以切换查看进程内存的单位 从kb -> mb -> gb -> tb -> pb
  ![20240920135650_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240920135650_rec_.gif)
- **shift + e** 可以切换内存的单位 （同上）
  ![20240920135926_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240920135926_rec_.gif)

- **o + 过滤条件 用来看指定的进程**
  ![20240920154848_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240920154848_rec_.gif)

- **k + PID** 直接杀掉指定进程
- **c** 切换COMMAND为完整指令
  ![20240920155148_rec_](https://kuimo-markdown-pic.oss-cn-hangzhou.aliyuncs.com/20240920155148_rec_.gif)

- **P**：按 CPU 使用率排序。
- **M**：按内存使用率排序。
- **f**: 设置任意字段来排序

### 其他参数

- -p 加上PID， 就只看某个进程



### free





## 进程相关

### lsof

如果本地一个端口发现夯住了，可以通过

```
lsof -i:8080
```

来查看端口占用情况

```
/home #  lsof -i:8080  
COMMAND     PID      USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Code\x20H  4427 bytedance  242u  IPv4 0x7a9e72d2be1aac57      0t0  TCP localhost:http-alt (LISTEN)
Code\x20H  4427 bytedance  243u  IPv4 0x742941a703a70434      0t0  TCP localhost:http-alt->localhost:63576 (ESTABLISHED)
node      58628 bytedance   44u  IPv4 0x6849cfa039fbc345      0t0  TCP *:http-alt (LISTEN)
node      98321 bytedance  150u  IPv4 0x7c4533a00a28b9e5      0t0  TCP localhost:63576->localhost:http-alt (ESTABLISHED)
```







ps