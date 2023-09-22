编译 OpenWrt 固件
===


## 1. Development

> 安装 `Ubuntu 18 LTS x64`

### 1.1 /etc/apt/sources.list

```bash
$ sudo sed -i '/^deb/s#http.*bionic#http://mirrors.aliyun.com/ubuntu/ bionic#' /etc/apt/sources.list

$ grep ^deb /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ bionic universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates universe
deb http://mirrors.aliyun.com/ubuntu/ bionic multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ bionic-security universe
deb http://mirrors.aliyun.com/ubuntu/ bionic-security multiverse

$ sudo apt-cdrom add
$ sudo apt install open-vm-tools-desktop
$ reboot

```

<br>

##  2. Requirements

```bash
$ sudo apt-get update

$ sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync

```

<br>

##  3. Quickstart

### 3.a 第一次编译

```bash
#git clone https://github.com/coolsnowwolf/lede
$ git clone https://gitee.com/jjm2473/lede.git

$ cd lede

$ ./scripts/feeds update -a && ./scripts/feeds install -a
$ make menuconfig

```
> <strong style='color: #3F659F'>T</strong>arget System (<strong>MediaTec Ralink MIPS</strong>)   --->
>
> <strong style='color: #3F659F'>S</strong>ubtarget (<strong>MT7620 based boards</strong>)   --->
>
> <strong style='color: #3F659F'>T</strong>arget Profile (<strong>HiWiFi HC5861</strong>)   --->
>
> <strong style='color: #3F659F'>L</strong>uCI   --->
>
> 1. <strong style='color: #3F659F'>C</strong>ollections   --->
>           -*****- <strong style='color: #3F659F'>l</strong>uci
>
> <<strong style='color: #3F659F'>E</strong>xit>

   ```bash
$ make -j8 download V=s

   ```
   ```bash
# -j1 jobs 后面是线程数。第一次编译推荐用单线程
$ make -j1 V=s

$ ls lede/bin/targets

   ```

### 3.a 第二次编译
```bash
$ cd lede
$ git pull
$ ./scripts/feeds update -a && ./scripts/feeds install -a
$ make defconfig
$ make -j8 download
$ make -j$(($(nproc) + 1)) V=s

```

### 3.b 重新配置
```bash
$ rm -rf ./tmp && rm -rf .config
$ make menuconfig
```
><strong style='color: #3F659F'>T</strong>arget System (<strong>MediaTec Ralink MIPS</strong>)   --->
>
><strong style='color: #3F659F'>S</strong>ubtarget (<strong>MT7620 based boards</strong>)   --->
>
><strong style='color: #3F659F'>T</strong>arget Profile (<strong>HiWiFi HC5861</strong>)   --->
>
><strong style='color: #3F659F'>L</strong>uCI   --->
>
>       3. <strong style='color: #3F659F'>A</strong>pplications   --->
>          <<strong>*</strong>> <strong style='color: #3F659F'>l</strong>uci-app-baidupcs-web
>  		 <  > <strong style='color: #3F659F'>l</strong>uci-app-vlmcsd
>
><<strong style='color: #3F659F'>E</strong>xit>
```bash
$ make -j8 download V=s
$ make -j1 V=s

$ cp bin/targets/ramips/mt7620/openwrt-ramips-mt7620-hiwifi_hc5861-squashfs-sysupgrade.bin /mnt/wd

```



---

# APPENDENCES

## 注意权限
> - **不**要用 `root` 用户 git 和编译

## 登陆信息

> - 登录 IP `192.168.1.1`
> - 密码 `password`

## 软路由

> x86/x64 路由器，[小马v1 的铝合金版本 (N3710 4千兆)](https://item.taobao.com/item.htm?spm=a230r.1.14.20.144c763fRkK0VZ&id=561126544764 " 小马v1 的铝合金版本")