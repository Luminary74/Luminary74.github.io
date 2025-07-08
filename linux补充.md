## 配置网卡

**vi  /etc/sysconfig/network-scripts/ifcfg-ens33**

未配置的网卡初始信息

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="9409a8c3-ab6c-4590-af37-eb1496c1cacf"
DEVICE="ens33"
ONBOOT="yes"
```

配置的网卡信息

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=10.0.0.5
NETMASK=255.255.255.0
GATEWAY=10.0.0.254
DNS1=223.5.5.5          # 主DNS（推荐阿里公共DNS）
DNS2=114.114.114.114    # 备用DNS（全网通用）
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=27a73818-89f3-4991-9890-2da7924a73c4
DEVICE=ens33
ONBOOT=yes
```

编辑完毕后，点击键盘上的ESC键，然后输入:wq点击回车确认write quit

systemctl restart network  --重启网卡 start stop

service network restart --配置网络信息之后必须重启网卡才能更新

ping www.baidu.com 进行验证是否成功

配置外网：

可以ping 外网地址 ping 223.5.5.5

可以ping外网域名  ping www.baidu.com

如果ping不通 如何解决:

添加域名访问功能 ---DNS server :223.5.5.5

systemctl restart network

涉及到的命令 nmtui 进入编辑网卡界面

北方 223.5.5.5  南方114.114.114.114



## 配置yum源

1、cd /etc/yum.repos.d/ 

该目录存放 CentOS 系统的 YUM 仓库配置文件（`.repo` 文件），所有软件源定义在此处

```shell
[root@koko ~]# cd /etc/yum.repos.d/
[root@koko yum.repos.d]# ls
CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo  epel.repo

```

2、curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

阿里云镜像站下载 CentOS 7 的仓库配置文件，并保存为 `CentOS-Base.repo`

```shell
[root@koko yum.repos.d]# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2523  100  2523    0     0  12318      0 --:--:-- --:--:-- --:--:-- 12367
```

3、cat CentOS-Base.repo	

在终端打印 `CentOS-Base.repo` 文件的内容

```shell
[root@koko yum.repos.d]# cat CentOS-Base.repo
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

```

4、下载测试

```shell
下载一些普通程序：
  curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
下载一些扩展程序：
  curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
```

centos中有些基础软件可以提前下载好

tree  树形图

nmap  探测端口状态是否开放

lrzsz  下载上传软件

vim  文本编辑器

bash-completion  控制网络命令的

```shell
yum install -y  nmap  bash-completion lrzsz tree vim sl cowsay net-tools 

sl

cowsay "helloworld"

animalsay "hello"
```

扩展：yum search 命令名 ---找命令名相关的程序包

   yum makecache  将安装包缓存 可能会解决找不到包的问题

   yum clean all   清除缓存

   yum remove 命令名  移除命令文件

## 优化ssh连接

修改 SSH 服务器配置以提高连接速度和安全性

1、打开配置文件

```shell
vim /etc/ssh/sshd_config
```

2、定位并修改以下参数

```shell
79 GSSAPIAuthentication no
82 GSSAPIKeyExchange no
103 X11UseLocalhost no
115  UseDNS no
```

| 配置项               | 原值 | 新值 | 作用                                                         |
| -------------------- | ---- | ---- | ------------------------------------------------------------ |
| GSSAPIAuthentication | yes  | no   | 禁用 GSSAPI 认证，加速 SSH 登录过程                          |
| GSSAPIKeyExchange    | yes  | no   | 禁用 GSSAPI 密钥交换，减少认证环节                           |
| UseDNS               | yes  | no   | 禁用 DNS 反向解析，解决 SSH 连接延迟问题（尤其 DNS 服务不可用时） |
| X11UseLocalhost      | yes  | no   | 允许远程主机连接 X11 转发，启用远程 GUI 应用支持             |

3、重启远程服务 使优化配置生效

systemctl restart sshd

4、关闭开机自启项：firewalld selinux NetworkManager

**关闭并禁用 firewalld**

systemctl stop firewalld && systemctl disable firewalld 

