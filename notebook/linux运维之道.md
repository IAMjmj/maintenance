### 无人职守自动安装linux操作系统
____________________________
* PXE技术 -- 客户端通过网络从远端服务器下载启动镜像，整个过程要求服务器分配IP地址，再用TFTP协议下载位于服务器上的启动镜像到本机内存中执行，客户端从网络启动后，剩下的服务器配置步骤仍然需要手动完成，为此需要使用kickstart技术通过读取自动应答文件来实现自动安装配置。
* kickstart可以通过 system-config-kickstart 图形界面工具自动配置生成。
* 安装服务器需要运行DHCP、TFTP、NFS三种服务。
* DHCP服务器 -- 为客户端自动分配IP和其它参数(next -> TFTP， configure /etc/dhcp/dhcpd.conf)。
* TFTP -- 简单文件传输服务(configure /etc/xinetd.d/tftp)，CentOS7使用vsftpd软件来安全地共享文件，客户机通过服务器的vsftpd来传输系统文件。

#### 自动化安装案例
* 准备两台安装服务器，boot.example.com(DHCP/TFTP)，ftp.example.com(FTP)
boot主机上安装DHCP服务，并配置好dhcp启动项，启动dhcp并设置为开机自启动，关闭防火墙，关闭SELinux防护功能。
```sh
# 安装dhcp
yum -y install dhcp
# 配置dhcp
vim /etc/dhcp/dhcpd.conf
# 启动和开机启动设置
systemctl start dhcpd
systemctl enable dhcpd
# 查看端口
netstat --nutlp | grep :67
# 关闭禁用防火墙
systemctl stop firewalld
systemctl disable firewalld
# 关闭SELinux防护功能
setenforce 0
```

* boot主机上安装tftp服务器和xinetd，编辑配置文件/etc/xinetd.d/tftp，启动tftp，配置共享路径。
```sh
yum -y install tftp-server xinetd
vim /etc/xinetd.d/tftp
```

* boot主机上将客户端所需的引导文件和镜像复制到tftp共享目录。
```sh
# 获得引导文件
yum -y install syslinux
# 引导文件复制到tftp共享目录
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
# 挂在启动镜像并复制文件
umount /dev/cdrom
mount /dev/cdrom /var/ftp/pub
cp /var/ftp/pub/isolinux/* /var/lib/tftpboot/
# 创建启动配置文件
mkdir /var/lib/tftpboot/pxelinux.cfg/
cp /var/ftp/pub/isolinux/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default
chmod 644 /var/lib/tftpboot/pxelinux.cfg/default
```

* 修改相应的安装启动配置文件，制定kickstart启动文件
```sh
vim /var/lib/tftpboot/pxelinux.cfg/default
```
* 重启tftp服务并设置为开机启动
```sh
systemctl restart xinetd
systemctl enable xinetd
ss -nutlp | grep :69
```

* 创建kickstart自动应答文件，使用system-config-kickstart软件进行图形化配置

### 命令工具
___________

#### 基本命令

* 复制
```sh
# 递归复制
cp -r source target
# 保留源文件的所有属性
cp -a source target
```

* 删除
```sh
# 删除提示
rm -i target
# 递归删除
rm -r target
```

* 搜索文件或目录
```sh
## find [命令选项] [路径] [表达式选项]
#
# 当前路径下的hello.doc
find -name hello.doc
# /root目录下的以.log结尾的文档
find /root -name "*.log"
# 不区分大小写
find -iname "Java.doc"
# 查找计算机中的所有空文档
find / -empty
# 查找所属组为tom的文档
find / -group tom
# 查找计算机中所有在3天内被修改过的文档
find / -mtime -3
# 查找计算机所有在4天前被修改过的文档
find / -mtime +4
# 查找计算机中2天前当天被修改过的文档
find / -mtime 2
# 查找当前目录下大于4M的文档
find ./ -size +4M
# 查找当前目录下的所有普通文件
find ./ -type f
# 查找用户tom的所有文件
find / -user tom
# 使用exec对找到的文件执行特定的命令
find / -size +4M -exec ls -al
```

* 计算文件或目录的占用量
```sh
# 统计
du -s path
# 人性化显示
du -h
```

* 统计文件的行、字节和单词信息
```sh
# 字节
wc -c file
# 行
wc -l file
# 单词数
wc -w file
```

* 查找关键词并打印匹配的行
```sh
# 过滤包含关键字的那行
grep th test.txt
# 高亮匹配的关键字
grep --color th test.txt
# 大小写不敏感
grep -i th test.txt
# 反向匹配不包含关键字的行
grep -v th test.txt
```

