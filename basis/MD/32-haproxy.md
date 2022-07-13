## 1、Web架构介绍

单机房架构

![haproxy_01](./images/32-haproxy/haproxy_01.png)

<!-- more -->

多机房架构

![haproxy_02](./images/32-haproxy/haproxy_02.png)

公有云Web架构：

![haproxy_03](./images/32-haproxy/haproxy_03.png)

私有云Web架构：

![haproxy_04](./images/32-haproxy/haproxy_04.png)

## 2、负载均衡介绍

### 2.1、什么是负载均衡

负载均衡(Load Balance，简称LB)是一种服务或基于硬件设备等实现的高可用反向代理技术，负载均衡将特定的业务(web服务、网络流量等)
分担给指定的一个或多个后端特定的服务器或设备，从而提高了公司业务的并发处理能力、保证了业务的高可用性、方便了业务后期的水平动态扩展。

https://yq.aliyun.com/articles/1803#阿里云SLB介绍

### 2.2、为什么使用负载均衡：

```
Web服务器的动态水平扩展
  对用户无感知
增加业务并发访问及处理能力
  解决单服务器瓶颈问题
节约公网IP地址
  降低IT支出成本
隐藏内部服务器IP
  提高内部服务器安全性
配置简单
  固定格式的配置文件
功能丰富
  支持四层和七层，支持动态下线主机
性能较强
  并发数万甚至数十万
```

### 2.3、常见有哪些负载均衡：

```
软件负载：
  四层：
    LVS(Linux Virtual Server)
    HAProxy(High Availability Proxy)
    Nginx()
    ……
  七层：
    HAProxy
    Nginx
    ……
硬件负载：
  F5
  etscaler
  ……
```

### 2.4、典型应用场景：

```
应用场景：
  四层：Redis、Mysql、RabbitMQ、Memcache等
  七层：Nginx、Tomcat、Apache、PHP、图片、动静分离、API等
```

## 3、HAProxy

HAProxy:是法国开发者Willy Tarreau开发的一个开源软件，是一款具备高并发、高性能的TCP和HTTP负载均衡器，支持基于cookie的持久性，自动故障切换，支持正则表达式及web状态统计。

```
LB Cluster:
  四层：lvs, nginx(stream模式且nginx1.9.0或更新版本)，haproxy(mode tcp)
  七层：http: nginx(http), haproxy(mode http), httpd...
官网：
  http://www.haproxy.org
  https://www.haproxy.com
文档：https://cbonte.github.io/haproxy-dconv/
```

### 3.1、HAProxy功能

```
HAProxy是TCP/HTTP反向代理服务器，尤其适合于高可用性高并发环境
   可以针对HTTP请求添加cookie，进行路由后端服务器
   可平衡负载至后端服务器，并支持持久连接
   支持基于cookie进行调度
   支持所有主服务器故障切换至备用服务器
   支持专用端口实现监控服务
   支持不影响现有连接情况下停止接受新连接请求
   可以在双向添加，修改或删除HTTP报文首部
   支持基于pattern实现连接请求的访问控制
   通过特定的URI为授权用户提供详细的状态信息
```

```
历史版本更新功能：1.4 1.5 1.6 1.7 1.8 1.9 2.0 2.2dev
  2.0：第7层网络重试，Traffic shadowing……
  1.8：多线程，HTTP/2缓存……
  1.7：服务器动态配置，多类型证书……
  1.6：DNS解析支持，HTTP连接多路复用……
  1.5：开始支持SSL，IPV6，keepalived……
```

### 3.2、HAProxy 应用

![haproxy_05](./images/32-haproxy/haproxy_05.png)

### 3.3、HAProxy安装：

#### 3.3.1、包管理器安装

Centos通过yum方式安装：

```
[root@c81 ~]# yum install haproxy -y
```

Ubuntu 安装：

```
[root@c81 ~]# apt-get install haproxy -y
```

#### 3.3.2、编译安装HAProxy：

##### 3.3.2.1、安装编译工具

```
[root@c82 ~]#  yum install gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl openssl-devel systemd-devel net-tools vim iotop bc zip unzip zlib-devel lrzsz tree screen lsof tcpdump wget -y
```

##### 3.3.2.2、编译安装：

```
[root@c82 src]# tar xf haproxy-1.8.24.tar.gz && cd haproxy-1.8.24
[root@c82 haproxy-1.8.24]# make ARCH=x86_64 TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_SYSTEMD=1 USE_CPU_AFFINITY=1 PREFIX=/usr/local/haproxy
[root@c82 haproxy-1.8.24]# make install PREFIX=/usr/local/haproxy
[root@c82 haproxy]# cp sbin/haproxy /usr/sbin/
```

##### 3.3.2.3、创建启动脚本：

```
[root@c82 haproxy]# cat /usr/lib/systemd/system/haproxy.service
[Unit]
Description=HAProxy Load Balancer
After=network.target

[Service]
Environment="CONFIG=/etc/haproxy/haproxy.cfg" "PIDFILE=/run/haproxy.pid"
ExecStartPre=/usr/sbin/haproxy -f $CONFIG -c -q
ExecStart=/usr/sbin/haproxy -Ws -f $CONFIG -p $PIDFILE
ExecReload=/usr/sbin/haproxy -f $CONFIG -c -q
ExecReload=/bin/kill -USR2 $MAINPID
KillMode=mixed
Type=notify

[Install]
WantedBy=multi-user.target
```

##### 3.3.2.4、创建目录和用户：

