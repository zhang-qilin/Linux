## 1、高可用集群概念

集群类型：

```
LB lvs/nginx（http/upstream, stream/upstream）
HA 高可用性
  SPoF: Single Point of Failure，单点故障
HPC高性能集群(High Performance Computing)
```

https://www.top500.org

系统可用性：

```
SLA(Service-Level Agreement)
   95%=(60*24*30)*(1-0.9995)
  （指标）=99%, ..., 99.999%，99.9999%
```

系统故障：

```
硬件故障：设计缺陷、wear out（损耗）、自然灾害……
软件故障：设计缺陷
```

提升系统高用性的解决方案之降低MTTR(平均故障时间)

```
解决方案：建立冗余机制
  active/passive主/备
  active/active双主
  active --> HEARTBEAT --> passive
  active <--> HEARTBEAT <--> active
```

高可用的是“服务”

```
HA nginx service：
  vip/nginx process[/shared storage]

资源：组成一个高可用服务的“组件”
  (1) passive node的数量
  (2) 资源切换
```

shared storage：

```
NAS(Network Attached Storage)：网络附加存储，基于网络的共享文件系统。
SAN(Storage Area Network)：存储区域网络，基于网络的块级别的共享
```

Network partition：网络分区

```
quorum：法定人数
  with quorum：> total/2
  without quorum: <= total/2

隔离设备：fence
  node：STONITH = Shooting The Other Node In The Head(强制下线/断电)
  https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/s1-unfence-haar
```

双节点集群(TWO nodes Cluster)

```
Failover：故障切换，即某资源的主节点故障时，将资源转移至其它节点的操作
Failback：故障移回，即某资源的主节点故障后重新修改上线后，将之前已转移至其它节点的资源重新切回的过程
  辅助设备：ping node, quorum disk(仲裁设备)
```

HA Cluster实现方案:

```
AIS(ApplicaitonInterface Specification)应用程序接口规范
  RHCS：Red Hat Cluster Suite红帽集群套件
  heartbeat：基于心跳监测实现服务高可用
  pacemaker+corosync：资源管理与故障转移
  
vrrp(Virtual Router Redundancy Protocol)：虚拟路由冗余协议,解决静态网关单点风险
  软件层—keepalived
  物理层—路由器、三层交换机
```

高可用集群-后端存储

![keepalived_01.png](./images/30-keepalived/keepalived_01.png)

JBOD （Just a Bunch Of Disks ）不是标准的RAID 等级，它通常用来表示一个没有控制软件提供协调控制的磁盘集合，JBOD 将多个物理磁盘串联起来，提供一个巨大的逻辑磁盘，JBOD
的数据存放机制是由第一块磁盘开始按顺序往后存储，当前磁盘存储空间用完后，再依次往后面的磁盘存储数据，JBOD 存储性能完全等同于单块磁盘，而且也不提供数据安全保护，它只是简单提供一种扩展存储空间的机制，JBOD
可用存储容量等于所有成员磁盘的存储空间之和。

![keepalived_02.png](./images/30-keepalived/keepalived_02.png)

## 2、Keepalived简介

vrrp协议的软件实现，原生设计目的为了高可用ipvs服务

### 2.1、功能：

```
基于vrrp协议完成地址流动
为vip地址所在的节点生成ipvs规则(在配置文件中预先定义)
为ipvs集群的各RS做健康状态检测
基于脚本调用接口通过执行脚本完成脚本中定义的功能，进而影响集群事务，以此支持nginx、haproxy等服务
```

### 2.2、VRRP-网络层实现

![keepalived_03.png](./images/30-keepalived/keepalived_03.png)

### 2.3、组件：

```
用户空间核心组件：
  vrrpstack-VIP消息通告
  checkers-监测real server
  system call-标记real server权重
  SMTP-邮件组件
  ipvs wrapper-生成IPVS规则
  Netlink Reflector-网络接口
  WatchDog-监控进程
控制组件：配置文件分析器
IO复用器
内存管理组件
```

![keepalived_04.png](./images/30-keepalived/keepalived_04.gif)

http://keepalived.org/documentation.html

### 2.4、术语：