* 压缩和解压
```sh
# 打包目录
tar -cf etc.tar /etc
# 打包并压缩 -- gzip
tar -czf boot.tar.gz /boot
# 打包并压缩 -- bzip
tar -cjf boot.tar.bz2 /boot
# 从打包文件中删除某个文件
tar --delete etc/hosts -f etc.tar
# 追加文件至压缩包
tar -f etc.tar -f /root/install.log
# 查看打包文档信息
tar -tf boot.tar.gz
# 查看打包文档详细信息
tar -tvf etc.tar
# 解压bz2格式的压缩包
tar -xjf etc.tar.bz2
# 解压gz格式的压缩包
tar -xzf etc.tar.gz
# 指定解压路径并解压
tar -xzf boot.tar.gz -C /tmp
# 压缩后删除源文件
tar -czf mess.tar.gz /var/log/messages --remove-files
```

#### 账户与安全
>linux对账户和组的管理是通过id号来实现的  
>uid -- 用户id  
>gid -- 组id  

* 创建新账户 -- useradd
* 创建组账户 -- groupadd
* 显示账户和组信息 -- id
* 更新账号认证信息 -- passwd
```sh
# 修改tom的密码
passwd tom
# 从管道读取数据设置密码
echo "yw020154" | passwd --stdin tom
# 锁定账户
passwd -l tom
# 解锁账户
passwd -u tom
# 清除密码
passwd -d tom
```
* 修改账户信息 usermod
```sh
# 修改home目录
usermod -d /home/tomcat tom
# 修改账户失效日期
usermod -e 2018-08-01 tom
# 修改账户所属基本组
usermod -g mail tom
# 修改账户所属附加组
usermod -G mail tom
# 修改账户登录shell
usermod -s /bin/zsh
# 修改账户uid
usermod -u 1001 tom
```
* 删除账户 userdel
* 删除组 groupdel
* 更改文件或目录全新啊 chmod
* 更改所有者和所属组 chuser

#### 存储管理

* 磁盘分区  
传统的MBR分区方式，一块硬盘最多四个主分区，如果需要划分多个分区可以采用3个主分区和一个扩展分区的方案，其中扩展分区再划分其它的逻辑分区。  
```sh
# 显示磁盘信息
fdisk -l
# 对某个磁盘分区
fdisk /dev/sd*
# 让内核立即读取新的分区表，无需重启即可识别新创建分区
partprobe /dev/sdb
# mgr无法创建大于2TB的分区，修改分区表类型为gpt
parted /dev/sdc mklabel gpt
# 创建分区 -- 从磁盘的1M开始到2G的格式为ext3的主分区
parted /dev/sdc mkpart primary ext4 1 2G
# 查看分区表信息
parted /dev/sdc print
# 删除分区 -- 分区号2
parted /dev/sdc rm 2
```
* 格式化和挂载文件系统
```sh
# 格式化
mkfs.xfs /dev/sdc1
# 创建交换空间
mkswap /dev/sdc2
# 挂载到目录
mkdir /detail
mount /dev/sdc1 /detail
# 卸载设备
umount /devsdc1
# 开机启动自动挂载 -- 编辑/etc/fstab文件添加以下内容:
/dev/sdc1 /detail xfs defaults 0 0
# 立即读取/etc/fstab文件并挂载所有设备
mount -a
```
* LVM逻辑卷  
允许用户动态调整文件系统的大小。
```sh
# 使用LVM对磁盘或分区进行初始化
pvcreate /dev/sdc4 /dev/sde
pvcreate /dev/sd{1,2,3}
# 创建卷组
vgcreate test_vg1 /dev/sdb5 /dev/sdb6
# 创建卷组 -- 指定pe大小
vgcreate test_vg2 -s 16M /dev/sdc5 /dev/sdc6
# 从卷组中提取存储空间创建逻辑卷
lvcreate -L 2G -n test_lv1 test_vg1 # 提取2G
lvcreate -l 100 -n test_lv2 test_vg2  # 提取100个pe的大小
lvcreate -L 2G -n test_lv3 test_vg1 /dev/sdb6 # 指定物理卷提取
```

#### 网络配置

```sh
# # 网络接口参数
ifconfig interface 选项 | 地址
# 配置网卡地址和子网掩码
ifconfig en0s3 10.0.6.231 netmask 255.255.255.0
# 查看网卡信息
ifconfig en0s3
# 关闭网卡
ifconfig en0s3 down
# 开启网卡
ifconfig en0s3 up
# # 主机名
hostnamectl status
hostnamectl set-hostname nojsja
# # route 显示或设置静态IP路由表
route [选项] # 查看路由信息
route add [ip] gw [gateway] # 添加路由表记录
route # 删除路由表记录
# 添加默认网关为10.0.6.1
route add(del) default gw 10.0.6.1
# 添加指定网段的网关
route add(del) -net 172.16.0.0/16 gw 10.0.6.1
# 指定通过网卡传输到192.56.76.0网段的数据
route add(del) -net 192.56.76.0 netmask 255.255.255.0 dev en0s3
```

