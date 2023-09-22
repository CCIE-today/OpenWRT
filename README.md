# 硬件型号 - 极路由3

> system type		: `MediaTek MT7620A` ver:2 eco:6  
> machine		: HiWiFi `HC5861`  
> cpu model		: `MIPS 24KEc` V5.0

<img src="https://gitee.com/suzhen99/OpenWrt/raw/master/images/HiWIFI3-F.png" width="40%"><br>
<img src="https://gitee.com/suzhen99/OpenWrt/raw/master/images/HiWIFI3-B.png" width="40%">

<br>

# 软件版本 - OpenWrt 21.02
## 恢复出厂设置
### Router 网线
> + NIC <strong style='color: #A30000'>YELLOW</strong>
### STEP1、reset键
> 按住`reset`键，加电重启
### STEP2、PC IP
> - <strong>Laptop Wired</strong>  
> NIC `192.168.1.10/24`  
### STEP3、恢复出厂设置
>   - firefox `http://192.168.1.1`  
> <kbd>恢复出厂设置</kbd> / 固件类型 `OpenWrt` / <kbd>执行</kbd>  
> <kbd>重启</kbd> / <Kbd>重启</kbd>

<br>

## `firefox` http://192.168.1.1
### <kbd>Network</kbd> / <kbd>Interfaces</kbd>
>   - WAN <kbd>Delete</kbd>  
>   - WAN6 <kbd>Delete</kbd>  
>   - LAN <kbd>Edit</kbd>  
>       - <strong>常规设置</strong>  
> 设备 `eth0.2`  
> IPv4 地址 `192.168.3.2`  
> IPv4 网关 `192.168.3.1`  
>       - <strong>高级设置</strong>  
> 使用自定义的 DNS 服务器 `192.168.3.1` 
> - <kbd>Save & Apply</kbd>
<br>

### Router 网线
> - NIC `red`

<br>

## `firefox` http://192.168.3.2
### <kbd>System</kbd> / <kbd>Administration</kbd>
```bash
MIMA=P@33w0rd
(echo $MIMA; echo $MIMA) | passwd

```

<br>

### <kbd>System</kbd> / <kbd>Software</kbd> / <kbd>Configure opkg...</kbd>
```bash
sed -i.bk 's#https://downloads.openwrt.org#http://mirrors.tuna.tsinghua.edu.cn/openwrt#' /etc/opkg/distfeeds.conf
opkg update

```

<br>

### bash & vim & PS1
```bash
# bash - Configuring terminfo, libncurses6, libreadline8.
opkg install bash
# vim
opkg install vim-fuller
sed -i 's#/ash#/bash#' /etc/passwd

# /etc/profile
sed -i "/^export.*PS1/s#'.*'#'\\\e[36;1m\\\u\\\e[m@\\\e[35;1m\\\h\\\[\\\e[m:\\\W\\\\$ '#" /etc/profile
source /etc/profile

# coreutils - Configuring coreutils, libbz2-1.0, bzip2, xz-utils, xz.
opkg install coreutils-pwd
opkg install tar
opkg install unzip
opkg install luci-i18n-base-zh-cn

```

<br>

### <kbd>系统</kbd> / <kbd>启动项</kbd>
```bash
# firewall & dns & dhcp
/etc/init.d/firewall stop
/etc/init.d/firewall disable
/etc/init.d/dnsmasq stop
/etc/init.d/dnsmasq disable

```

<br>

### <kbd>System</kbd> / <kbd>Mount Points</kbd> - ext4
> Insert TF storage or USB storage