```
[root@c82 haproxy]# mkdir /etc/haproxy
[root@c82 haproxy]# useradd -r -s /sbin/nologin haproxy
[root@c82 haproxy]# cat /etc/haproxy/haproxy.cfg
global
    maxconn 100000
    chroot /usr/local/haproxy
    #stats socket /var/lib/haproxy/haproxy.sock mode 600 level admin
    uid 993
    #gid 99
    daemon
    nbproc 2
    cpu-map 1 0
    cpu-map 2 1
    #cpu-map 3 2
    #cpu-map 4 3
    pidfile /usr/local/haproxy/run/haproxy.pid
    log 127.0.0.1 local3 info

defaults
    option http-keep-alive
    option  forwardfor
    maxconn 100000
    mode http
    timeout connect 300000ms
    timeout client  300000ms
    timeout server  300000ms

listen stats
    bind :9999
    stats enable
    stats hide-version
    stats uri /haproxy-status
    stats realm HAPorxy\Stats\Page
    stats auth haadmin:123456
    stats auth admin:123456
    stats refresh 30s
    stats admin if TRUE

listen  web_port
    bind 0.0.0.0:80
    mode http
    log global
    server web1  127.0.0.1:8080  check inter 3000 fall 2 rise 5
    
    #haproxy.cfg文件中定义了chroot、pidfile、user、group等参数，如果系统没有相应的资源会导致haproxy无法启动，具体参考日志文件/var/log/messages

```

##### 3.3.2.5、启动HAProxy：

```
systemctl enable haproxy
systemctl restart haproxy
```

##### 3.3.2.6、验证HAProxy：

```
[root@c82 haproxy]# ps -ef | grep haproxy
root       8549      1  0 22:23 ?        00:00:00 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
haproxy    8550   8549  0 22:23 ?        00:00:00 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
haproxy    8551   8549  0 22:23 ?        00:00:00 /usr/sbin/haproxy -Ws -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
root       8555   1862  0 22:24 pts/0    00:00:00 grep --color=auto haproxy
```

```
[root@c82 haproxy]# lsof -i :80
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
haproxy 8550 haproxy    6u  IPv4  58120      0t0  TCP *:http (LISTEN)
haproxy 8551 haproxy    6u  IPv4  58120      0t0  TCP *:http (LISTEN)
```

### 3.4、HAProxy组成

```
程序环境：
主程序：/usr/sbin/haproxy
配置文件：/etc/haproxy/haproxy.cfg
Unit file：/usr/lib/systemd/system/haproxy.service

配置段：
  global：全局配置段
    进程及安全配置相关的参数
    性能调整相关参数
    Debug参数
  proxies：代理配置段
    defaults：为frontend, backend, listen提供默认配置
    frontend：前端，相当于nginx中的server {}
    backend：后端，相当于nginx中的upstream {}
    listen：同时拥有前端和后端配置
```

### 3.5、HAProxy配置

#### 3.5.1、Haproxy配置-global

官方配置文档：https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#3

```
global配置参数：
  chroot #锁定运行目录
  deamon #以守护进程运行
  #stats socket /var/lib/haproxy/haproxy.sockmode 600 level admin #socket文件
  user,group,uid,gid #运行haproxy的用户身份
  nbproc #开启的haproxy进程数，与CPU保持一致
  nbthread #指定每个haproxy进程开启的线程数，默认为每个进程一个线程
  cpu-map 1 0 #绑定haproxy进程至指定CPU
  maxconn #每个haproxy进程的最大并发连接数
  maxsslconn #SSL每个haproxy进程ssl最大连接数
  maxconnrate #每个进程每秒最大连接数
  spread-checks #后端server状态check随机提前或延迟百分比时间，建议2-5(20%-50%)之间
  pidfile #指定pid文件路径
  log 127.0.0.1 local3 info #定义全局的syslog服务器；最多可以定义两个
```

#### 3.5.2、HAProxy Proxies配置

官方配置文档：https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#4

```
defaults [<name>] #默认配置项，针对以下的frontend、backend和lsiten生效，可以多个name
frontend <name> #前端servername，类似于Nginx的一个虚拟主机server。
backend <name> #后端服务器组，等于nginx的upstream
listen <name> #将frontend和backend合并在一起配置

注：name字段只能使用”-”、”_”、”.”、和”:”，并且严格区分大小写，例如：Web和web是完全不同的两组服务器。
```

##### 3.5.2.1、Proxies配置-defaults

```
defaults 配置参数：
  option redispatch #当server Id对应的服务器挂掉后，强制定向到其他健康的服务器
  option abortonclose #当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接
  option http-keep-alive 60#开启会话保持
  option forwardfor #开启IP透传
  mode http #默认工作类型
  timeout connect 120s #转发客户端请求到后端server的最长连接时间(TCP之前)
  timeout server 600s #转发客户端请求到后端服务端的超时超时时长（TCP之后）
  timeout client 600s #与客户端的最长空闲时间
  timeout http-keep-alive 120s #session 会话保持超时时间，范围内会转发到相同的后端服务器
  #timeout check 5s #对后端服务器的检测超时时间
```

##### 3.5.2.2、Proxies配置-frontend配置参数

```
bind：指定HAProxy的监听地址，可以是IPV4或IPV6，可以同时监听多个IP或端口，可同时用于listen字段中
  bind [<address>]:<port_range> [, ...] [param*]
mode http/tcp #指定负载协议类型
use_backend backend_name #调用的后端服务器组名称

示例：
frontend WEB_PORT
    bind 172.16.0.130:10080,192.168.0.130:10043
    use_backend backend_name
```

##### 3.5.2.3、Proxies配置-backend配置参数

```
mode http/tcp #指定负载协议类型
option #配置选项
server #定义后端realserver

注意：option后面加httpchk，smtpchk, mysql-check, pgsql-check，ssl-hello-chk方法，可用于实现更多应用层检测功能。
```

**后端服务器状态监测及相关配置**