```
虚拟路由器：Virtual Router
虚拟路由器标识：VRID(0-255)，唯一标识虚拟路由器
物理路由器：
  master：主设备
  backup：备用设备
  priority：优先级
VIP：Virtual IP
VMAC：VirutalMAC (00-00-5e-00-01-VRID)
通告：心跳，优先级等；周期性
工作方式：抢占式，非抢占式
安全工作：
    认证：
      无认证
      简单字符认证：预共享密钥
工作模式：
  主/备：单虚拟路由器
  主/主：主/备（虚拟路由器1），备/主（虚拟路由器2）
```

## 3、keepAlived配置

Keepalived环境准备

```
各节点时间必须同步
关闭selinux和防火墙
```

### 3.1、Keepalived安装

```
# yum install keepalived(CentOS)
# apt-get install keepalived(Ubuntu)
```

### 3.2、程序环境：

```
主配置文件：/etc/keepalived/keepalived.conf
主程序文件：/usr/sbin/keepalived
Unit File：
  /usr/lib/systemd/system/keepalived.service(CentOS)
  /lib/systemd/system/keepalived.service(Ubuntu)
Unit File的环境配置文件：
  /etc/sysconfig/keepalived
```

### 3.3、配置文件组件部分：

```
GLOBAL CONFIGURATION
  Global definitions

VRRP CONFIGURATION
  VRRP instance(s)：即一个vrrp虚拟路由器

LVS CONFIGURATION
  Virtual server group(s)
  Virtual server(s)：ipvs集群的vs和rs
```

### 3.4、配置语法：

```
配置虚拟路由器：
vrrp_instance <STRING> {
    ....
}

配置参数：
  state MASTER|BACKUP：当前节点在此虚拟路由器上的初始状态，状态为MASTER或者BACKUP
  interface IFACE_NAME：绑定为当前虚拟路由器使用的物理接口ens32,eth0,bond0,br0
  virtual_router_id VRID：当前虚拟路由器惟一标识，范围是0-255
  priority 100：当前物理节点在此虚拟路由器中的优先级；范围1-254
  advert_int 1：vrrp通告的时间间隔，默认1s

authentication { #认证机制
    auth_type AH|PASS
    auth_pass <PASSWORD> 仅前8位有效
}

virtual_ipaddress { #虚拟IP
    <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
    192.168.200.17/24 dev ens160
    192.168.200.18/24 dev ens224 label ens224:1
}

track_interface { #配置监控网络接口，一旦出现故障，则转为FAULT状态实现地址转移
    eth0
    eth1
    …
}
```

### 3.5、组播配置示例

#### 3.5.1、组播配置示例-MASTER：

```
global_defs{
   notification_email{
     root@zsjshao.com #keepalived发生故障切换时邮件发送的对象，可以按行区分写多个
   }
   notification_email_from keepalived@zsjshao.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id c81.zsjshao.com
   vrrp_skip_check_adv_addr #所有报文都检查比较消耗性能，此配置为如果收到的报文和上一个报文是同一个路由器则跳过检查报文中的源地址
   vrrp_strict #严格遵守VRRP协议,不允许状况:1,没有VIP地址,2.单播邻居,3.在VRRP版本2中有IPv6地址.
   vrrp_garp_interval 0 #ARP报文发送延迟
   vrrp_gna_interval 0 #消息发送延迟
   vrrp_mcast_group4 224.0.0.18 #默认组播IP地址，224.0.0.0到239.255.255.255
   vrrp_iptables #不自动生成iptables规则
}

vrrp_instance VI_1 {
    state MASTER
    interface ens160
    virtual_router_id 80
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.200 dev ens160
    }
}
```

#### 3.5.2、组播配置示例-BACKUP：

```
global_defs {
   notification_email {
     root@zsjshao.com
   }
   notification_email_from keepalived@zsjshao.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id c82.zsjshao.com
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_iptables
}
 
vrrp_instance VI_1 {
    state BACKUP
    interface ens160
    virtual_router_id 80
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.200 dev ens160
    }
}

```

VIP测试

```
# iptables -D INPUT -s 0.0.0.0/0 -d 192.168.7.248 -j DROP #yum安装会自动生成防火墙策略，可以删除或禁止生成
```