#### 内核模块
```sh
# # 内核存放位置
uname -r # 当前内核
ls /lib/modules/`uname -r`
# # 内核模块
lsmod # 查看内核模块
modprobe ip_vs # 动态加载内核模块
modprobe -r ip_vs # 动态卸载内核模块
modinfo ip_vs # 查看内核模块信息
# # 动态修改内核参数
echo "1" > /proc/sys/net/ipv4/ip_forward # 开启内核路由转发功能
echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_all # 禁止所有的icmp回包(禁止其它主机的ping功能)
echo "108248" > /proc/sys/fs/file-max # 调整所有进程可以打开的文件总数量
```


### 自动化运维
____________

#### Bash
```sh
# 调用历史命令 -- word关键字
!word
# 调用历史命令 -- n历史记录编号
!n
# 调用历史命令 -- 使用快捷搜索
命令行：ctrl + r
# /dev/null -- 无用信息删除
passwd > /dev/null
# 使用控制字符控制命令的执行方式
(; && || &)
# 花括号使用技巧
echo {a, b, c}
echo user{1,5,8}
echo {0..10}
echo {0..10..2}
echo a{2..-1}
# # 变量的展开替换
# 如果varname存在且非null，则返回其值，否则返回word
${varname:-word}
# 如果varname存在且非null，则返回其值，设置为word
${varname:=word}
# 如果varname存在且非null，则返回其值，否则显示varname:word
${varname:?word}
# 如果varname存在且非null，则返回word，否则返回null
${varname:+word}

# 数组
array=(1 2 3 4 5)
array[0]=9
${array[0]}

# 算数运算
$((x+y))
$((x-y))
$((x/y))
$((x*y))
$((x%y))
$((x++))
$((x--))
$((x**y)) # 次方

# expr 运算工具
expr p1 + p2
expr p1 - p2
expr p1 \* p2
expr p1 / p2
expr p1 % p2

# # test测试命令 和 测试表达式 []
#
# FILE是否存在且为目录
-d FILE
# 文件是否存在且为普通文件
-f FILE
# 文件是否存在且可写
-w FILE
# 文件是否存在且非空
-s FILE
# 字符串长度非零
-n STRING
# 字符串相等
STRING1 = STRING2
# 整数1与整数2相等
INT1 -eq INT2
# 整数1大于整数2
INT1 -gt INT2
# 整数1小于整数2
INT1 -lt INT2

# # shell符号
# 正斜线和反斜线
/ # 命令继续
\ # 命令转义
# 单引号解释字面意义
# 双引号不会屏蔽 ` $ \ 的意义
# 反引号将命令字符替换为命令输出的结果
echo `passwd`
```

#### 正则表达式
```sh
# # 基本字符
#
# 匹配任意字符
.
# 匹配一个字符出现0次或多次
*
# 匹配任意多个任意字符
.*
# 匹配字符集中的任意一个字符
[]
# 匹配连续的字符串范围
[x-y]
# 匹配字符串的开头
^
# 匹配字符串的结尾
$
# 取非集
[^]
# 转义
\
# 匹配一个字符出现n到m次
\{n, m\}
# 匹配至少出现n次
\{n,}
# 匹配一个字符出现n次
\{n}
# 将\(\)之间的内容存储在“保留空间”，最大存储9个
\(\)
# 通过\1至\9调用保留空间中的内容
\n

# # 正则案例
grep root /etc/passwd
grep --color :..0 /etc/passwd
grep --color 00* /etc/passwd
grep --color 0[os]t /etc/passwd
grep --color [0-9] /etc/passwd
grep --color [f-q] /etc/passwd
grep --color ^root /etc/passwd
grep --color root$ /etc/passwd
grep --color sbin/[^n] /etc/passwd
grep --color '0\{1,2\}' /etc/passwd  # 0出现1、2次的行
grep --color '\(root\.*\1)' /etc/passwd # 匹配包含两个root的行
grep --color '\(root\)\(:\).*\2\1' # 匹配以root:开头和:root结尾的行
grep --color ^$ /etc/passwd # 过滤文件的空白行
grep --color -v ^$ /etc/passwd # 过滤文件的非空白行

# # 扩展正则表达式
#
# 基本字符
{n,m} # 等同于\{n,m\}\
+ # 匹配字符出现一次或多次
? # 匹配字符出现零次或一次
| # 匹配逻辑或者
() # 匹配正则集合

# # POSIX规范
#
# 基本字符
[:alpha:] # 字母字符
[:alnum:] # 字母和数字字符
[:cntrl:] # 控制字符
[:digit:] # 数字字符
[:xdigit:] # 16进制数字字符
[:punct:] # 标点符号
[:graph:] # 非空格字符
[:print:] # 任何可以显示的字符
[:space:] # 任何可以产生空白的字符
[:black:] # 空白和Tab字符
[:lower:] # 小写字符
[:upper:] # 大写字符

# # GNU规范
#
# 基本字符
\b # 边界字符
\B # 非边界字符

```
