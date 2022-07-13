## 1、计时方式

### 1.1、古代计时方式

在远古时期，人类用来确定时间的方式是一些自然界“相对”亘古不变的周期。如地球的公转是为一年，月球的公转是为一月，地球的自转是为一天等，最早的计时可以追溯到公元前大约2000年，古埃及人利用光线留下的影子用作计时的工具。影子拉得越长，计时越精确。古埃及人修建高耸入云的大型方尖碑，来追踪太阳的移动，随后人们又利用了沙漏、日晷、钟摆等工具，巧妙地利用一些相对固定而准确的周期来计时

商朝人开发并使用了一种泄水型水钟——漏壶。后来又有用蜡烛和线香计时的

北宋元祐元年（1086年），天文学家苏颂将浑仪、浑象和报时装置结合，建造一个划时代的计时工具——“水运仪象台”

14世纪时，西方国家广泛使用机械钟。在十六世纪，奥斯曼帝国的科学家达兹·艾-丁（Taqi al-Din）发明出了机械闹钟

1583年，伽利略提出了著名的等时性理论，即不论摆动幅度的大小，完成一次摆动的时间是相同的。1656年，荷兰科学家克里斯蒂安·惠更斯（Christiaan Huygens）应用他的理论，设计出了世界第一只钟摆

1868年，百达翡丽（Patek Philippe）发明了手表

### 1.2、现代计时方式

石英晶体受到电池的电力影响时，会产生规律的振动。每秒的振动次数是32768次，可以设计电路来计算振动次数，当计数到32768次时，即计时1秒。1967年，瑞士人发布了世界上首款石英表

当原子从一个相对高的“能量态”迁至低的“能量态”时，会释放出电磁波，产生共振频率。依据此原理，拉比构想出了一种全新的计时仪器——原子钟（Atomic clock）

因为原子的共振频率是固定的。如：铯原子（Caesium133）的固有频率是9192631770赫兹，约合92亿赫兹，对铯原子钟计数9192631770次，即可测量出一秒钟。很多国家（包括我国和美国NIST）的标准局，就是用铯原子钟来作为时间精度标准的。GPS系统也是用铯原子钟来计时

2008年诞生的锶（Strontium87）原子钟，固有频率为429228004229873，约合430万亿赫兹，将精度提高到了10的17次方

2013年镱元素（ytterbium）制成的原子钟问世，镱原子钟的固有频率约合518万亿赫兹，精度高达10的18次方。宇宙的年龄为138亿年。如果这台镱原子钟从宇宙诞生之初就开始计时，直到今天也不会发生1秒的误差

## 2、网络时间协议

### 2.1、时间同步和chrony

时间同步：多主机协作工作时，各个主机的时间同步很重要，时间不一致会造成很多重要应用的故障，如：加密协议，日志，集群等， 利用NTP（Network Time Protocol）
协议使网络中的各个计算机时间达到同步。目前NTP协议属于运维基础架构中必备的基本服务之一

时间同步实现：ntp，chrony

ntp：将系统时钟和世界协调时UTC同步，精度在局域网内可达0.1ms，在互联网上绝大多数的地方精度可以达到1-50ms，项目官网：http://www.ntp.org

chrony：实现NTP协议的的自由软件。可使系统时钟与NTP服务器，参考时钟（例如GPS接收器）以及使用手表和键盘的手动输入进行同步。还可以作为NTPv4（RFC
5905）服务器和对等体运行，为网络中的计算机提供时间服务。设计用于在各种条件下良好运行，包括间歇性和高度拥挤的网络连接，温度变化（计算机时钟对温度敏感），以及不能连续运行或在虚拟机上运行的系统。通过Internet同步的两台机器之间的典型精度在几毫秒之内，在LAN上，精度通常为几十微秒。利用硬件时间戳或硬件参考时钟，可实现亚微秒的精度

### 2.2、chrony

#### 2.2.1、chrony 的优势：

- 更快的同步只需要数分钟而非数小时时间，从而最大程度减少了时间和频率误差，对于并非全天 24 小时运行的虚拟计算机而言非常有用
- 能够更好地响应时钟频率的快速变化，对于具备不稳定时钟的虚拟机或导致时钟频率发生变化的节能技术而言非常有用
- 在初始同步后，它不会停止时钟，以防对需要系统时间保持单调的应用程序造成影响
- 在应对临时非对称延迟时（例如，在大规模下载造成链接饱和时）提供了更好的稳定性
- 无需对服务器进行定期轮询，因此具备间歇性网络连接的系统仍然可以快速同步时钟

chrony官网：https://chrony.tuxfamily.org

chrony官方文档：https://chrony.tuxfamily.org/documentation.html

#### 2.2.2、chrony组成

包：chrony

两个主要程序：chronyd和chronyc

- chronyd：后台运行的守护进程，用于调整内核中运行的系统时钟和时钟服务器同步。它确定计算机增减时间的比率，并对此进行补偿
- chronyc：命令行用户工具，用于监控性能并进行多样化的配置。它可以在chronyd实例控制的计算机上工作，也可在一台不同的远程计算机上工作

服务unit 文件： /usr/lib/systemd/system/chronyd.service

监听端口： 323/udp，123/udp

配置文件： /etc/chrony.conf

#### 2.2.3、配置文件chrony.conf