```
[root@c81 ~]# tcpdump -i ens160 -nn host 224.0.0.18
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), capture size 262144 bytes
16:14:02.066561 IP 172.16.0.129 > 224.0.0.18: VRRPv2, Advertisement, vrid 80, prio 100, authtype none, intvl 1s, length 20
16:14:03.066942 IP 172.16.0.129 > 224.0.0.18: VRRPv2, Advertisement, vrid 80, prio 100, authtype none, intvl 1s, length 20

[root@c81 ~]# ping 172.16.0.200
PING 172.16.0.200 (172.16.0.200) 56(84) bytes of data.
64 bytes from 172.16.0.200: icmp_seq=1 ttl=64 time=0.026 ms
64 bytes from 172.16.0.200: icmp_seq=2 ttl=64 time=0.030 ms
64 bytes from 172.16.0.200: icmp_seq=3 ttl=64 time=0.029 ms
```

### 3.6、VIP单播配置及示例

```
global_defs {
   notification_email {
     root@zsjshao.com
   }
   notification_email_from keepalived@zsjshao.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id c81.zsjshao.com
   vrrp_skip_check_adv_addr
#   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
   vrrp_iptables
}
 
vrrp_instance VI_1 {
    state MASTER
    interface ens160
    virtual_router_id 80
    priority 100
    advert_int 1
    unicast_src_ip 172.16.0.129 #本机源IP
    unicast_peer {
        172.16.0.130 #目标主机IP
    }
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.200 dev ens160
    }
}
```

```
[root@c81 ~]# tcpdump -i ens160 -nn host 172.16.0.130
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), capture size 262144 bytes
16:24:25.053784 IP 172.16.0.129 > 172.16.0.130: VRRPv2, Advertisement, vrid 80, prio 100, authtype simple, intvl 1s, length 20
16:24:26.054157 IP 172.16.0.129 > 172.16.0.130: VRRPv2, Advertisement, vrid 80, prio 100, authtype simple, intvl 1s, length 20
16:24:27.055258 IP 172.16.0.129 > 172.16.0.130: VRRPv2, Advertisement, vrid 80, prio 100, authtype simple, intvl 1s, length 2
```

### 3.7、非抢占

```
nopreempt：定义工作模式为非抢占模式,需要VIP state都为BACKUP
preempt_delay 300：抢占式模式，节点上线后触发新选举操作的延迟时长，默认模式

vrrp_instance VI_1 {
#    state MASTER
    state BACKUP
    interface ens160
    virtual_router_id 80
    priority 100
    advert_int 1
    nopreempt
    unicast_src_ip 172.16.0.129
    unicast_peer{
        172.16.0.130
    }
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.200 dev ens160
    }
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160
    virtual_router_id 80
    priority 80
    advert_int 1
    nopreempt
    unicast_src_ip 172.16.0.130
    unicast_peer {
        172.16.0.129
    }
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.200 dev ens160
    }
}
```

### 3.8、Keepalivde 双主配置

两个或以上VIP分别运行在不同的keepalived服务器，以实现服务器并行提供web访问的目的，提高服务器资源利用率。

```
vrrp_instance VI_1 {
    state MASTER
    interface ens160
    virtual_router_id 80
    priority 100
    advert_int 1
    unicast_src_ip 172.16.0.129
    unicast_peer{
        172.16.0.130
    }
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.200 dev ens160
    }
}
 
vrrp_instance VI_2 {
    state BACKUP
    interface ens160
    virtual_router_id 81
    priority 80
    advert_int 1
    unicast_src_ip 172.16.0.129
    unicast_peer{
        172.16.0.130
    }
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.201 dev ens160
    }
}
```

### 3.9、Keepalived通知配置

发件人配置:

```
[root@c81 ~]# yum install mailx -y
[root@c81 ~]# vim /etc/mail.rc
set from=634802317@qq.com
set smtp=smtp.qq.com
set smtp-auth-user=634802317@qq.com
set smtp-auth-password=pcqizacnzzktbefe
set smtp-auth=login
set ssl-verify=ignore
```

```
定义通知脚本：
notify_master <STRING>|<QUOTED-STRING>：
  当前节点成为主节点时触发的脚本

notify_backup <STRING>|<QUOTED-STRING>：
  当前节点转为备节点时触发的脚本

notify_fault <STRING>|<QUOTED-STRING>：
  当前节点转为“失败”状态时触发的脚本

notify <STRING>|<QUOTED-STRING>：
  通用格式的通知触发机制，一个脚本可完成以上三种状态的转换时的通知
```

Keepalived通知脚本

