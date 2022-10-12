![OpenWrt logo](include/logo.png)

OpenWrt V21.02.3官方稳定版源码  根据个人需要修改了自定义配置，添加了常用插件  
官方仓库地址 https://github.com/openwrt/openwrt.git  
version `r16554-1d4dea6d4f`

## Ubuntu22.04编译工具
官方建议的工具
https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem
```
sudo apt update
sudo apt install build-essential gawk gcc-multilib flex git gettext libncurses5-dev libssl-dev python3-distutils rsync unzip zlib1g-dev qemu-utils -y
```
ImmortalWrt建议的工具
```
sudo apt update -y
sudo apt full-upgrade -y
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
  bzip2 ccache cmake cpio curl device-tree-compiler ecj fastjar flex gawk gettext gcc-multilib g++-multilib \
  git gperf haveged help2man intltool lib32gcc1 libc6-dev-i386 libelf-dev libglib2.0-dev libgmp3-dev libltdl-dev \
  libmpc-dev libmpfr-dev libncurses5-dev libncursesw5 libncursesw5-dev libreadline-dev libssl-dev libtool lrzsz \
  mkisofs msmtp nano ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pip python3-ply \
  python-docutils qemu-utils re2c rsync scons squashfs-tools subversion swig texinfo uglifyjs upx-ucl unzip \
  vim wget xmlto xxd zlib1g-dev
```
```
# 基本编译命令
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
make -j8 download V=s 下载dl库（国内请尽量全局科学上网）
make -j4 V=s （-j1 后面是线程数。第一次编译推荐用单线程）（虚拟机最好指定线程数）

# 用管道将编译日志传送给 tee 命令（需要安装），最终写入到 compile.log 文件里
make -jN V=s | tee ../compile.log

cat ../compile.log | grep ERROR:
cat ../compile.log | grep failed
```


## 自定义配置
LAN ip `192.168.123.1`  
默认时区 `Asia/Shanghai`  
修正连接数为`165535`  
锁定默认主题为`bootstrap`  

## 常用插件
```
# PassWall  
git clone -b packages https://github.com/xiaorouji/openwrt-passwall.git openwrt-passwall
git clone -b luci https://github.com/xiaorouji/openwrt-passwall.git luci-passwall
```
```
# PassWall2
git clone https://github.com/xiaorouji/openwrt-passwall2.git luci-passwall2
```
```
# SSR Plus
git clone --depth=1 https://github.com/fw876/helloworld.git ssr-plus

# 对于 OpenWrt 21.02 或更低版本,必须手动将 Golang 工具链升级到1.18或更高版本才能编译 Xray-core。
./scripts/feeds update packages
rm -rf feeds/packages/lang/golang
svn co https://github.com/openwrt/packages/branches/openwrt-22.03/lang/golang feeds/packages/lang/golang
```
```
# OpenClash
mkdir package/luci-app-openclash
cd package/luci-app-openclash
git init
git remote add -f origin https://github.com/vernesong/OpenClash.git
git config core.sparsecheckout true
echo "luci-app-openclash" >> .git/info/sparse-checkout
git pull --depth 1 origin master
git branch --set-upstream-to=origin/master master
```
```
# AdGuardHome
git clone https://github.com/rufengsuixing/luci-app-adguardhome.git
```
```
# luci-app-xray
git clone https://github.com/yichya/luci-app-xray.git
git clone https://github.com/Mitsuhaxy/luci-i18n-xray-zh-cn.git
```
```
# luci-app-aliddns
git clone https://github.com/pymumu/smartdns.git
```
```
# luci-app-dockerman
git clone https://github.com/lisaac/luci-app-dockerman.git
```
```
# luci-theme-argon
git clone https://github.com/jerrykuku/luci-theme-argon.git
```
```
# luci-theme-atmaterial
git clone https://github.com/esirplayground/luci-theme-atmaterial-ColorIcon.git
```
```
# luci-theme-neobird
git clone https://github.com/thinktip/luci-theme-neobird.git
```

