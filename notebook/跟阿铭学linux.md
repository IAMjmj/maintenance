### 安装centos
_____________

#### 设置网络
1. 自动获取ip地址
```sh
dhclient
```
2. 查看ip地址
```sh
# IP 命令
ip addr
# 安装网络工具
yum install -y net-tools
ifconfig
# ping 测试
ping -c 4 www.baidu.com
```
3. 手动配置IP
```sh
# 静态IP配置需要手动增加几行
# ip地址、子网掩码、网关、DNS
IPADDR=192.168.199.*
NETMASK=255.255.255.0
GATEWAY=192.168.199.*
DNS1=119.29.29.29
```
>静态IP需要修改ONBOOT=yes BOOTPROTO=static，
配置文件地址：vim /etc/sysconfig/network-scripts/ifcfg-en**

### Linux文件和目录管理
____________________

1. 文件操作命令
```sh
# 将新目录加入系统路径
PATH=$PATH:/some/path
# 文件查看命令
cat tac
more less
head tail
```

2. 更改文件所属组chgrp
```sh
groupadd testgroup
chgrp testgroup testfile
# 递归更改
chgrp -R testgroup testfile
```

3. 更改文件的所属者chown
```sh
useradd testuser1
chown testuser1 testfile
# 同时递归更改用户和组
chown -R testuser:testgroup testfile
```

4. 更改文件的权限chmod
```sh
# 权限：读写执行 -- r/w/x -- 4/2/1
# 对象：所有/用户/组/其它用户 -- a/u/g/o
#
chmod -R 700 target # 数字更改权限
chmod -R u=rwx,og=rx target  # 字母更改权限
chmod -R u-x target # 字母减少权限
chmod -R a+x target # 字母所有增加权限
```

5. 更改文件的默认权限umask
```sh
# umask -- 200
# 新文件预设权限666，新目录预设权限777
# umask的值表示 默认权限为预设权限减去这个值后的权限
# 权限计算不能单独做数学减法，比如666 - 033 != 633，因为可能有重复计算值
#
sudo vim /etc/bashrc
```

6. 修改文件的特殊属性chattr
```sh
# chattr [+-=][Asaci]
# a -- 只能追加不能删除
# i -- 不能更改和删除
#
chattr +i dir # 给目录新增i权限后即使是root用户也不能再dir内创建和删除文件
chattr +a dir # 给目录a权限后只能再dir内新建文件，不能删除文件
#
# 递归列出所有文件的特殊属性
lsattr -aR ./
```

7. 第四位特殊属性(rwx之外)
```sh
# set uid -- 二进制可执行文件在执行期间拥有文件所有者的权限 -- passwd
# set gid -- 针对文件和目录，功能同set uid
# sticky bit -- 防删除位，文件是否能被删除取决于该文件所属目录对该用户是否有w权限，如果
# 用户有w权限，但是要让他不能删除该目录下的其它用户文件，则可以设置此权限。
#
ls -l /usr/bin/passwd
```

8. 查找和定位文件
```sh
which # 查找文件的绝对路径
whereis  # 模糊查找
locate # 模糊查找所有目录和文件
find # 最常用的查找命令 find [路径] [参数]
```

### 用户和用户组管理
_________________

1. 用户身份切换su
>su [-] username, 加-时同时切换到该用户目录，并且会初始化当前用户的环境变量，等同于以该用户登录。

2. 临时切换到root身份sudo
>使用visudo编辑/etc/sudoers配置文件添加 $user ALL=(ALL) ALL就可以让$user用户拥有切换sudo权限。

3. 不允许远程登录root账户
>编辑配置文件/etc/ssh/sshd_config，将#PermitRootLogin 值修改为no

### 磁盘管理
__________

#### 查看磁盘或目录的容量

1. 查看磁盘分区容量df
```sh
# 查看inodes的使用情况
df -i
# 自动选择合适的单位
df -h
# 手动设置统计单位
df -[mk]
```

2. 查看目录和文件占用大小du
```sh
du -[abckmsh]
# 显示所有目录和文件空间占用大小，默认单位KB
du -a
# 设置单位bkmh -- B/KB/MB/auto
du -[bkmh]
# 所有信息之后增加总占用量统计
du -c
# 只统计总占用
du -s
```

#### 磁盘的分区和格式化

1. fdisk -l 查看分区情况

2. fdisk /dev/sd* 对磁盘进行分区

3. 分区后格式化磁盘分区  
mkesfs/mkfs.ext2/mkfs.ext3/mkfs.ext4/mkfs.xfs