**关闭并禁用 NetworkManager**

systemctl stop NetworkManager && systemctl disable NetworkManager

vim /etc/selinux/config

⚠️ 安全警告：

| 服务               | 禁用风险                     | 安全替代方案                         |
| ------------------ | ---------------------------- | ------------------------------------ |
| **firewalld**      | 暴露所有端口，易受网络攻击   | 配置精确防火墙规则                   |
| **SELinux**        | 降低系统安全性，增加提权风险 | 设置为 `permissive` 模式仅记录不拦截 |
| **NetworkManager** | 可能导致服务器网络配置混乱   | 使用 `network-scripts` 静态配置      |





## 挂载

为什么要学挂载？

类似于银行，工作环境是不能联网的 ，需要读取数据---U盘

挂载概念（操作磁盘/操作分区）

linux 没有盘符设置 如何设置好分区 给分区再设置一个入口

**目录和分区磁盘建立关联过程 称为挂载**

```shell
[root@koko ~]# cd /mnt
[root@koko mnt]# ls
[root@koko mnt]# touch b.txt
[root@koko mnt]# ls
b.txt

# 将光驱挂载到/mnt下，挂载后的/mnt是新的内容，之前的文件不能再查看

[root@koko mnt]# mount /dev/cdrom /mnt
mount: /dev/sr0 写保护，将以只读方式挂载

# 此时 /mnt 已被光盘内容覆盖，原文件 b.txt 被隐藏

[root@koko mnt]# ll
总用量 0
-rw-r--r--. 1 root root 0 7月   3 19:11 b.txt
[root@koko mnt]# cd
[root@koko ~]# cd /mnt
[root@koko mnt]# ls
CentOS_BuildTag  EFI  EULA  GPL  images  isolinux  LiveOS  Packages  repodata  RPM-GPG-KEY-CentOS-7  RPM-GPG-KEY-CentOS-Testing-7  TRANS.TBL

[root@koko mnt]# umount /mnt	
umount: /mnt：目标忙。
        (有些情况下通过 lsof(8) 或 fuser(1) 可以
         找到有关使用该设备的进程的有用信息)
[root@koko mnt]# cd ..
[root@koko /]# umount /mnt
[root@koko /]# cd /mnt
[root@koko mnt]# ls
b.txt

# 此时 b.txt回来了
```





## 网卡操作

1、查看eth0网卡信息

```shell
cat /etc/sysconfig/network-scripts/ifcfg-eth0
```

2、查看ens33网卡信息

```shell
cat /etc/sysconfig/network-scripts/ifcfg-ens33
```

企业中网卡名 ethx eth0 eth1 ...ethN

3、更改ens33网卡信息

vi /etc/sysconfig/network-scripts/ifcfg-ens33

```shell
[root@koko mnt]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet	# --使用网络类型：以太网：Ethernet
PROXY_METHOD=none 
BROWSER_ONLY=no
BOOTPROTO=static # --地址分配的方式：dhcp（自动获取地址）/none（static）（手工配置地址）
IPADDR=10.0.0.5	 # 网卡 --ipv4地址配置
NETMASK=255.255.255.0 # --网卡子网掩码配置
GATEWAY=10.0.0.254 # --配置可以访问外网的地址(可以实现访问外网)
DNS1=223.5.5.5          # 主DNS（推荐阿里公共DNS） --网卡DNS地址配置(可以访问域名)
DNS2=114.114.114.114    # 备用DNS（全网通用）
DEFROUTE=yes	# --最下面有个网关地址 yes 就是可以使用GATEWAY=10.0.0.1 --是否激活网关地址功能（可以实现访问外网）
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33		# --通过唯一识别码,可以识别虚拟主机的硬件信息
DEVICE=ens33	# --两个差不多  定义网卡名称信息  要保持一致
UUID=27a73818-89f3-4991-9890-2da7924a73c4
ONBOOT=yes		# --如果关闭网卡的话  状态改为no --使网卡默认开机启动(激活)
```

4、重启网卡

ps:如果要修改相关配置 记得**重启网卡**使之生效

**systemctl restart network**