```
check #对指定real进行健康状态检查
  addr IP#可指定的健康状态监测IP
  port num#指定的健康状态监测端口
  inter num#健康状态检查间隔时间，默认2000 ms
  fall num#后端服务器失效检查次数，默认为3
  rise num#后端服务器从下线恢复检查次数，默认为2
  weight #默认为1，最大值为256，0表示不参与负载均衡
  backup #将后端服务器标记为备份状态
  disabled #将后端服务器标记为不可用状态
  redirect prefix http://www.magedu.com/#将请求临时重定向至其它URL，只适用于http模式
  maxconn <maxconn>：当前后端server的最大并发连接数
  backlog <backlog>：当server的连接数达到上限后的后援队列长度
```

###### # Web服务器状态监测：

```
三种状态监测方式：
  基于四层的传输端口做状态监测
  server 172.18.200.103 172.18.200.103:80 check port 9000 addr 172.18.200.104 inter 3s fall 3 rise 5 weight 1
  基于指定URI 做状态监测
  基于指定URI的request请求头部内容做状态监测
```

```
option httpchk
option httpchk <uri>
option httpchk <method> <uri>
option httpchk <method> <uri> <version>

listen web_prot_http_nodes
  bind 192.168.7.102:80
  mode http
  log global
  option httpchk GET /wp-includes/js/jquery/jquery.js?ver=1.12.4 HTTP/1.0 #基于指定URL
  #option httpchk HEAD /wp-includes/js/jquery/jquery.js?ver=1.12.4 HTTP/1.0\r\nHost:\192.168.7.102 #通过request获取的头部信息进行匹配进行健康检测
  server 192.168.7.102 blogs.studylinux.net:80 check inter 3000 fall 3 rise 5
  server 192.168.7.101 192.168.7.101:8080 cookie web1 check inter 3000 fall 3 rise 5
```

##### 3.5.2.4、frontend/ backend 配置案例

```
#官网业务访问入口======================================
frontend WEB_PORT_80
    bind 172.16.0.130:80
    mode http
    use_backend web_prot_http_nodes_80

frontend WEB_PORT_443
    bind 172.16.0.130:443
    mode http
    use_backend web_prot_http_nodes_443

backend web_prot_http_nodes_80
    mode http
    option forwardfor
    server web1 172.16.0.129:80 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:80 check inter 3000 fall 3 rise 5
  
backend web_prot_http_nodes_443
    mode http
    option forwardfor
    server web1 172.16.0.129:443 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:443 check inter 3000 fall 3 rise 5
```

##### 3.5.2.5、Proxies配置-listen

```
使用listen替换frontend和backend的配置方式：
#官网业务访问入口=====================================
listen WEB_PORT_80
    bind 172.16.0.130:80
    mode tcp
    option forwardfor
    server web1 172.16.0.129:80 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:80 check inter 3000 fall 3 rise 5

listen WEB_PORT_443
    bind 172.16.0.130:443
    mode tcp
    option forwardfor
    server web1 172.16.0.129:443 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:443 check inter 3000 fall 3 rise 5
```

#### 3.5.3、HAProxy 调度算法

balance：指明对后端服务器的调度算法，配置在listen或backend

##### 3.5.3.1、HAProxy 静态调度算法

```
静态算法：按照事先定义好的规则轮询公平调度，不关心后端服务器的当前负载、链接数和相应速度等，且无法实时修改权重，只能重启后生效。
  static-rr：基于权重的轮询调度，不支持权重的运行时调整及后端服务器慢启动，其后端主机数量没有限制
  first：根据服务器在列表中的位置，自上而下进行调度，但是其只会当第一台服务器的连接数达到上限，新请求才会分配给下一台服务，因此会忽略服务器的权重设置。
    注：关闭option http-keep-alive,timeout client 300s等优化选项才能测出效果
```

##### 3.5.3.2、HAProxy 动态调度算法

```
动态算法：基于后端服务器状态进行调度适当调整，比如优先调度至当前负载较低的服务器，且权重可以在haproxy运行时动态调整无需重启。
  roundrobin：基于权重的轮询动态调度算法，支持权重的运行时调整，不等于lvs的rr，支持慢启动即新加的服务器会逐渐增加转发数，每个后端backend中最多支持4095个server，此为默认调度算法，server权重设置weight
  leastconn：加权的最少连接的动态，支持权重的运行时调整和慢启动，即当前后端服务器连接最少的优先调度，比较适合长连接的场景使用，比如MySQL等场景。
```

##### 3.5.3.3、HAProxy 调度算法-source

```
source：源地址hash，基于用户源地址hash并将请求转发到后端服务器，默认为静态即取模方式，但是可以通过hash-type支持的选项更改，后续同一个源地址请求将被转发至同一个后端web服务器，比较适用于session保持/缓存业务等场景。
  map-based：取模法，基于服务器权重的hash数组取模，该hash是静态的即不支持在线调整权重，不支持慢启动，其对后端服务器调度均衡，缺点是当服务器的总权重发生变化时，即有服务器上线或下线，都会因权重发生变化而导致调度结果整体改变，hash（o）modn 。
  consistent：一致性哈希，该hash是动态的，支持在线调整权重，支持慢启动，优点在于当服务器的总权重发生变化时，对调度结果影响是局部的，不会引起大的变动。
```

```
listen web_prot_http_nodes
    bind 172.16.0.130:80
    mode http
    balance source
    hash-type consistent
    log global
    option forwardfor
    server web1 172.16.0.129:80 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:80 check inter 3000 fall 3 rise 5
```

一致性hash算法

![hash](./images/32-haproxy/haproxy_06.png)

![haproxy_07](./images/32-haproxy/haproxy_07.png)![haproxy_08](./images/32-haproxy/haproxy_08.png)

##### 3.5.3.4、HAProxy 调度算法-uri

```
uri：基于对用户请求的uri做hash并将请求转发到后端指定服务器
  map-based：取模法
  consistent：一致性哈希
    http://example.org/absolute/URI/with/absolute/path/to/resource.txt #URI/URL
    ftp://example.org/resource.txt #URI/URL
    /relative/URI/with/absolute/path/to/resource.txt #URI

uri: uniform resource identifier，统一资源标识符,是一个用于标识某一互联网资源名称的字符串
```