#### 磁盘的挂载和卸载

1. 查看挂载 mount

2. 挂载 mount /dev/sd* /dir

3. 系统挂载配置文件 /etc/fstab

4. mount -a 自动挂载fstab配置文件里声明的设备

5. 获取分区UUID -- blkid  
让分区再开机之后自动挂载可以：  
1) 添加/etc/fstab配置文件  
2) 将挂载命令添加到/etc/rc.d/rc.local文件中去
```sh####
# /etc/rc.d/rc.local
#
# 可以使用UUID和Label
/usr/bin/mount UUID="f8fa9bcd-ae3e-40e5-8bd9-7b3872259d03" /dir
```
3) 赋予可执行权限 chmod a+x /etc/rc.d/rc.local  

6. umount -l 强制卸载

7. 建立swap分区增加虚拟内存
>建立swapfile -> 格式化为swap格式 -> 启动虚拟磁盘

```sh
# 建立swap文件 -- if检测存在 of目标文件 bs块的大小 count块数量
dd if=/dev/zero of=/tmp/dir bs=1M count=8096
# 格式化为swap格式
mkswap -f /tmp/dir
# 查看内存大小
free -m
```

### 文本编辑工具vim
_________________

#### 一般模式
>此模式下可以复制、粘贴、删除  

```sh
# 上下左右
k j h l
# 下一页
ctrl + f
# 上一页
ctrl + b
# 行首 和 行尾
0 $
# 第一行
gg
# 去第n行
nG
# 最后一行
G
# 向后和向前删除字符
x X
# 向后删除n个字符
nx
# 删除/剪切光标所在
dd
# 向后删除n行
ndd

```

#### 编辑模式
```sh
# # 由一般模式进入编辑模式
#
# 在当前字符插入
i
# 在鼠标所在的行首插入
I
# 在当前字符后插入
a
# 在鼠标所在的行尾插入
A
# 在当前行的下一行插入新的一行
o
# 载当前行的上一行插入一行
O
```

#### 命令模式
```sh
# 在光标之后查找一个字符，按n向后继续搜索
/word
# 在光标之前查找一个字符，按n继续向前查找
?word
# 在n1和n2行之间查找所有的word1并替换为wor2，不加g则只替换第一个
:n1,n2s/word1/word2/g
# 将所有word1替换为word2，不加g则只替换每一行的第一个word1
:1,$s/word1/word2/g
# 显示行号
:set nu
# 不显示行号
:set nonu
```

### 安装RPM包或源码包
_________________

#### RPM工具
```sh
# # 安装RPM包
# i -- 表示安装
# v -- 可视化
# h -- 显示安装进度
rpm -ivh *.rpm
# # 升级RPM包
rpm -Uvh pkg
# # 卸载RPM包
rpm -e pkg
# # 查询一个包是否安装
rpm -q pkg
# # 查询所有安装的包
rpm -qa
# # 得到一个包的相关信息
rpm -qi pkg
# # 列出一个包的安装文件
rpm -ql pkg
# # 列出某个文件属于哪个RPM包
rpm -qf filename
```

#### yum工具
```sh
# # 列出所有yum资源
yum list
# # 搜索rpm包
yum search pkg
# # 安装yum包
yum install -y pkg
# # 卸载包
yum remove pkg
# # 升级包
yum update -y pkg
# # 利用yum工具下载RPM包
yum install pkg -y --downloadonly --downloaddir=/tmp/
# # 在包已经安装的情况下下载RPM包
yum reinstall pkg -y --downloadonly --downlaoddir=/tmp/

```

#### 安装源码包
```sh
# # 安装步骤
# 配置功能，检测编译所需的库是否完整，最后生成MakeFile文件
./configure
./configure --help
# 根据MakeFile进行编译
make
# 创建相关软件的存放目录和配置文件安装软件
make install
```

### Shell
_________

#### Shell基础

```sh
# 输出环境变量
env
set
# 设置临时变量
var=*
# 提升变量到子shell
export var=*
# 让所有用户永久都能访问变量
echo "export var=*" >> /etc/profile
source /etc/profile
# 仅允许当前用户使用该变量
echo "export var=*" >> ~/.bashrc
source ~/.bashrc
# 变量中使用其它命令
myname=`pwd`
# 变量内容累加其它内容
myname="johnson"Young
```

#### 系统环境变量和个人环境变量的配置文件

1. /etc/profile
>这个文件预设了几个重要的变量，例如PATH、USER、LOGNAME、MAIL、INPUTRC、HOSTNAME、HISTSIZE、umask等。  