## 修改网卡名字

1、修改文件内容 网卡ens33 改成eth0

​	vim  /etc/sysconfig/network-scripts/ifcfg-ens33

​	NAME=eth0

​	DEVICE=eth0

2、修改/etc/sysconfig/network-scripts/ifcfg-ens33为/etc/sysconfig/network-scripts/ifcfg-eth0

​	mv  ifcfg-ens33  ifcfg-eth0  

3、修改/etc/default/grub 加入net.ifnames=0 biosdevname=0

`net.ifnames=0`：禁用可预测网卡命名（ens33、eno16777728 等）

`biosdevname=0`：禁用基于 BIOS 的命名

```shell
[root@koko network-scripts]# cat /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

```shell
[root@koko network-scripts]# vi /etc/default/grub

[root@koko network-scripts]# cat /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet net.ifnames=0 biosdevname=0"
GRUB_DISABLE_RECOVERY="true"

```

4、执行 命令:   grub2-mkconfig -o /boot/grub2/grub.cfg

这一步是生成新的引导配置文件 `/boot/grub2/grub.cfg`

它会读取 `/etc/default/grub`，并把我们上一步加的参数写入实际使用的 grub 配置中

```shell
[root@koko network-scripts]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-a1b92567c14f40d98c588175eafe98f5
Found initrd image: /boot/initramfs-0-rescue-a1b92567c14f40d98c588175eafe98f5.img
done