```
listen web_prot_http_nodes
    bind 172.16.0.130:80
    mode http #不支持tcp，会切换到tcp的roundrobin负载模式
    balance uri
    hash-type consistent
    log global
    option forwardfor
    server web1 172.16.0.129:80 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:80 check inter 3000 fall 3 rise 5

[root@c82 ~]# curl http://172.16.0.130/index.html
web2 172.16.0.131
[root@c82 ~]# curl http://172.16.0.130/test.html
test 172.16.0.131
```

##### 3.5.3.5、HAProxy 调度算法-url_param

```
url_param：
  对用户请求的url中的<params>部分中的参数name作hash计算，并由服务器总权重相除以后派发至某挑出的服务器；通常用于追踪用户，以确保来自同一个用户的请求始终发往同一个Backend Server
  
  假设url= http://www.magedu.com/foo/bar/index.php?k1=v1&k2=v2
    则：
    host = "www.magedu.com"
    url_param= "k1=v1&k2=v2"
```

```
listen web_prot_http_nodes
    bind 172.16.0.130:80
    mode http #不支持tcp，会切换到tcp的roundrobin负载模式
    balance uri_param name #基于参数name做hash
    hash-type consistent
    log global
    option forwardfor
    server web1 172.16.0.129:80 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:80 check inter 3000 fall 3 rise 5

[root@c82 ~]# curl http://172.16.0.130/index.html?name=tom
web1 172.16.0.129
[root@c82 ~]# curl http://172.16.0.130/index.html?name=tom
web1 172.16.0.129
[root@c82 ~]# curl http://172.16.0.130/index.html?name=tom
web1 172.16.0.129
[root@c82 ~]# curl http://172.16.0.130/index.html?password=123456
web2 172.16.0.131
[root@c82 ~]# curl http://172.16.0.130/index.html?password=123456
web2 172.16.0.131
[root@c82 ~]# curl http://172.16.0.130/index.html?password=123456
web1 172.16.0.129
[root@c82 ~]# curl http://172.16.0.130/index.html?password=123456
web1 172.16.0.129
```

##### 3.5.3.6、HAProxy 调度算法-hdr

```
hdr(<name>)：针对用户每个http头部(header)请求中的指定信息做hash，此处由<name>指定的http首部将会被取出并做hash计算，然后由服务器总权重相除以后派发至某挑出的服务器，假如无有效的值，则会被轮询调度
  hdr( Cookie、User-Agent、host )
```

```
listen web_prot_http_nodes
    bind 172.16.0.130:80
    mode http #不支持tcp，会切换到tcp的roundrobin负载模式
    balance hdr(User-Agent)
    hash-type consistent
    log global
    option forwardfor
    server web1 172.16.0.129:80 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:80 check inter 3000 fall 3 rise 5

[root@c82 ~]# curl -A 4 http://172.16.0.130/index.html
web1 172.16.0.129
[root@c82 ~]# curl -A 3 http://172.16.0.130/index.html
web2 172.16.0.131
[root@c82 ~]# curl -A 2 http://172.16.0.130/index.html
web2 172.16.0.131
[root@c82 ~]# curl -A 4 http://172.16.0.130/index.html
web1 172.16.0.129
[root@c82 ~]# curl -A 1 http://172.16.0.130/index.html
web2 172.16.0.131
[root@c82 ~]# curl -A 4 http://172.16.0.130/index.html
web1 172.16.0.129
```

##### 3.5.3.7、HAProxy 调度算法-rdp-cookie

```
rdp-cookie对远程桌面的负载，使用cookie保持会话
  rdp-cookie(<name>)
```

```
listen RDP
    bind 172.16.0.130:3389
    balance rdp-cookie
    mode tcp
    server rdp0 172.16.0.1:3389 check fall 3 rise 5 inter 2000 weight 1
    server rdp1 172.16.0.2:3389 check fall 3 rise 5 inter 2000 weight 1

```

![haproxy_09](./images/32-haproxy/haproxy_09.png)

##### 3.5.3.8、算法总结：

```
static-rr--------->tcp/http 静态
first------------->tcp/http 静态
roundrobin-------->tcp/http 动态
leastconn--------->tcp/http 动态
source------------>tcp/http ---|
Uri--------------->http        |
url_param--------->http        |---> 取决于hash_type是否consistent
hdr--------------->http        |
rdp-cookie-------->tcp   ------|
```

#### 3.5.4、ACL定义与调用

官方配置文档：https://cbonte.github.io/haproxy-dconv/1.8/configuration.html#7

```
acl：对接收到的报文进行匹配和过滤，基于请求报文头部中的源地址、目标地址、源端口、目标端口、请求方法、URL、文件后缀等信息内容进行匹配并执行进一步操作。

acl <aclname> <criterion> [flags] [operator] [<value>]

acl 名称条件条件标记位具体操作符操作对象类型
acl image_service hdr_dom(host) -i img.magedu.com
ACL名称，可以使用大字母A-Z、小写字母a-z、冒号：、点.、中横线和下划线，并且严格区分大小写，必须Image_site和image_site完全是两个acl。
```

##### 3.5.4.1、criterion

**地址和端口匹配**

```
<criterion> ：匹配条件
  src 源IP
  dst 目标IP
  src_port 源PORT
  dst_port 目标PORT 
```

**httpd**报文头部匹配**

```
hdr（[<name> [，<occ>]]）：完全匹配字符串
hdr_beg（[<name> [，<occ>]]）：前缀匹配
hdr_dir（[<name> [，<occ>]]）：路径匹配
hdr_dom（[<name> [，<occ>]]）：域匹配
hdr_end（[<name> [，<occ>]]）：后缀匹配
hdr_len（[<name> [，<occ>]]）：长度匹配
hdr_reg（[<name> [，<occ>]]）：正则表达式匹配
hdr_sub（[<name> [，<occ>]]）：子串匹配

hdr <string>用于测试请求头部首部指定内容
hdr_dom(host) 请求的host名称，如www.magedu.com
hdr_beg(host) 请求的host开头，如www. img. video. download. ftp.
hdr_end(host) 请求的host结尾，如.com .net .cn
```