## 编译常见问题
开启ipv6支持
```
make menuconfig——extra packages——ipv6helper
make menuconfig——NetWork——Firewall——ip6tables——ip6tables-extra & ip6tables-mod-nat
```
编译passwall 出现 v2ray-plugin 报错  
```
下载upx
wget https://github.com/upx/upx/releases/download/v3.96/upx-3.96-amd64_linux.tar.xz
解压
tar -Jxf upx-3.96-amd64_linux.tar.xz
进入upx文件夹，赋予执行权限
chmod +x upx
将upx相关文件复制到相关文件夹
cp ~/xxx/upx-3.96-amd64_linux/upx staging_dir/host/bin
```
更新默认的smartdns版本
```
# HASH值在项目查看 https://github.com/pymumu/smartdns.git
sed -i 's/1.2022.37/1.2022.37.2/g' feeds/packages/net/smartdns/Makefile
sed -i 's/5a2559f0648198c290bb8839b9f6a0adab8ebcdc/64e5b326cc53df1fec680cfa28ceec5d8a36fcbc/g' feeds/packages/net/smartdns/Makefile
sed -i 's/^PKG_MIRROR_HASH/#&/' feeds/packages/net/smartdns/Makefile

安装smartdns（可选）
git clone -b lede https://github.com/pymumu/luci-app-smartdns.git package/luci-app-smartdns
# git clone https://github.com/pymumu/smartdns.git package/smartdns
```
---
官方懒人脚本  
可执行如下命令，一次性下载smartdns以及luci-app-smartdns。
下列命令可采用复制粘贴的方式执行。

注意事项：

执行下列命令时，需要确保当前路径为openwrt代码路径。
确保执行过./scripts/feeds进行更新。

```
WORKINGDIR="`pwd`/feeds/packages/net/smartdns"
mkdir $WORKINGDIR -p
rm $WORKINGDIR/* -fr
wget https://github.com/pymumu/openwrt-smartdns/archive/master.zip -O $WORKINGDIR/master.zip
unzip $WORKINGDIR/master.zip -d $WORKINGDIR
mv $WORKINGDIR/openwrt-smartdns-master/* $WORKINGDIR/
rmdir $WORKINGDIR/openwrt-smartdns-master
rm $WORKINGDIR/master.zip

LUCIBRANCH="master" #更换此变量
WORKINGDIR="`pwd`/feeds/luci/applications/luci-app-smartdns"
mkdir $WORKINGDIR -p
rm $WORKINGDIR/* -fr
wget https://github.com/pymumu/luci-app-smartdns/archive/${LUCIBRANCH}.zip -O $WORKINGDIR/${LUCIBRANCH}.zip
unzip $WORKINGDIR/${LUCIBRANCH}.zip -d $WORKINGDIR
mv $WORKINGDIR/luci-app-smartdns-${LUCIBRANCH}/* $WORKINGDIR/
rmdir $WORKINGDIR/luci-app-smartdns-${LUCIBRANCH}
rm $WORKINGDIR/${LUCIBRANCH}.zip

./scripts/feeds install -a
make menuconfig
```
---
修改默认 IP（可选）
```
sed -i 's/192.168.1.1/192.168.123.1/g' package/base-files/files/bin/config_generate
```
设置密码为空（可选）
```
sed -i 's@.*CYXluq4wUazHjmCDBCqXF*@#&@g' package/lean/default-settings/files/zzz-default-settings
```
更改Linux内核（可选）
```
sed -i 's/KERNEL_PATCHVER:=5.15/KERNEL_PATCHVER:=5.19/g' target/linux/x86/Makefile
```
旁路由模式，如使用`xtls-rprx-splice`流控模式，建议关闭系统的`ip转发`   
参见https://github.com/XTLS/Xray-core/discussions/59
```
vi /etc/sysctl.d/10-default.conf
# net.ipv4.ip_forward=1  //注释掉
# net.ipv6.conf.all.forwarding=1  //注释掉
:wq
sysctl -p /etc/sysctl.d/10-default.conf 或者重启
```