```bash
# install blockd - Configuring kmod-fs-autofs4.
opkg install block-mount blockd
# install ext4 - Configuring kmod-crypto-hash, librt, libuuid1, kmod-lib-crc16, kmod-crypto-crc32c, libblkid1, libcomerr0, libss2, libext2fs2.
opkg install kmod-fs-ext4 e2fsprogs
# install usb2 - Configuring kmod-scsi-core.
opkg install kmod-usb2 kmod-usb-storage

# install fdisk - Configuring libsmartcols1, libfdisk1.
opkg install fdisk

# mkfs.ext4 /dev/mmcblk0p1 - TF
until block info | grep -o /dev/mmcblk0p1; do
  echo Please wait a moment...
done
echo y | mkfs.ext4 /dev/mmcblk0p1
 
# cp /overlay /mnt/tf
mkdir /mnt/tf
mount /dev/mmcblk0p1 /mnt/tf
tar -C /overlay -cvf - . | tar -C /mnt/tf -xf -
umount /mnt/tf
rm -rf /mnt/tf

# /etc/config/fstab
block detect > /etc/config/fstab
sed -i "/enabled/s#'0'#'1'#" /etc/config/fstab
sed -i "/mmcblk0p1/s#mnt/mmcblk0p1#overlay#" /etc/config/fstab
cat /etc/config/fstab

# active
df -h /overlay
mount /dev/mmcblk0p1 /overlay
df -h /overlay

sync && reboot

```

<br>

### <kbd>System</kbd> / <kbd>Software</kbd> / <kbd>Free space</kbd>
```bash
df -h /overlay

```


<br>

### <kbd>System</kbd> / <kbd>Mount Points</kbd> - cifs
```bash
# install cifs
opkg update \
&& opkg install kmod-fs-cifs cifsmount kmod-nls-utf8 kmod-crypto-misc

# mkdir mount point
read -p "Please input your password: " SMBP
mkdir /mnt/wd
sed -i /exit/i"SMBP=$SMBP" /etc/rc.local
sed -i /exit/i'mount.cifs -o iocharset=utf8,vers=1.0,username=alex,password=$SMBP,file_mode=0644 //192.168.3.1/wd /mnt/wd' /etc/rc.local
mount.cifs -o iocharset=utf8,vers=1.0,username=alex,password=$SMBP,file_mode=0644 //192.168.3.1/wd /mnt/wd

```

<br>

### rsyncd
```bash
SPASS=P@33w0rd
# install
opkg install rsyncd

# /etc/rsyncd.conf
cat > /etc/rsyncd.conf <<-EOF 
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
use chroot = yes
uid = root
gid = root
read only = no
charset = utf-8
[WD]
path = /mnt/wd
list = yes
hosts allow = 192.168.3.0/255.255.255.0
auth users = alex
secrets file = /etc/rsyncd.secrets
EOF

# /etc/rsyncd.secrets
echo alex:$SPASS > /etc/rsyncd.secrets
chmod 600 /etc/rsyncd.secrets

# service
/etc/init.d/rsyncd restart

```

<br>

### `Services` / `aria2`
```bash
opkg install luci-app-aria2
opkg install webui-aria2
opkg install wget


# Update /etc/config/aria2
BTT=$(wget -qO- https://gitee.com/suzhen99/trackerslist/raw/master/trackers_all.txt | awk NF | sed ":a;N;s/\n/,/g;ta")
sed -i \
    -e /enabled/s#0#1# \
    -e /user/s#aria2#root# \
    -e "/ dir/s#'.*'#'/mnt/wd/Downloads'#" \
    -e "/bt_tracker ''/s#'.*'#'$BTT'#" /etc/config/aria2
cat >> /etc/config/aria2 <<EOF

        option enable_logging '0'
        option rpc_auth_method 'token'
        option rpc_secret 'qagnbo77ecggjadxxzsphme53xwwqglb'
        option rpc_secure 'false'
        option _rpc_url 'http://192.168.3.2:6800/jsonrpc'
        option enable_proxy '0'
        option check_certificate 'false'
        option enable_dht6 'false'
        option enable_peer_exchange 'true'
        option seed_time '1'
EOF
/etc/init.d/aria2 start

```

<br>