**路径匹配**

```
path_beg 请求的URL开头，如/static、/images、/img、/css
path_end 请求的URL中资源的结尾，如.gif .png .css .js .jpg .jpeg
```

##### 3.5.4.2、flags

```
<flags>-条件标记
  -i 不区分大小写
  -m 使用指定的pattern匹配方法
  -n 不做DNS解析
  -u 禁止acl重名，否则多个同名ACL匹配或关系
```

##### 3.5.4.3、operator

```
[operator]-操作符：
整数比较：eq、ge、gt、le、lt
字符比较：
  -exact match (-m str) :字符串必须完全匹配模式
  -substring match (-m sub) :在提取的字符串中查找模式，如果其中任何一个被发现，ACL将匹配
  -prefix match (-m beg) :在提取的字符串首部中查找模式，如果其中任何一个被发现，ACL将匹配
  -suffix match (-m end) :将模式与提取字符串的尾部进行比较，如果其中任何一个匹配，则ACL进行匹配
  -subdir match (-m dir) :查看提取出来的用斜线分隔（“/”）的字符串，如果其中任何一个匹配，则ACL进行匹配
  -domain match (-m dom) :查找提取的用点（“.”）分隔字符串，如果其中任何一个匹配，则ACL进行匹配
```

##### 3.5.4.4、value

```
<value>的类型：
  -Boolean #布尔值false，true
  -integer or integer range #整数或整数范围，比如用于匹配端口范围，1024～32768
  -IP address / network #IP地址或IP范围, 192.168.0.1 ,192.168.0.1/24
  -string
    exact –精确比较
    substring—子串www.magedu.com
    suffix-后缀比较
    prefix-前缀比较
    subdir-路径，/wp-includes/js/jquery/jquery.js
    domain-域名，www.magedu.com
  -regular expression #正则表达式
  -hex block #16进制
```

##### 3.5.4.5、多ACL逻辑关系

```
多个acl作为条件时的逻辑关系：
  -与：隐式（默认）使用
  -或：使用“or” 或“||”表示
  -否定：使用“!“ 表示
示例：
  if valid_src valid_port #与关系
  if invalid_src || invalid_port #或
  if ! invalid_src #非
```

##### 3.5.4.6、预定义acl

```
ACL name         Equivalent to                 Usage
FALSE            always_false                  never match
HTTP             req_proto_http                match if protocol is valid HTTP
HTTP_1.0         req_ver 1.0                   match HTTP version 1.0
HTTP_1.1         req_ver 1.1                   match HTTP version 1.1
HTTP_CONTENT     hdr_val(content-length) gt 0  match an existing content-length
HTTP_URL_ABS     url_reg ^[^/:]*://            match absolute URL with scheme
HTTP_URL_SLASH   url_beg /                     match URL beginning with "/"
HTTP_URL_STAR    url *                         match URL equal to "*"
LOCALHOST        src 127.0.0.1/8               match connection from local host
METH_CONNECT     method CONNECT                match HTTP CONNECT method
METH_DELETE      method DELETE                 match HTTP DELETE method
METH_GET         method GET HEAD               match HTTP GET or HEAD method
METH_HEAD        method HEAD                   match HTTP HEAD method
METH_OPTIONS     method OPTIONS                match HTTP OPTIONS method
METH_POST        method POST                   match HTTP POST method
METH_PUT         method PUT                    match HTTP PUT method
METH_TRACE       method TRACE                  match HTTP TRACE method
RDP_COOKIE       req_rdp_cookie_cnt gt 0       match presence of an RDP cookie
REQ_CONTENT      req_len gt 0                  match data in the request buffer
TRUE             always_true                   always match
WAIT_END         wait_end                      wait for end of content analysis
```

预定义acl使用

```
listen web_port
    bind 172.16.0.130:80
    mode http
    use_backend web1 if HTTP_1.1
    default_backend web2
 
backend web1
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5
 
backend web2
    mode http
    server web1 172.16.0.131:80 check inter 2000 fall 3 rise 5

[root@c82 ~]# curl 172.16.0.130
web1 172.16.0.129
[root@c82 ~]# curl 172.16.0.130
web1 172.16.0.129
[root@c82 ~]# curl -0 172.16.0.130
web2 172.16.0.131
[root@c82 ~]# curl -0 172.16.0.130
web2 172.16.0.131
```

##### 3.5.4.7、ACL示例

Acl 示例-域名匹配：

```
listen web_port
    bind 172.16.0.130:80
    mode http
    log global
    acl test_host hdr_dom(host) www.zsjshao.net
    use_backend test_host if test_host
    default_backend default_web     #以上都没有匹配到的时候使用默认backend
 
backend test_host
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5
 
backend default_web
    mode http
    server web2 172.16.0.131 check inter 2000 fall 3 rise 5

[root@c82 ~]# curl http://www.zsjshao.net
web1 172.16.0.129
[root@c82 ~]# curl http://www.zsjshao.net
web1 172.16.0.129
[root@c82 ~]# curl http://www.zsjshao.net
web1 172.16.0.129
[root@c82 ~]# curl http://mobile.zsjshao.net
web2 172.16.0.131
[root@c82 ~]# curl http://mobile.zsjshao.net
web2 172.16.0.131
[root@c82 ~]# curl http://172.16.0.130
web2 172.16.0.131
```

Acl-源地址子网匹配