- server - 可用于时钟服务器，iburst 选项当服务器可达时，发送一个八个数据包而不是通常的一个数据包。 包间隔通常为2秒,可加快初始同步速度
- driftfile - 根据实际时间计算出计算机增减时间的比率，将它记录到一个文件中，会在重启后为系统时钟作出补偿
- rtcsync - 启用内核模式，系统时间每11分钟会拷贝到实时时钟（RTC）
- allow / deny - 指定一台主机、子网，或者网络以允许或拒绝访问本服务器
- cmdallow / cmddeny - 可以指定哪台主机可以通过chronyd使用控制命令
- bindcmdaddress - 允许chronyd监听哪个接口来接收由chronyc执行的命令
- makestep -
  通常chronyd将根据需求通过减慢或加速时钟，使得系统逐步纠正所有时间偏差。在某些特定情况下，系统时钟可能会漂移过快，导致该调整过程消耗很长的时间来纠正系统时钟。该指令强制chronyd在调整期大于某个阀值时调整系统时钟
- local stratum 10 - 即使server指令中时间服务器不可用，也允许将本地时间作为标准时间授时给其它客户端

配置文件示例

```
[root@ali ~]# cat /etc/chrony.conf
# Use Alibaba NTP server
# Public NTP
# Alicloud NTP


server ntp.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp.aliyun.com minpoll 4 maxpoll 10 iburst
server ntp1.aliyun.com minpoll 4 maxpoll 10 iburst
server ntp1.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp10.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp11.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp12.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp2.aliyun.com minpoll 4 maxpoll 10 iburst
server ntp2.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp3.aliyun.com minpoll 4 maxpoll 10 iburst
server ntp3.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp4.aliyun.com minpoll 4 maxpoll 10 iburst
server ntp4.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp5.aliyun.com minpoll 4 maxpoll 10 iburst
server ntp5.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp6.aliyun.com minpoll 4 maxpoll 10 iburst
server ntp6.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp7.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp8.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst
server ntp9.cloud.aliyuncs.com minpoll 4 maxpoll 10 iburst

# Ignore stratum in source selection.
stratumweight 0.05

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Enable kernel RTC synchronization.
rtcsync

# In first three updates step the system clock instead of slew
# if the adjustment is larger than 10 seconds.
makestep 10 3

# Allow NTP client access from local network.
#allow 192.168/16

# Listen for commands only on localhost.
bindcmdaddress 127.0.0.1
bindcmdaddress ::1

# Disable logging of client accesses.
noclientlog

# Send a message to syslog if a clock adjustment is larger than 0.5 seconds.
logchange 0.5

logdir /var/log/chrony
#log measurements statistics tracking
```

#### 2.2.4、chronyc命令

- help命令可以查看更多chronyc的交互命令
- accheck 检查是否对特定主机可访问当前服务器
- activity 显示有多少NTP源在线/离线
- sources [-v] 显示当前时间源的同步信息
- sourcestats [-v]显示当前时间源的同步统计信息
- add server 手动添加一台新的NTP服务器
- clients 报告已访问本服务器的客户端列表
- delete 手动移除NTP服务器或对等服务器
- settime 手动设置守护进程时间
- sracking 显示系统时间信息

#### 2.2.5、chronyc示例

```
[root@centos7 ~]#chronyc
chrony version 3.5
Copyright (C) 1997-2003, 2007, 2009-2019 Richard P. Curnow and others
chrony comes with ABSOLUTELY NO WARRANTY.  This is free software, and
you are welcome to redistribute it under certain conditions.  See the
GNU General Public License version 2 for details.
chronyc> clients
Hostname NTP Drop Int IntL Last Cmd Drop Int Last
===============================================================================
192.168.8.7 18 0 6 - 17 0 0 - -
192.168.8.6 14 0 6 - 56 0 0 - -
chronyc> activity
200 OK
1 sources online
0 sources offline
0 sources doing burst (return to online)
0 sources doing burst (return to offline)
0 sources with unknown address
chronyc> sources -v
210 Number of sources = 1
.-- Source mode '^' = server, '=' = peer, '#' = local clock.
/ .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| / '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
|| .- xxxx [ yyyy ] +/- zzzz
|| Reachability register (octal) -. | xxxx = adjusted offset,
|| Log2(Polling interval) --. | | yyyy = measured offset,
|| \ | | zzzz = estimated error.
|| | | \
MS Name/IP address Stratum Poll Reach LastRx Last sample
===============================================================================
^? 192.168.8.100 3 8 1 338 -40ms[ -40ms] +/- 237ms
```

### 2.3、公共NTP服务

pool.ntp.org：项目是一个提供可靠易用的NTP服务的虚拟集群cn.pool.ntp.org，0-3.cn.pool.ntp.org

阿里云公共NTP服务器

- Unix/linux类：ntp.aliyun.com，ntp1-7.aliyun.com
- windows类： time.pool.aliyun.com

大学ntp服务

- s1a.time.edu.cn 北京邮电大学
- s1b.time.edu.cn 清华大学
- s1c.time.edu.cn 北京大学

国家授时中心服务器

- 210.72.145.44

## 3、时间相关工具

timedatectl

- 查看日期时间、时区及NTP状态：timedatectl
- 查看时区列表：timedatectl list-timezones
- 修改时区：timedatectl set-timezone Asia/Shanghai
- 修改日期时间：timedatectl set-time "2017-01-23 10:30:00"
- 开启NTP： timedatectl set-ntp true/flase

system-config-date：图形化配置chrony服务的工具