```

5、重启 reboot

6、sshd –t 



## 域名解析相关配置

你提到的内容涉及域名解析的两个关键配置文件，以及与安全相关的 **DNS域名欺骗攻击（DNS Spoofing）** 问题，以下是详细解释：

------

### 一、域名解析配置文件说明

### 1. `/etc/resolv.conf` —— DNS客户端配置文件（**外网域名解析**）

这个文件用于配置系统在访问外部网络（如百度、谷歌）时，通过哪个 **DNS服务器** 来解析域名。示例：

```bash
[root@xiaoX ~]# cat /etc/resolv.conf
# Generated by NetworkManager
search com
nameserver 223.5.5.5
```

- `search com`：默认补全域名（可忽略）
- `nameserver 223.5.5.5`：使用的是 **阿里云公共DNS服务器**，它可以把 `www.baidu.com` 转换为对应的 IP 地址。
- **注意**：这个 IP 不能写错，否则会导致无法访问外网域名。

------

### 2. `/etc/hosts` —— 本地静态域名映射表（**内网/测试用**）

用于**手动指定某个域名对应的IP地址**，不经过DNS服务器。优先级高于 DNS 查询。

```bash
[root@xiaoX ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

你可以加上自定义项，例如：

```bash
192.168.10.20   webserver.intranet.local
```

以后你在浏览器中访问 `webserver.intranet.local`，系统就会直接找 `192.168.10.20`，无需DNS服务器参与。

------

### 二、异常解析 —— DNS欺骗攻击（DNS Spoofing）

### 背景

在早期网络时代，大多数网站使用的是 **HTTP** 协议，数据明文传输，域名解析和数据内容都可以被中间人伪造。这就带来了**DNS欺骗攻击**的风险：

### 攻击原理（简要流程）：

1. 用户访问 `http://www.bank.com`
2. 攻击者伪造DNS响应，让用户以为 `www.bank.com` 的IP是攻击者服务器的IP
3. 用户访问了假的网站（长得很像真实的银行页面）
4. 用户输入用户名和密码，被攻击者盗取

这种攻击手法被称为 **DNS spoofing**（DNS域名欺骗），也叫 **中间人攻击的一种手段**。

------

### 三、HTTPS的作用 —— 防止DNS欺骗

现在很多网站都启用了 **HTTPS**（加密的超文本传输协议），可以防止上述攻击：

### HTTPS如何防护：

1. 浏览器访问 `https://www.bank.com` 后，会验证该网站的 **SSL证书**
2. SSL证书中包含了域名的合法身份、颁发机构、有效期等信息
3. 如果证书域名不匹配或证书不可信，浏览器就会报错（证书不受信、伪造站点）
4. 通过加密握手，数据内容也被加密，即使DNS被篡改，攻击者也很难伪造一个有效证书

------

### 四、总结：操作与安全对比

| 配置文件           | 用途                | 场景           | 风险             | 安全防护               |
| ------------------ | ------------------- | -------------- | ---------------- | ---------------------- |
| `/etc/resolv.conf` | 配置DNS服务器       | 外网访问解析   | DNS被劫持        | 使用可信DNS + HTTPS    |
| `/etc/hosts`       | 本地域名映射        | 内网或测试用途 | 配错可能影响业务 | 谨慎修改，配合IP白名单 |
| HTTP               | 明文传输            | 老旧网站       | 易遭DNS劫持      | 使用HTTPS替代          |
| HTTPS              | 加密传输 + 身份校验 | 新网站标准     | 基本防御DNS欺骗  | 合法证书+浏览器校验    |

------

如果你在公司或学习中对网络安全方向感兴趣，还可以了解：

- DNSSEC（DNS安全扩展）
- HSTS（强制HTTPS）
- 防火墙+DNS缓存投毒检测







## zip压缩方式

 

-r 递归地将一个目录及其所有子目录和文件压缩到ZIP文件中

-q 在压缩文件时启用静默模式，即不显示压缩过程的详细信息

-d 从现有的ZIP文件中删除指定的文件或目录

-u 用于更新现有的ZIP文件，将新的文件或修改后的文件添加到ZIP存档中

-f 用于刷新（更新）现有ZIP文件中的指定文件。

-m 用于移动（归档）文件到一个ZIP压缩文件中，并在移动后将源文件删除。

-e 用于对ZIP压缩文件进行加密。

-z 为压缩文件添加注释

 

zip -r 压缩包名.zip 文件1 文件2 文件n

 

zip -r /home/test.zip /home/test

 

压缩test目录下的文件，排除test01目录 -x 指定排除目录，注意没有双引号将不起作用。

zip -r test.zip /home/test -x "/home/test/test01/*"

 

zip -r test.zip /home/test/ -e

之后输入密码 

 

 zip -r test.zip test/ -z 

添加注释信息。添加完成后输入回车后输入 . 之后再输入回车来结束

## 文件权限

![img](file:///C:\Users\20759\AppData\Local\Temp\ksohtml19536\wps1.png)

文件的权限针对三类对象进行定义

 

owner 属主，缩写u 

 

group 属组，缩写g

 

other 其他，缩写o

 

所有人 a 

权限表达的含义:

r  读权限 4

针对文件：利用此权限可以看到文件中的内容  cat

针对目录：可以看到目录下有什么数据  ls/目录

w 写权限 2

针对文件：利用此权限可以编辑文件中的内容  vim  echo> >>

针对目录: 利用此权限可以在目录中创建  删除文件信息  修改文件名称信息

x 执行权限 1

针对文件:利用此权限 执行脚本信息

针对目录:利用此权限可以切换到目录中

 

另 X：针对目录加执行权限，文件不加执行权限（因文件具备执行权限有安全隐患）

 

赋权 

 

没权限:0---------:

对于普通用户  无法读 写 执行文件或目录信息

对于皇帝root 读 写操作不会有影响  执行权限是有限制的

 

企业真实场景权限配置:

默认配置:

文件权限: 644  属主拥有读和写写权限  属组和其他用户只有读权限

目录权限: 755  属主拥有查看编辑进入权限 属组和其他用户只有查看进入权限

严格权限:

文件权限: 600 只有属主有读和写权限

目录权限: 700 只有属主有读写和进入权限

 

chmod -R命令表示将某个目录下面所有的内容都增加相应权限

 

 



## 负载均衡案例

```shell
窗口1
# CPU 压力测试
stress --cpu 2 --timeout 600
# I/O 压力测试
stress --io 10 --timeout 600
# 高并发 CPU 压力测试
stress -c 8 --timeout 600

窗口2
# 系统负载检查
uptime
# 实时负载监控
watch uptime

窗口3
# 详细 CPU 性能分析
mpstat -P ALL 5

```















