```
listen web_port
    bind 172.16.0.130:80
    mode http
    log global
    acl ip_range_test src 172.16.0.130 192.168.0.0/24
    use_backend web1 if ip_range_test
    default_backend web2
 
backend web1
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5
 
backend web2
    mode http
    server web1 172.16.0.131:80 check inter 2000 fall 3 rise 5

[root@c82 ~]# curl 172.16.0.130
web1 172.16.0.129
[root@c82 ~]# curl 172.16.0.130
web1 172.16.0.129

[root@c83 ~]# curl 172.16.0.130
web2 172.16.0.131
[root@c83 ~]# curl 172.16.0.130
web2 172.16.0.131
```

Acl示例-匹配浏览器

```
listen web_port
    bind 172.16.0.130:80
    mode http
    log global
    acl Chrome hdr(User-Agent) -m sub -i "Chrome"
    use_backend web1 if Chrome
    default_backend web2
 
backend web1
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5
 
backend web2
    mode http
    server web1 172.16.0.131:80 check inter 2000 fall 3 rise 5

[root@c82 ~]# curl -A Chrome 172.16.0.130
web1 172.16.0.129
[root@c82 ~]# curl -A Chrome 172.16.0.130
web1 172.16.0.129
[root@c82 ~]# curl  172.16.0.130
web2 172.16.0.131
[root@c82 ~]# curl  172.16.0.130
web2 172.16.0.131
```

基于acl+文件后缀实现动静分离

```
listen web_port
    bind 172.16.0.130:80
    mode http
    acl php_server path_end -i .php
    use_backend php_server_host if php_server
    acl image_server path_end -i .jpg .png .jpeg .gif
    use_backend image_server_host if image_server
    default_backend default_host
 
backend php_server_host
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5
 
backend image_server_host
    mode http
    server web1 172.16.0.131:80 check inter 2000 fall 3 rise 5
 
backend default_host
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5

[root@c81 ~]# echo web1 172.16.0.129 this is phpfile > /var/www/html/test.php

[root@c83 ~]# echo web2 172.16.0.131 this is image > /var/www/html/image.jpg

[root@c82 ~]# curl 172.16.0.130
web1 172.16.0.129
[root@c82 ~]# curl 172.16.0.130/image.jpg
web2 172.16.0.131 this is image
[root@c82 ~]# curl 172.16.0.130/test.php
web1 172.16.0.129 this is phpfile
```

acl-匹配访问路径

```
listen web_port
    bind 172.16.0.130:80
    mode http
    acl static_path path_beg -i /static /images /javascript
    use_backend static_path_host if static_path
    default_backend default_host

backend static_path_host
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5

backend default_host
    mode http
    server web1 172.16.0.131:80 check inter 2000 fall 3 rise 5

[root@c81 ~]# mkdir /var/www/html/{static,images,javascript}
[root@c81 ~]# echo web1 172.16.0.129 static test file > /var/www/html/static/index.html

[root@c82 ~]# curl 172.16.0.130
web2 172.16.0.131
[root@c82 ~]# curl 172.16.0.130/static/index.html
web1 172.16.0.129 static test file
[root@c82 ~]# 
```

#### 3.5.5、Cookie 配置

```
cookie <value>：为当前server指定cookie值，实现基于cookie的会话黏性
  cookie <name> [ rewrite | insert | prefix ] [ indirect ] [ nocache ] [ postonly ] [ preserve ] [ httponly ] [ secure ] [ domain <domain> ]* [ maxidle <idle> ] [ maxlife <life> ]
  <name>：cookie名称，用于实现持久连接
    rewrite：重写
    insert：插入
    prefix：前缀
    nocache：当client和hapoxy之间有缓存时，不缓存cookie
```

##### 3.5.5.1、基于cookie实现的session 保持

```
listen web_prot_http_nodes
    bind 172.16.0.130:80
    mode http
    cookie SERVER-COOKIE insert indirect nocache
    server web1 172.16.0.129:80 cookie web1 check inter 3000 fall 3 rise 5
    server web2 172.16.0.131:80 cookie web2 check inter 3000 fall 3 rise 5
```

![haproxy_10](./images/32-haproxy/haproxy_10.png)

#### 3.5.6、修改报文首部

模式必须是http，haproxy才能修改报文首部信息

##### 3.5.6.1、添加头部

```
在请求报文尾部添加指定报文：
  reqadd <string> [{if | unless} <cond>]#支持条件判断
在响应报文尾部添加指定报文：
  rspadd <string> [{if | unless} <cond>]

示例：rspaddX-Via:\HAPorxy
[root@c82 ~]# curl -I 172.16.0.130
HTTP/1.1 200 OK
Date: Mon, 02 Mar 2020 11:39:04 GMT
Server: Apache/2.4.37 (centos)
Last-Modified: Thu, 27 Feb 2020 06:56:32 GMT
ETag: "12-59f8938a02dce"
Accept-Ranges: bytes
Content-Length: 18
Content-Type: text/html; charset=UTF-8
X-Via: Haproxy
```

##### 3.5.6.2、删除头部

```
从请求报文中删除匹配正则表达式的首部
  reqdel <search> [{if | unless} <cond>]
  reqidel <search> [{if | unless} <cond>] 不分大小写
从响应报文中删除匹配正则表达式的首部
  rspdel <search> [{if | unless} <cond>]
  rspidel <search> [{if | unless} <cond>]

示例：rspidel Server.* #从响应应报文删除server信息
  rspidel Server.*
[root@c82 ~]# curl -I 172.16.0.130
HTTP/1.1 200 OK
Date: Mon, 02 Mar 2020 11:41:22 GMT
Last-Modified: Thu, 27 Feb 2020 06:56:32 GMT
ETag: "12-59f8938a02dce"
Accept-Ranges: bytes
Content-Length: 18
Content-Type: text/html; charset=UTF-8
X-Via: Haproxy
```

#### 3.5.7、压缩功能

```
compression algo #启用http协议中的压缩机制，常用算法有gzip deflate
  compression type #要压缩的类型
示例：
  compression algo gzip deflate
  compression type text/plain text/html text/css text/xml text/javascript application/javascript
```