2. /etc/bashrc
>这个文件主要预设umask和ps1。  

3. ~/.bash_profile
>该文件定义了用户的个人化路径和环境变量的文件名称，每个用户都可使用这个文件输入专属于自己的shell信息，用户登录时，该文件仅仅执行一次。  

4. ~/.bashrc
>该文件包含专属于自己的shell的bash信息，当每次登录或打开新的shell时，该文件会被读取。例如，你可以将用户自定义的别名或自定义变量写到这个文件中。

5. ~/.bash_history
>该文件记录历史命令。  

6. ~/.bash_logout
>当退出shell时会执行该文件，文件中可以放一些清理的工作。  

### 三神器 grep / sed / awk
____________

#### grep工具的使用
```sh
# # grep -[ cinvABC] 'word' filename
#
# 表示打印符合要求的行数
-c
# 忽略大小写
-i
# 输出符合要求的行以及行号
-n
# 打印不符合要求的行
-v
# 跟一个数字，例如A2表示打印符合要求的行及上面两行
-A2
# 跟一个数字，例如A2表示打印符合要求的行及下面两行
-B2
# 跟一个数字，例如C2表示打印符合要求的行和其上下两行
-C2

# # 实例
#
# 过滤带有某个关键字的行并输出行号
grep -n 'root' /etc/passwd
# 过滤出不带某个关键字的行并输出行号
grep -nv 'root' /etc/passwd
# 过滤出所有包含数字的行
grep '[0-9]' /etc/inittab
# 过滤出所有不包含数字的行
grep -v '[0-9]' /etc/inittab
# 过滤掉所有以#开头的行
grep -v '^#' /etc/inittab
# 过滤掉所有空行和以#开头的行
grep -v '^#' /etc/inittab | grep -v '^$'
# 过滤出任意一个字符、零个或多字任意字符
grep 'r.o' /etc/passwd
grep '.*' /etc/passwd
# 指定要过滤的字符出现次数
grep 'o\{2\}' /etc/passwd
grep 'o\{1,3\}' /etc/passwd
grep 'o\{1,\}' /etc/passwd

# # egrep工具的使用
#
# 过滤出一个或多个指定的字符
egrep 'o+' test.txt
egrep 'oo+' test.txt
# 过滤出零个或一个指定的字符
egrep 'o?' test.txt
egrep 'oo?' test.txt
# 过滤出字符串1或字符串2
egrep 'uuu|iii' test.txt
egrep 'r(oo|at)o' test.txt
```

#### sed工具的使用
```sh
# # sed -n 'n'p filename
# n表示第几行
# -n表示只显示要打印的行

# 指定一个区间打印
sed -n '1,3'p test.txt
# 打印包含某个字符串的行(使用正则表达式)
sed -n '/word/'p filename
# 使用-e参数可以指定多个条件
sed -e '1'p -e '/111/'p test.txt
# 删除某些行
sed '1'd test.txt
sed '1,3'd test.txt
sed '/root/'d test.txt
# 替换字符或字符串
sed '1,2s/ot/to/g' test.txt
# 删除文件中的所有字母
sed 's/[0-9]//g' test.txt
# 直接修改文件的内容
sed -i 's/ot/ro/g' test.txt
```

#### awk工具的使用
```sh
# # awk 兼具sed的所有功能
#
# 截取文档中的某个段
# -F 指定分隔符
# 命令要使用单引号，$n表示第n个分隔段落，$0表示所有分隔段落
head -n2 /etc/passwd | awk -F ':' '{print $1}'
# 自定义打印格式
head -n2 /etc/passwd | awk -F ':' '{print "#"$1"#"$2}'
# 匹配字符或字符串
awk '/oo/' test.txt
awk -F ':' '$1 ~/oo/' test.txt # 以分隔出的字符再进行二次匹配oo
# 多个匹配条件
awk -F ':' '/root/ {print $1,$3} /test/ {print $1,$3}' test.txt
# 条件操作符(> >= < <= != && ||)
awk -F ':' '$3=="0"' test.txt
awk -F ':' '$3>40' test.txt
awk -F ':' '$3>1000 && $4=="/bin/bash"' /etc/passwd
# awk内置变量
OFS NF NR
# # awk的数学计算
#
# awk改变某个段值
head -n3 /etc/passwd | awk -F ':' '$1="root"'
# 计算某个段的总和
awk -F ':' '{(tot=tot+$3)}; END {print tot}' /etc/passwd
# 条件语句
awk -F ':' '{if ($1=="root") {print $0}}' /etc/passwd
```