```
[root@c81 ~]# cat /etc/keepalived/notify.sh
#!/bin/bash
contact='634802317@qq.com'
notify() {
    mailsubject="$(hostname) to be $1, vip转移"
    mailbody="$(date +'%F %T'): vrrp transition, $(hostname) changed to be $1"
    echo "$mailbody" | mail -s "$mailsubject" $contact
}
case $1 in
    master)
        notify master
       	;;
    backup)
        notify backup
        ;;
    fault)
        notify fault
        ;;
    *)
        echo "Usage: $(basename$0) {master|backup|fault}"
        exit 1
        ;;
esac
[root@c81 ~]# chmod +x /etc/keepalived/notify.sh
```

脚本的调用方法： notify_master "/etc/keepalived/notify.sh master"

notify_backup "/etc/keepalived/notify.sh backup"

notify_fault "/etc/keepalived/notify.sh fault"

```
vrrp_instance VI_1 {
    state MASTER
    interface ens160
    virtual_router_id 80
    priority 100
    advert_int 1
    unicast_src_ip 172.16.0.129
    unicast_peer{
        172.16.0.130
    }
    authentication {
        auth_type PASS
        auth_pass 1122
    }
    virtual_ipaddress {
        172.16.0.200 dev ens160
    }
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}

```

Keepalived通知验证

重新启动keepalived服务，验证IP切换后是否收到通知邮件：

![keepalived_05.png](./images/30-keepalived/keepalived_05.png)

### 3.10、KeepAlived与IPVS

```
虚拟服务器配置参数：
virtual server （虚拟服务）的定义：

virtual_server IP port #定义虚拟主机IP地址及其端口
virtual_server fwmark int #ipvs的防火墙打标，实现基于防火墙的负载均衡集群
virtual_server group string #将多个虚拟服务器定义成组，将组定义成虚拟服务
virtual_server IP port
{
    delay_loop <INT>：检查后端服务器的时间间隔
    lb_algo rr|wrr|lc|wlc|lblc|sh|dh：定义调度方法
    lb_kind NAT|DR|TUN：集群的类型
    persistence_timeout <INT>：持久连接时长
    protocol TCP|UDP|SCTP：指定服务协议
    sorry_server <IPADDR> <PORT>：所有RS故障时，备用服务器地址

    real_server <IPADDR> <PORT> {
        weight <INT> RS权重
        notify_up <STRING>|<QUOTED-STRING> RS上线通知脚本
        notify_down <STRING>|<QUOTED-STRING> RS下线通知脚本
        HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK { ... }：定义当前主机的健康状态检测方法
}
```

#### 3.10.1、应用层监测

```
HTTP_GET|SSL_GET：应用层检测
HTTP_GET|SSL_GET {
  url{
    path <URL_PATH>：定义要监控的URL
    status_code <INT>：判断上述检测机制为健康状态的响应码
  }
  connect_timeout <INTEGER>：连接请求的超时时长
  nb_get_retry <INT>：重试次数
  delay_before_retry <INT>：重试之前的延迟时长
  connect_ip <IP ADDRESS>：向当前RS哪个IP地址发起健康状态检测请求
  connect_port <PORT>：向当前RS的哪个PORT发起健康状态检测请求
  bindto <IP ADDRESS>：发出健康状态检测请求时使用的源地址
  bind_port <PORT>：发出健康状态检测请求时使用的源端口
}
```

#### 3.10.2、TCP监测

```
传输层检测TCP_CHECK
TCP_CHECK {
  connect_ip <IP ADDRESS>：向当前RS的哪个IP地址发起健康状态检测请求
  connect_port <PORT>：向当前RS的哪个PORT发起健康状态检测请求
  bindto <IP ADDRESS>：发出健康状态检测请求时使用的源地址
  bind_port <PORT>：发出健康状态检测请求时使用的源端口
  connect_timeout <INTEGER>：连接请求的超时时长
}
```

#### 3.10.3、Keepalived案例一：实现LVS-DR模式

