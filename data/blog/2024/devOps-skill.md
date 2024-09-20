## 网络相关



dig



ping



curl 



ifconfig

Tcpdump

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

## 进程相关

lsof





ps