### linux防火墙
--------------
```sh
#
# # selinux ------------------------
# 限制太多，配置繁琐，一般会强制关掉
# 临时关闭selinux
setenforce 0
# 修改配置文件永久关闭 -- disabled
vim /etc/selinux/config
# 获取selinux状态
getenforce
#
# # netfilter -----------------------
# centos7之前的防火墙为netfilter，centos7使用firewalld
# 很多服务器仍然使用之前的iptables管理
# 这边先禁用firewalld
systemctl stop firewalld
systemctl disable firewalld
#
# # netfilter的五个链
# PREROUTING: 数据包进入路由表之前
# INPUT: 通过路由表后目的地为本机
# FORWARDING: 通过路由表后，目的地不为本机
# OUTPUT: 由本机产生，向外转发
# POSTROUTTONG: 发送到网卡接口之前
#
# # iptables ------------------------
# iptables规则文件/etc/sysconfig/iptables
# centos上默认没有iptables规则
# 查看规则
iptables -nvL
# 清除规则
iptables -F
# 保存iptables规则
service iptables save
# iptables规则选项
# -A/-D: 增加或删除一条规则
# -I: 插入一条规则
# -p: 指定协议--tcp/udp/icmp
# --dport: 跟-p一起使用，表示指定目标端口
# --sport: 跟-p一起使用，表示指定源端口
# -s: 表示指定源IP/IP段
# -d: 表示指定目标IP/IP段
# -j: 后面跟动作，其中ACCEPT表示允许包，DROP表示丢掉包，REJECT表示拒绝包
# -i: 表示指定网卡
#
# 实例:
# 插入一条规则，把来自1.1.1.1的所有数据包丢掉
iptables -I INPUT -s 1.1.1.1 -j DROP
# 删除上个规则，命令格式需一致
iptables -D INPUT -s 1.1.1.1 -j DROP
# 把来自2.2.2.2的协议为tcp，到本机80端口的数据包丢掉
iptables -I INPUT -s 2.2.2.2 -p tcp -dport 80 -j DROP
# 把发送到10.0.1.14的22端口的数据包丢掉
iptables -I OUTPUT -p tcp -dport 22 -d 10.0.1.14 -j DROP
# 让来自192.168.1.0/24这个网段且作用在网卡eth0上的包通过
iptables -A INPUT -s 192.168.1.0/24 -i eth0 -j ACCEPT
# 通过查看规则类型和规则编号删除规则
iptables -nvL --line-numbers
iptables -D INPUT 1
# 更改预设策略
iptables -P INPUT DROP # 预设禁用
iptables -P INPUT ACCEPT # 预设恢复
# nat表的应用
# 假设机器上有两块网卡eth0(10.0.2.68)和eth1(192.168.1.1)，eth0联网了，etho1没有，现在
# 有另一台机器(192.168.1.2)和eth1是互通的，那么如何设置才能让eth1的机器连上网并与10.0.2.68互通呢？
# 打开内核路由转发功能
echo '1' > /proc/sys/net/ipv4/ip_forward
# 让iptables对nat表做一个IP转发的操作
# MASQUERADED -- 伪装
# -o 跟设备名
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADED
#
# 保存和备份iptables规则
service iptables save
# 停止防火墙服务
service iptables stop
# 备份防火墙规则
iptables-save > myipt.rule
# 恢复防火墙规则
iptables-restore < myipt.rule
#
# # firewalld -------------------------
# CentOS7的防火墙
# 先关闭iptables，打开filrewalld
systemctl disable iptables
systemctl enable firewalld
systemctl start firewalld
# firewalld有两个基础概念，分别是zone和service，每一个zones有不同的iptables规则
# 默认一共9个zone，centos7默认zone为public
#
# 获取所有zone
firewall-cmd --get-zones
# 查看默认zone
firewall-cmd --get-default-zone
# 关于zone的说明
# drop -- 丢弃
# block -- 限制
# public -- 公共
# external-- 外部
# dmz -- 非军事区
# work -- 工作区
# home -- 家庭网络
# internal -- 内部网络
# trusted -- 信任网络
#
# 设定默认zone
firewall-cmd --set-default-zone=work
# 查看指定网卡所在的zone
firewall-cmd --get-zone-of-interface=ens33
# 给指定网卡设置zone
firewall-cmd --zone=dmz --add-interface=lo
# 针对网卡更改zone
firewall-cmd --zone=home --change-interface=lo
# 针对网卡移除zone
firewall-cmd --zone=home --remove-interface=lo
# 查看所有网卡所在的zone
firewall-cmd --get-active-zones
# # 每个zone包含不同的service，service就是针对一个服务(端口)所做的iptables规则
# service 配置文件 -- /usr/lib/firewalld/services <- /etc/firewalld/services
#
# 查看当前zone有哪些service
firewall-cmd --list-services
# 查看指定zone下有哪些service
firewall-cmd --zone=public --list-services
# 添加一个service到指定zone下(内存)
firewall-cmd --zone=public --add-service=http
# 添加一个service到指定zone下(永久)
firewall-cmd --zone=public --add-service=http --permanent
# 重新加载firewalld
firewall-cmd --reload
```