```
virtual_server 172.16.0.200 80 {
    delay_loop 6
    lb_algo wrr
    lb_kind DR
    #persistence_timeout 120 #会话保持时间
    protocol TCP
    sorry_server 172.16.0.133 80
    
    real_server 172.16.0.131 80 {
        weight 1
        TCP_CHECK {
          connect_timeout 5
          nb_get_retry 3
          delay_before_retry 3
          connect_port 80
        }
    }
    real_server 172.16.0.132 80 {
        weight 1
        HTTP_GET {
          url {
            path /index.html
            status_code 200
          }
          connect_timeout 5
          nb_get_retry 3
          delay_before_retry 3
        }
    }
}

[root@c81 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.16.0.200:80 wrr
  -> 172.16.0.131:80              Route   1      0          0         
  -> 172.16.0.132:80              Route   1      0          0 

在realserver执行
[root@c83 ~]# cat real.sh 
#!/bin/bash
vip=172.16.0.200
mask='255.255.255.255'
dev=lo:1
case $1 in
    start)
        echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
        echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
        echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
        ip addr add $vip/32 dev $dev
        #route add -host $vip dev $dev
        ;;
    stop)
        ip addr del $vip/32 dev $dev
        echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
        echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
        echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
        ;;
    *)
        echo "Usage: $(basename $0) start|stop"
        exit 1
        ;;
esac
[root@c83 ~]# bash real.sh start
```

### 3.11、VRRP script

keepalived调用外部的辅助脚本进行资源监控，并根据监控的结果状态能实现优先动态调整

```
vrrp_script:自定义资源监控脚本，vrrp实例根据脚本返回值进行下一步操作，脚本可被多个实例调用。track_script:调用vrrp_script定义的脚本去监控资源，定义在实例之内，调用事先定义的vrrp_script

分两步：(1) 先定义一个脚本；(2) 调用此脚本
vrrp_script <SCRIPT_NAME> {
    script <STRING>|<QUOTED-STRING>
    OPTIONS
}
track_script{
    SCRIPT_NAME_1
    SCRIPT_NAME_2
}
```

```
vrrp_script <SCRIPT_NAME> { #定义一个检测脚本，在global_defs之外配置
    script <STRING>|<QUOTED-STRING> # shell命令或脚本路径
    interval <INTEGER> # 间隔时间，单位为秒，默认1秒
    timeout <INTEGER> # 超时时间
    weight <INTEGER:-254..254> # 权重，监测失败后会执行权重+操作
    fall <INTEGER> #脚本几次失败转换为失败
    rise <INTEGER> # 脚本连续监测成果后，把服务器从失败标记为成功的次数
    user USERNAME [GROUPNAME] # 执行监测的用户或组
    init_fail# 设置默认标记为失败状态，监测成功之后再转换为成功状态
}
```

```
vrrp_script chk_down { #基于第三方仲裁设备
    script "/bin/bash -c '[[ -f /etc/keepalived/down ]]' && exit 7 || exit 0"
    interval 1
    weight -30
    fall 3
    rise 5
    timeout 2
}

vrrp_instanceVI_1 {
    …
    track_script {
        chk_down
    }
}

[root@c81 ~]# tcpdump -i ens160 -nn host 172.16.0.130
19:49:51.815807 IP 172.16.0.130 > 172.16.0.129: VRRPv2, Advertisement, vrid 80, prio 80, authtype simple, intvl 1s, length 20

[root@c81 ~]# touch /etc/keepalived/down
[root@c81 ~]# tcpdump -i ens160 -nn host 172.16.0.130
19:52:10.558553 IP 172.16.0.129 > 172.16.0.130: VRRPv2, Advertisement, vrid 80, prio 100, authtype simple, intvl 1s, length 20
```

#### 3.11.1、高可用HAProxy

```
vrrp_scriptchk_haproxy{
    script "/etc/keepalived/chk_haproxy.sh"
    interval 1
    weight -30
    fall 3
    rise 5
    timeout 2
}

track_script{
    chk_haproxy
}

[root@c81 ~]# yum install psmisc -y
[root@c81 ~]# cat /etc/keepalived/chk_haproxy.sh
#!/bin/bash
/usr/bin/killall -0 haproxy
[root@c81 ~]# chmod a+x /etc/keepalived/chk_haproxy.sh
```

#### 3.11.2、高可用Nginx

```
vrrp_scriptchk_nginx{
    script "/etc/keepalived/chk_nginx.sh"
    interval 1
    weight -30
    fall 3
    rise 5
    timeout 2
}

track_script{
    chk_haproxy
}

[root@c81 ~]# yum install psmisc-y
[root@c81 ~]# cat /etc/keepalived/chk_nginx.sh
#!/bin/bash
/usr/bin/killall-0 nginx
[root@c81 ~]# chmod a+x /etc/keepalived/chk_nginx.sh
```