> - Installed web interface: [WebUI-Aria2](http://192.168.3.2/webui-aria2/)  
> `设置` / `连接设置` /
> 密码令牌（可选）: $(RPC token)

<br>

### upgradable
```bash
# upgrade
opkg update \
&& opkg list-upgradable \
   | cut -f 1 -d' '| xargs opkg upgrade

# reboot
sync && reboot
  
```

<br>

### Git 免密
```bash
# Configure - id_rsa openssh-client
opkg update \
&& opkg install openssh-keygen
[ -f /root/.ssh/id_rsa ] && rm /root/.ssh/id_rsa*
ssh-keygen -b 1024 -f /root/.ssh/id_rsa -N ''
cat /root/.ssh/id_rsa.pub

```
- [https://github.com/settings/keys](https://github.com/settings/keys)
- [https://gitee.com/profile/sshkeys](https://gitee.com/profile/sshkeys)
```bash
# /root/.gitconfig
opkg install git
git config --global user.name "Alex"
git config --global user.email "adder99@163.com"
opkg install openssh-client-utils
ssh -o StrictHostKeyChecking=no -T git@gitee.com
ssh -o StrictHostKeyChecking=no -T git@github.com

```

<br>

###  `系统` / `计划任务`
```bash
FDIR=/root/trackers
[ ! -d $FDIR ] && mkdir \$FDIR
cd $FDIR
git init
git remote add origin git@github.com:ngosang/trackerslist.git
git remote add gitee git@gitee.com:suzhen99/trackerslist.git

# Generate - /usr/lib/TLUpdate_crontab.sh
cat > /usr/lib/TLUpdate_crontab.sh <<EOF
#!/bin/bash
wget https://ngosang.github.io/trackerslist/trackers_all.txt -O trackers_all.txt
BTT=$(awk NF trackers_all.txt | sed ":a;N;s/\n/,/g;ta")
sed -i "/\tlist bt_tracker/s#'.*'#'\$BTT'#" /etc/config/aria2
/etc/init.d/aria2 restart
EOF

# Modify - /etc/crontabs/root
echo "0 4 */1 * * bash /usr/lib/TLUpdate_crontab.sh" > /etc/crontabs/root

# Service - /etc/init.d/cron
/etc/init.d/cron restart

```

<br>

### `服务` / [V2Ray](https://github.com/kuoruan)
```
# 1. Add new opkg key:
wget -O kuoruan-public.key http://openwrt.kuoruan.net/packages/public.key
opkg-key add kuoruan-public.key

# 2. Add opkg repository:
echo "src/gz kuoruan_packages http://openwrt.kuoruan.net/packages/releases/$(. /etc/openwrt_release ; echo $DISTRIB_ARCH)"   >> /etc/opkg/customfeeds.conf
echo "src/gz kuoruan_universal http://openwrt.kuoruan.net/packages/releases/all" >> /etc/opkg/customfeeds.conf

# 3. Install package:
opkg update
opkg install v2ray-core
#Configuring ca-certificates.
opkg install luci-app-v2ray
opkg install luci-i18n-v2ray-zh-cn
#Configuring libnfnetlink0.
#Configuring kmod-nf-conntrack-netlink.
#Configuring libnetfilter-conntrack3.
#Configuring libgmp10.
#Configuring libnettle7.


#4 . Upgrade package:
opkg update
opkg upgrade v2ray-core

```
```
opkg remove luci-i18n-v2ray-zh-cn
opkg remove luci-app-v2ray
opkg remove v2ray-core
echo > /etc/opkg/customfeeds.conf 
opkg-key remove kuoruan-public.key

```

<br>

### mariadb
```
opkg install mariadb-server
#Configuring liblzma.
#Configuring mariadb-common.
#Configuring libatomic1.
#Configuring libaio.
#Configuring mariadb-server-base
opkg install mariadb-client
#Configuring libedit.
#Configuring mariadb-client-base.
#Configuring mariadb-client.

mysql_install_db

uci set /etc/config/mysqld.general.enabled='1'

/etc/init.d/mysqld start

mysql -e "set password for root@localhost = password('P@33w0rd'); "

```

<br>

### php7-fpm
```
opkg install php7-fpm php7-cgi php7-fastcgi php7-cli
opkg install php7-mod-curl php7-mod-gd php7-mod-iconv php7-mod-json php7-mod-mbstring php7-mod-opcache php7-mod-intl php7-mod-session php7-mod-zip install php7-mod-pdo php7-mod-pdo-mysql php7-mod-ctype php7-mod-xml php7-mod-simplexml  php7-mod-hash php7-mod-dom
opkg install php7-mod-fileinfo php7-mod-exif

sed -i /open_basedir/s#=.*#= /mnt/sda1# /etc/php.ini
sed -i /doc_root/s#=.*#= "mnt/sda1/www"# /etc/php.ini
echo session.save_path = "/tmp" >> /etc/php.ini
echo mysqli.default_socket = "/var/run/mysqld/mysqld.sock" >> /etc/php.ini
echo pdo_mysql.default_socket = "/var/run/mysqld/mysqld.sock" >> /etc/php.ini

sed -i /listen =/s#=.*#= 127.0.0.1:9000# /etc/php7-fpm.d/www.conf
sed -i /listen.mode/s#^;## /etc/php7-fpm.d/www.conf
sed -i /listen.allowed_clients/s#^;## /etc/php7-fpm.d/www.conf

sed -i /^user/s_=.*_= php_fpm_ /etc/php7-fpm.d/www.conf
sed -i /^user/a'group = php_fpm' /etc/php7-fpm.d/www.conf

/etc/init.d/php7-fpm start

```

<br>

### nginx
```
opkg install luci-nginx
#Configuring uwsgi.
#Configuring uwsgi-syslog-plugin.
#Configuring uwsgi-cgi-plugin.
#Configuring uwsgi-luci-support.
#Configuring nginx-mod-luci.
#Configuring luci-nginx.

mkdir /mnt/sda1/www
chown php_fpm:php_fpm /mnt/sda1/www

sed -i s_80_8080_ /etc/nginx/nginx.conf
sed -i s_/www_/mnt/sda1/www_ /etc/nginx/nginx.conf
sed -i 42i'\        location ~ \.php$ {' /etc/nginx/nginx.conf
sed -i 43i'\            fastcgi_pass 127.0.0.1:9000;' /etc/nginx/nginx.conf
sed -i 44i'\            fastcgi_index index.php;' /etc/nginx/nginx.conf
sed -i 45i'\            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;' /etc/nginx/nginx.conf
sed -i 46i'\            fastcgi_param PATH_INFO $fastcgi_path_info;' /etc/nginx/nginx.conf
sed -i 47i'\            include fastcgi_params;' /etc/nginx/nginx.conf
sed -i 48i'\        }' /etc/nginx/nginx.conf

/etc/init.d/nginx start

```

<br>

### phpMyAdmin
```
wget https://files.phpmyadmin.net/phpMyAdmin/5.0.4/phpMyAdmin-5.0.4-all-languages.zip
unzip phpMyAdmin-5.0.4-all-languages.zip -d /mnt/sda1/www
mv phpMyAdmin-5.0.4-all-languages phpmyadmin

```
[http://192.168.3.2:88/phpmyadmin](http://192.168.3.2:88/phpmyadmin)
> 用户名：`root`  
> 密码：`P@33w0rd`


<br>

### Nextcloud
```
wget https://download.nextcloud.com/server/releases/latest-20.zip

unzip -o latest-20.zip -d /mnt/sda1/www/
chown -R php_fpm:php_fpm /mnt/sda1/www/

mysql -e "create database nextcloud charset utf8mb4 collate utf8mb4_bin;"
mysql -e "create user nextcloud@localhost identified by 'P@33w0rd';"
mysql -e "grant all privileges on nextcloud.* to nextcloud@localhost;"
mysql -e "flush privileges;"

```
> - 创建 **管理员帐号**
> 用户名：`admin`  
> 密码：`P@33w0rd`
> - 存储与数据库目录
> 配置数据库 / `MySQL/MariaDB`  
> 数据库用户 `root`  
> 数据库密码 `P@33w0rd`  
> 数据库名 `nextcloud`  
> `完成安装`

<br>

### owncloud
```
wget https://download.owncloud.org/community/owncloud-10.6.0.zip

unzip owncloud-10.6.0.zip -d /mnt/sda1/www/

```

<br>

### WordPress 
```
SPATH=/mnt/sda1/www
SDB=wordpress
SUSER=wordpress
SPASS=P@33w0rd
wget https://cn.wordpress.org/wordpress-5.6.1-zh_CN.zip
unzip wordpress* -d $SPATH
chown -R php_fpm:php_fpm /mnt/sda1/www/wordpress

mysql -pP@33w0rd -e "create database $SDB charset utf8mb4 collate utf8mb4_bin;"
mysql -pP@33w0rd -e "create user $SUSER@localhost identified by 'P@33w0rd';"
mysql -pP@33w0rd -e "grant all privileges on $SDB.* to $SUSER@localhost;"
mysql -pP@33w0rd -e "flush privileges;"

```
[http://192.168.3.2:88/wordpress](http://192.168.3.2:88/wordpress)
> - `现在就开始`  
> - 请在下方填写您的数据库连接信息
> 用户名：`wordpress`  
> 密码：`P@33w0rd`  
> `提交`  
> - `现在安装`
 

<br>

### h5ai 
```
wget https://release.larsjung.de/h5ai/h5ai-0.30.0.zip
unzip h5ai-0.30.0.zip -d /mnt/sda1/www
mv /mnt/sda1/www/h5ai-0.30.0 /mnt/sda1/www/h5ai

```

<br>

### Lychee
```
wget https://github.com/electerious/Lychee/archive/master.zip
unzip master.zip -d /mnt/sda1/www
mv /mnt/sda1/www/master /mnt/sda1/www/lychee

```

<br>

### kodexplorer
```
wget http://static.kodcloud.com/update/download/kodexplorer4.40.zip
unzip kodexplorer4.40.zip -d /mnt/sda1/www
mv /mnt/sda1/www/kodexplorer4.40 /mnt/sda1/www/kodexplorer

```

<br>

### Typecho 
```
wget http://typecho.org/downloads/1.1-17.10.30-release.tar.gz
unzip 1.1-17.10.30-release.tar.gz -d /mnt/sda1/www
mv /mnt/sda1/www/1.1-17.10.30-release /mnt/sda1/www/typecho

```

<br>

### Z-Blog 
```
wget https://update.zblogcn.com/zip/Z-Blog_2_3_Avengers_180518.zip
unzip Z-Blog_2_3_Avengers_180518.zip -d /mnt/sda1/www
mv /mnt/sda1/www/Z-Blog_2_3_Avengers_180518 /mnt/sda1/www/z-blog

```

<br>

### DzzOffice
```
git clone https://gitee.com/zyx0814/dzzoffice.git


```

<br>

### entware
```
mkdir /mnt/sda1/opt /opt
mount -o bind /mnt/sda1/opt /opt
wget -O - http://bin.entware.net/mipselsf-k3.4/installer/generic.sh | /bin/sh

sed -i /exit/i'mkdir -p /opt' /etc/rc.local
sed -i /exit/i'mount -o bind /mnt/sda1/opt /opt' /etc/rc.local
sed -i /exit/i'/opt/etc/init.d/rc.unslung start' /etc/rc.local

mv /opt/etc/profile-opkg /opt/etc/profile
. /opt/etc/profile

```

<br>

# APPEDICES

## [breed](https://breed.hackpascal.net)
```bash
RTIP=192.168.3.2
wget https://breed.hackpascal.net/breed-mt7620-hiwifi-hc5861.bin
scp breed-mt7620-hiwifi-hc5861.bin root@${RTIP}:/tmp
ssh root@${RTIP}
mtd -r write /tmp/breed-mt7620-hiwifi-hc5861.bin u-boot

```

## Chrome / SwitchyOmega
> - 情景模式 / proxy
> `SOCKS5` `127.0.0.1` `1080`  
> ACTIONS / `应用选项`
> - 情景模式 / auto switch
> remove*2
> `添加规则列表` /  
> 1. proxy  
> 2. 直接连接
> 3. AutoProxy
> 4. 规则列表地址 `https://gitee.com/gfwlist/gfwlist/raw/master/gfwlist.txt`
> 5. 立即更新情景模式


## `System` / `Mount Points` - swap
```bash
SFILE=/root/pagefile.sys
dd if=/dev/zero of=$SFILE bs=16777216 count=128
mkswap $SFILE
swapon $SFILE
free

cat >> /etc/config/fstab <<EOF
config swap
        option enabled '1'
        option device '$SFILE'
EOF

```

<br>

## `Services` / `qBittorrent` - right.com
```bash
IPATH=/mnt/wd/Downloads/mipsel_24kc-static
# install
opkg install ${IPATH}/qbittorrent_4.3.2-1_mipsel_24kc.ipk
opkg install ${IPATH}/luci-app-qbittorrent_unknown_all.ipk
# reboot
sync && reboot

```
> - <strong>Basic Settings</strong>  
> `Enabled`  
> Locale Language `Chinese (zh)`  
> - <strong>Log Settings</strong>  
> `Enable Log`
> - <strong>Downloads Settings</strong>  
> Save Path `/mnt/wd/Downloads`
> - `Save & Apply`

> - [http://192.168.3.2:8080](http://192.168.3.2:8080)  
> 用户名 `admin`  
> 密码 `adminadmin`  
> `登录`

```bash
# Uninstall
opkg remove luci-app-qbittorrent
opkg remove qbittorrent

```

<br>

## `Services` / `Wake on LAN`
```bash
# Install
opkg install luci-app-wol
# Configuring etherwake

```
```bash
# Usage
etherwake -D -i 'br-lan' "50:2B:73:D5:49:C0"
```
```
# Uninstall
opkg remove luci-app-wol
opkg remove etherwake
```

<br>

## `Services` / `Network Shares` - ok
```bash
# install
opkg install luci-app-samba

# useradd
opkg install shadow-useradd
chmod -R g+w /mnt/wd
useradd -G root alex
(echo alex99su; echo alex99su) | smbpasswd -as alex

# /etc/config/samba
sed -i '/home/s#1#0#' /etc/config/samba
cat >> /etc/config/samba <<EOF
config sambashare       
        option browseable 'yes'
        option name 'WD'       
        option path '/mnt/wd'
        option users 'alex'    
        option read_only 'no'  
        option guest_ok 'no' 
        option create_mask '644'
        option dir_mask '755'   
                                
config sambashare            
        option browseable 'yes'
        option name 'Cartoon'   
        option path '/mnt/wd/Cartoon'
        option read_only 'yes'        
        option create_mask '644'      
        option dir_mask '755'   
        option guest_ok 'yes'   
                             
config sambashare            
        option browseable 'yes'
        option name 'Film'
        option path '/mnt/wd/Film'
        option read_only 'yes'           
        option guest_ok 'yes'            

EOF

# /etc/samba/smb.conf.template
sed -i '/unix.*charset/s#=.*#= utf-8#' /etc/samba/smb.conf.template
sed -i '/interfaces =/s#=.*#= br-lan#' /etc/samba/smb.conf.template
sed -i '/invalid/a#       invalid users = alex' /etc/samba/smb.conf.template

# /etc/init.d/samba
/etc/init.d/samba restart

```

<br>

## nfsd - ok
```bash
# install
opkg install nfs-kernel-server

# /etc/exports
echo '/mnt/wd *(rw,sync,subtree_check,insecure,all_squash,anonuid=0,anongid=0)' > /etc/exports

# /etc/init.d/nfsd
/etc/init.d/nfsd restart

```

<br>

## python - err:memory
```bash
opkg install python python-pip
# Configuring libbz2-1.0, python-base, libffi, python-light, python-codecs, libxml2, libdb47, python-db, python-decimal, python-distutils, python-pydoc, python-ctypes, python-multiprocessing, libsqlite3-0, python-sqlite3, python-logging, libgdbm, python-gdbm, python-email, libexpat, python-xml, python-compiler, python-openssl, python-unittest, python-ncurses
# Configuring python-pkg-resources, python-setuptools, python-pip-conf

# 更改 pip 源
mkdir -p /root/.config/pip
cat > /root/.config/pip/pip.conf <<EOF
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF

# 安装 pip
opkg install wget
#Configuring libpcre
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py

# 安装 whl
pip install pyyaml==3.11
pip install requests==2.22
#installed certifi, chardet, urllib3, idna
pip install pyreadline==2.0

#pip --no-cache-dir install lxml
# 内存太小，swap 也不行
opkg install python-lxml
#Configuring libxslt, libexslt

BREAK
# cannot import name winconstants
opkg install gcc
#Configuring libbfd, libopcodes, objdump, ar, binutils
#opkg install pkg-config
pip install pyzmq

# icrm
cp -r /mnt/wd/OneDrive/icrm /usr/lib/python2.7/site-packages
cp /usr/lib/python2.7/site-packages/icrm/bin/icrm /usr/bin

icrm help
sed -i.bk '/repository/s|:.*|: /Volumes/DATA/INSTRUCTOR/repository|' .icrm/config.yml
cat .icrm/config.yml
```
```
chmod +x /usr/bin/icrm
```

<br>

## 无线中继
> `Netowrk` / `Wireless` /<br>
> MediaTek MT76x2E 802.11nac `[Scan]` /<br>
> `[Join Network]` /<br>
> &nbsp;&nbsp;&nbsp;&nbsp;WPA passphrase `P@33w0rd`<br>
> &nbsp;&nbsp;&nbsp;&nbsp;Create / Assign firewall-zone `unspecified`
> &nbsp;&nbsp;`[Submit]` / `[Save]` / `[Save & Apply]`

<br>

## 同网段无线中继
```bash
sed -i 's#http://downloads.openwrt.org#https://mirrors.tuna.tsinghua.edu.cn/openwrt#' /etc/opkg/distfeeds.conf
opkg update && \
opkg install luci-proto-relay && \
# firewall
/etc/init.d/firewall stop && \
/etc/init.d/firewall disable
```

### 1. Relay bridge
> `Network` / `Interface` / `Add new interface...` /<br>
> &nbsp;&nbsp;&nbsp;&nbsp;Name：`stabridege`<br>
> &nbsp;&nbsp;&nbsp;&nbsp;Protocol：`Relay bridge`<br>
> &nbsp;&nbsp;&nbsp;&nbsp;`[Create interface]` /<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Local IPv4 address `192.168.3.2`<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Relay between networks `lan` `wwan`<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`[Save]`<br>
> &nbsp;&nbsp;&nbsp;&nbsp;`[Save & Apply]`
### 2. Disable DHCP for LAN
> `Network` / `Interfaces` / `LAN` / `[Edit]` /<br>
> - `DHCP Server` / General Setup / `Ignore interface`
> - `Physical Settings` / Interface `eth0.1` `eth0.2` <br>
> `[Save]` / `[Save & Apply]`
### 3. Reboot
> `System` / `Reboot` / `[Perform reboot]`