### linux系统服务管理
-------------------

#### chkconfig
centos6上的系统服务管理软件，centos7使用了systemd，但是也可以用chkconfig进行服务管理
```sh
# # chkconfig 所有注册服务配置文件
ls /etc/init.d
# # 列出所有服务以及每个级别的开启状态
chkconfig --list
# 运行级别
# 0 --关机shutdown
# 1 -- 重启至单用户模式
# 2 -- 无NFS支持的多用户模式
# 3 -- 完全功能的多用户模式
# 4 -- 用户自定义
# 5 -- 图形登录方式
# 6 -- 重启
# 指定某服务的某个级别关闭或开启
chkconfig --level 3 mongodb off/on
# 默认操作目标级别2/3/4/5
chkconfig mongodb off/on
# # 将自定义服务加入到系统服务
chkconfig --add mongodb
chkconfig --del mongodb
```

#### systemd服务管理
centos7使用systemd代替chkconfig
```sh
# # 列出系统所有服务
systemctl list-units --all --type=service
# # 服务对应的脚本文件
ls /usr/lib/systemd/system
# # 常用操作
systemctl enable [service-name] # 开机自启
systemctl disable [service-name] # 禁止开机自启
systemctl status [service-name] # 查看服务状态
systemctl start [service-name] # 启动某个服务
systemctl stop [service-name] # 停止某个服务
systemctl restart [service-name] # 重启某个服务
systemctl is-enabled [service-name] # 查看某个服务是否开机自启
#
# # 关于service / unit / target
# unit类型
# service -- 系统服务
# target -- 多个unit组成的组合
# device -- 硬件设备
# mount -- 文件系统挂载点
# automount -- 自动挂载点
# path -- 文件或路径
# scope -- 不是由systemd启动的外部进程
# slice -- 进程组
# snapshot -- systemd快照
# socket -- 进程间通信的套接字
# swap -- 交换文件
# timer -- 定时器
#  
# # unit相关的命令
systemctl list-units # 列出正在运行的unit
systemctl list-units --all # 列出所有的unit
systemctl list-units --all --state=inactive # 列出所有未激活的unit
systemctl list-units --all --type=service # 列出所有serivce
systemctl list-units --type=servcie # 列出所有激活的service
systemctl is-active crond.service # 查看某个服务是否激活
# # target
systemctl list-unit-files --type=target # 列出所有target
systemctl list-dependencies multi-user.target # 列出target的所有依赖
systemctl get-default # 查看系统的默认target
systemctl set-default multi-user.target # 设置系统默认的target
# # target/unit/service关系
# service属于一种unit
# 多个unit组成target
# 一个target包含多个service
```

#### 数据备份工具rsync
>remote sync 可以同步本地数据和远程数据，它不会覆盖已有的数据，而是先判断目标目录的数据和新数据的差异，只有数据不同的时候才会把不相同的部分覆盖。  

命令格式:   
 * rsync [OPTION] SRC DST  
 * rsync [OPTION] SRC [USER@]HOST:DST  
 * rsync [OPTION] [USER@]HOST:SRC DEST  
 * rsync [OPTION] SRC [USER@]HOST::DST  
 * rsync [OPTION] [USER@]HOST::SRC DEST  

选项说明：  
* -a => 归档模式，以递归方式传输文件并保留属性，相当于 -rlptgoD，可以跟一个--no-[OPTION]来动态生成参数，比如-a--no-l 等同于 -rptgoD。  
* -r => 递归处理目录
* -v => 打印信息
* -l => 保留软链接
* -L => 根据链接来传输源文件
* -p => 保留文件权限
* -o => 保留属主信息
* -g => 保留组信息
* -D => 保留设备文件信息
* -t => 保留时间信息
* --delete => 删除DST中SRC没有的文件
* --exclude=[PATTERN] => 排除文件
* --progress => 显示进度信息
* -u => 不覆盖DST中较新的文件
* -z => 传输过程中压缩