![haproxy_11](./images/32-haproxy/haproxy_11.png)

#### 3.5.8、HAProxy-https协议

```
配置HAProxy支持https协议：
支持ssl会话；
  bind *:443 ssl crt /PATH/TO/SOME_PEM_FILE
  crt 后证书文件为PEM格式，且同时包含证书和所有私钥
  cat demo.crt demo.key > demo.pem
把80端口的请求重向定443
  bind *:80
  redirect scheme https if !{ ssl_fc }
向后端传递用户请求的协议和端口（frontend或backend）
  http_request set-header X-Forwarded-Port %[dst_port]
  http_request add-header X-Forwared-Proto https if { ssl_fc }
```

https-证书制作

```
[root@c82 ~]# mkdir /usr/local/haproxy/certs
[root@c82 ~]# cd /usr/local/haproxy/certs
[root@c82 certs]# openssl genrsa -out haproxy.key 2048
[root@c82 certs]# openssl req -new -x509 -key haproxy.key -out haproxy.crt -subj "/CN=www.zsjshao.net"
[root@c82 certs]# cat haproxy.key haproxy.crt > haproxy.pem
[root@c82 certs]# openssl x509 -in haproxy.pem -noout -text  #查看证书
```

https 示例

```
frontend web_server-http
    bind 172.16.0.130:80
    redirect scheme https if !{ ssl_fc }
    mode http
    use_backend web_host

#web server https
frontend web_server-https
    bind 172.16.0.130:443 ssl crt /usr/local/haproxy/certs/haproxy.pem
    mode http
    use_backend web_host

backend web_host
    mode http
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5
    server web2 172.16.0.131:80 check inter 2000 fall 3 rise 5
```

#### 3.5.9、HAProxy 日志配置

```
在default配置项定义：
  log 127.0.0.1 local{1-7} info #基于syslog记录日志到指定设备，级别有(err、warning、info、debug)

配置rsyslog：
  $ModLoad imudp
  $UDPServerRun 514
  local3.* /var/log/haproxy.log

配置HAProxy：
  listen web_port
    bind 127.0.0.1:80
    mode http
    log global
    option tcplog
    server web1 127.0.0.1:8080 check inter 3000 fall 2 rise 5
重启syslog服务并访问haproxy状态页
[root@c82 ~]# tail /var/log/haproxy.log 
Mar  2 19:47:36 localhost haproxy[2150]: Connect from 172.16.0.1:3183 to 172.16.0.130:9999 (stats/HTTP)
Mar  2 19:47:36 localhost haproxy[2150]: Connect from 172.16.0.1:3184 to 172.16.0.130:9999 (stats/HTTP)
```

##### 3.5.9.1、自定义记录日志

```
将特定信息记录在日志中(1.8版本不生效)
  capture cookie <name> len <length> #捕获请求和响应报文中的cookie并记录日志
  capture request header <name> len <length> #捕获请求报文中指定的首部内容和长度并记录日志
  capture response header <name> len <length> #捕获响应报文中指定的内容和长度首部并记录日志
示例：
  capture request header Host len 256
  capture request header User-Agent len 512
```

#### 3.5.10、配置HAProxy状态页：

```
stats enable #基于默认的参数启用stats page
stats hide-version # 隐藏版本
stats refresh <delay> # 设定自动刷新时间间隔
stats uri <prefix> #自定义stats page uri，默认值：/haproxy?stats
stats realm <realm> #账户认证时的提示信息，示例：stats realm : HAProxy\Statistics
stats auth <user>:<passwd> #认证时的账号和密码，可使用多次，默认：no authentication
stats admin { if | unless } <cond> #启用stats page中的管理功能
```

```
listen stats
    bind :9999
    stats enable
    #stats hide-version
    stats uri /haproxy-status
    stats realm HAPorxy\Stats\Page
    stats auth haadmin:123456
    stats auth admin:123456
    stats refresh 30s
    stats admin if TRUE
```

验证HAProxy状态页：

```
pid = 3698 (process #2, nbproc = 2, nbthread = 2) #pid为当前pid号，process为当前进程号，nbproc和nbthread为一共多少进程和每个进程多少个线程
uptime = 0d 0h00m08s #启动了多长时间
system limits: memmax = unlimited; ulimit-n = 131124 #系统资源限制：内存/最大打开文件数/
maxsock = 131124; maxconn = 65536; maxpipes = 0 #最大socket连接数/单进程最大连接数/最大管道数maxpipes
current conns = 1; current pipes = 0/0; conn rate = 1/sec #当前连接数/当前管道数/当前连接速率
Running tasks: 1/9; idle = 100 % #运行的任务/当前空闲率

active UP：#在线服务器    backup UP：#标记为backup的服务器
active UP, going down：#监测未通过正在进入down过程    backup UP, going down：#备份服务器正在进入down过程
active DOWN, going up：#down的服务器正在进入up过程    backup DOWN, going up：#备份服务器正在进入up过程
active or backup DOWN：#在线的服务器或者是backup的服务器已经转换成了down状态    not checked：#标记为不监测的服务器
active or backup DOWN for maintenance (MAINT) #active或者backup服务器认为下线的
active or backup SOFT STOPPED for maintenance #active或者backup被认为软下线(人为将weight改成0)
```

![haproxy_12](./images/32-haproxy/haproxy_12.png)

