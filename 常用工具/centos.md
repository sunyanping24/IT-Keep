<!-- TOC -->

- [防火墙](#防火墙)
  - [区域Zone](#区域zone)
  - [常用命令](#常用命令)
    - [1、`systemctl` [参数] `firewalld`](#1systemctl-参数-firewalld)
    - [2、`firewall-cmd` [参数]](#2firewall-cmd-参数)
- [常用的Shell命令](#常用的shell命令)
- [查看系统相关信息命令](#查看系统相关信息命令)

<!-- /TOC -->

# 防火墙

> Centos7的防火墙由CentOS6的iptables升级而来。防火墙的配置在`/etc/firewalld/`和`/usr/lib/firewalld`目录下。

**系统配置`/usr/lib/firewalld`目录**下中存放定义好的网络服务和端口参数，系统参数，不能修改。  
**用户配置`/etc/firewalld/`目录**下可以自定义添加端口等。

## 区域Zone

网络区域定义了网络连接的可信等级。对应的win系统就是防火墙的公用区域、专用区域等。

## 常用命令

### 1、`systemctl` [参数] `firewalld`

- `systemctl start firewalld`： 开启防火墙
- `systemctl stop firewalld`：关闭防火墙（开机重启会自动启动）
- `systemctl restart firewalld`：重启防火墙
- `systemctl enable firewalld`：启用防火墙
- `syatemctl disable firewalld`：禁用防火墙（开机不会自动启动）
- `systemctl status firewalld`：查看防火墙的状态

### 2、`firewall-cmd` [参数]

> `firewall-cmd` 是默认提供的一个操作防火墙的客户端

示例：`firewall-cmd --zone=public --add-port=3000/tcp --permanent`

- `--zone`：防火墙区域，
- `--add-port`：添加端口
- `--permanent`: 永久生效
- `--reload`：重载生效
- `--state`：查看防火墙状态，是否是running
- `--get-zones`：列出支持的zone
- `--get-services`：列出支持的服务，在列表中的服务是放行的
- `--query-service ftp`: 查看ftp服务是否支持，返回yes或者no
- `--add-service=ftp`: 临时开放ftp服务
- `--add-service=ftp --permanent`:永久开放ftp服务
- `--remove-service=ftp --permanent`：永久移除ftp服务
- `--list-ports`: 查看已经开放的端口

# 常用的Shell命令

```
rm -rf `ls ./*|egrep -v '(logs|xcarloan)'`      // 删除文件时排除文件 当前命令是删除当前目录下排除（logs、xcarloan）的其他所有文件
date +%Y%m%s    // 获取系统的当前的年月日
tar -zxvf xxx.tar.gz    // 解压
```

# 查看系统相关信息命令

```
cat /proc/cpuinfo	# 查看cpu信息
free -m 	# 查看内存和交换区使用量
env		# 查看环境变量
df -h 		# 查看各个分区的使用情况
grep MemTotal /proc/meminfo	 	# 查看内存总量
grep MemFree /proc/meminfo	 # 查看空闲内存余量
```

查看得到如下信息：

```
processor	: 0 	# 系统中逻辑处理核的编号
vendor_id	: GenuineIntel 	# CPU制造商
cpu family	: 6		# cpu产品系列代号
model		: 85	# CPU属于其系列的哪一代的代号
model name	: Intel(R) Xeon(R) Platinum 8163 CPU @ 2.50GHz
stepping	: 4		# CPU制作更新版本 
microcode	: 0x1	
cpu MHz		: 2499.996	# CPU实际使用主频
cache size	: 33792 KB	# CPU二级缓存大小
physical id	: 0		# 单个CPU的编号
siblings	: 2		# 单个CPU物理逻辑核数
core id		: 0		# 
cpu cores	: 1		# 该逻辑核所处CPU的屋里核数
apicid		: 0		#
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht sy2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch fsgsbase tsc_adjust bmisavec xgetbv1
bogomips	: 4999.99
clflush size	: 64
cache_alignment	: 64
address sizes	: 46 bits physical, 48 bits virtual

```