```
session rate(每秒的连接会话信息)：      Errors(错误统计信息)：
  cur:每秒的当前会话数量                 Req:错误请求量
  max:每秒新的最大会话数量               conn:错误链接量
  limit:每秒新的会话限制量               Resp:错误响应量

sessions(会话信息)：                   Warnings(警告统计信息)：
  cur:当前会话量                         Retr:重新尝试次数
  max:最大会话量                         Redis:再次发送次数
  limit: 限制会话量
  Total:总共会话量                     Server(real server信息)：
  LBTot:选中一台服务器所用的总时间          Status:后端机的状态，包括UP和DOWN
  Last：和服务器的持续连接时间              LastChk:持续检查后端服务器的时间
                                        Wght:权重
Bytes(流量统计)：                         Act:活动链接数量
  In:网络的字节输入总量                    Bck:备份的服务器数量
  Out:网络的字节输出总量                   Chk:心跳检测时间
                                         Dwn:后端服务器连接后都是DOWN的数量
Denied(拒绝统计信息)：                     Dwntme:总的downtime时间
  Req:拒绝请求量                          Thrtle:server 状态
  Resp:拒绝回复量
```

#### 3.5.11、自定义错误页面

```
errorfile 500 /usr/local/haproxy/html/500.html #自定义错误页面跳转
errorfile 502 /usr/local/haproxy/html/502.html
errorfile 503 /usr/local/haproxy/html/503.html
```

#### 3.5.12、自定义错误跳转

```
errorloc 503 http://192.168.7.103/error_page/503.html
```

### 3.6、四层与七层的区别：

四层：

在四层负载设备中，把client发送的报文目标地址(原来是负载均衡设备的IP地址)，根据均衡设备设置的选择web服务器的规则选择对应的web服务器IP地址，这样client就可以直接跟此服务器建立TCP连接并发送数据。

七层：

七层负载均衡服务器起了一个代理服务器的作用，服务器建立一次TCP连接要三次握手，而client要访问webserver要先与七层负载设备进行三次握手后建立TCP连接，把要访问的报文信息发送给七层负载均衡；然后七层负载均衡再根据设置的均衡规则选择特定的webserver，然后通过三次握手与此台webserver建立TCP连接，然后webserver把需要的数据发送给七层负载均衡设备，负载均衡设备再把数据发送给client；所以，七层负载均衡设备起到了代理服务器的作用。

![haproxy_13](./images/32-haproxy/haproxy_13.png)

七层IP 透传：

```
七层负载：
listen web_prot_http_nodes
    bind 172.16.0.130:80
    mode http
    option forwardfor
    server web1 172.16.0.129:80 check inter 3000 fall 3 rise 5

#后端web服务器配置
1、Apache:
[root@c82 ~]# vim /etc/httpd/conf/httpd.conf
  LogFormat "%{X-Forwarded-For}i %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined

172.16.0.1 172.16.0.130 - - [01/Mar/2020:02:07:13 +0800] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36 chrome-extension"

2、Nginx：
[root@c82 ~]# cat /apps/nginx/conf/nginx.conf
  "$http_x_forwarded_for"' #默认日志格式就有此配置

172.16.0.130 - - [01/Mar/2020:02:04:57 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36 chrome-extension" "172.16.0.1"
```

四层IP 透传：

```
四层负载：
listen web_prot_http_nodes
    bind 172.16.0.130:80
    mode tcp
    server web1 172.16.0.129:80 send-proxy check inter 3000 fall 3 rise 5

Nginx配置：（慎用，最好不用）
  listen 80 proxy_protocol;
  '"tcp_ip":"$proxy_protocol_addr",' #TCP获取客户端真实IP日志格式

172.16.0.130 - - [01/Mar/2020:02:12:58 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36 chrome-extension" "-" "172.16.0.1

```

四层负载

```
Memcache
Redis
MySQL
RabbitMQ
Web Server
……
listen redis-port
    bind 172.16.0.130:6379
    mode tcp
    balance leastconn
    server server1 172.16.0.129:6379 check
    server server2 172.16.0.131:6379 check backup
```

四层访问控制

```
tcp-request connection {accept|reject} [{if | unless} <condition>]
  根据第4层条件对传入连接执行操作
listen redis-port
    bind 172.16.0.130:6379
    mode tcp
    balance leastconn
    acl invalid_src src 172.16.1.0/24 172.16.0.101
    tcp-request connection reject if invalid_src
    server server1 172.16.0.129:6379 check
    server server1 172.16.0.131:6379 check backup
```

http 基于策略的访问控制

```
listen web_port
    bind 172.16.0.130:80
    mode http
    acl badguy_deny src 172.16.0.210 172.16.1.0/24
    http-request deny if badguy_deny
    http-request allow
    default_backend default_host

backend default_host
    mode http
    server web1 172.16.0.129:80 check inter 2000 fall 3 rise 5
```

### 3.7、HAProxy-服务器动态上下线

```
[root@c82 ~]# vim /etc/haproxy/haproxy.cfg
global
  nbproc 2
  stats socket /var/lib/haproxy/haproxy.sock1 mode 600 level admin process 1
  stats socket /var/lib/haproxy/haproxy.sock2 mode 600 level admin process 2
[root@c82 ~]# systemctl restart haproxy

[root@c82 ~]# yum install socat -y
[root@c82 ~]# echo help | socat stdio /var/lib/haproxy/haproxy.sock1

设置realserver权重
[root@c82 ~]# echo "get weight WEB_PORT_80/web1" | socat stdio /var/lib/haproxy/haproxy.sock1
[root@c82 ~]# echo "get weight WEB_PORT_80/web1" | socat stdio /var/lib/haproxy/haproxy.sock2
[root@c82 ~]# echo "set weight WEB_PORT_80/web2 3" | socat stdio /var/lib/haproxy/haproxy.sock1
[root@c82 ~]# echo "set weight WEB_PORT_80/web2 3" | socat stdio /var/lib/haproxy/haproxy.sock2

临时禁用realserver
[root@c82 ~]# echo disable server  "WEB_PORT_80/web2" | socat stdio /var/lib/haproxy/haproxy.sock1
[root@c82 ~]# echo disable server  "WEB_PORT_80/web2" | socat stdio /var/lib/haproxy/haproxy.sock2
```

注：多进程需要给所有sock文件发送指令，不然会出现异常现象。
