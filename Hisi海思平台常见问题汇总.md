[TOC]
# Hisi海思平台常见问题汇总
## 1.HISI资源
### 1.1 文档
海思的文档非常详细，请仔细阅读，不要重复询问已知的问题（public项目不需要权限）  
`http://pms.inspur.com/scholar/hisilicon-docs`  
hisi文档非常的多，应该阅读哪一个？  
看对应代码git log中最新的hisi补丁打的是哪个，就阅读哪个，例如下述的补丁包。
HiSTBAndroidV600R003C01SPC011 就阅读这个文件夹下的。
### 1.2 软件
```
http://pms.inspur.com/scholar/HiSTBAndroid4.4
http://pms.inspur.com/scholar/HiSTBAndroid5.1.1
http://pms.inspur.com/scholar/HiSTBAndroid7.0
```
Jenkins 管理地址.（非AFW组使用ott/ott 登陆）  
`http://10.180.88.88:8080/view/broadcom-hisi/job/hisi4.4/`
## 2. hisi原厂支持
### 2.1 咨询性的问题直接电话
田明 ming.tian@hisilicon.com  135-8181-2422  
魏明刚 minggang.wei@hisilicon.com  180-3807-8052  
### 2.2 技术细节问题
请去hisupport系统提单  
```
https://hisupport.hisilicon.com/hisupport
User ID: inspur_004
#Password: ugY;`#6b
Password: 123456aA!
```
提单的时候可以修改联系人和联系方式:
![question](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171221155910895.png)

## 3. 开发环境配置
### 3.1 更新apt
`apt-get update`
###  3.2  安装工具包
```
apt-get install git make zlib1g-dev:i386 cpp gcc-multilib \
g++ g++-4.8-multilib cpp-4.8 g++-4.8 gcc-4.8 binutils gnupg flex \
lib32ncurses5-dev bison gperf build-essential zip curl libc6-dev \
x11proto-core-dev libx11-dev lib32readline6-dev zlib1g-dev \
lib32z-dev libgl1-mesa-dev g++-multilib mingw32 tofrodos gettext \
libxml2-utils xsltproc u-boot-tools
```
### 3.3 gcc降级
```
apt-get install gcc-4.4 g++-4.4 g++-4.4-multilib gcc-4.4-multilib
rm /usr/bin/gcc
ln -s /usr/bin/gcc-4.4 /usr/bin/gcc
rm /usr/bin/g++
ln -s /usr/bin/g++-4.4 /usr/bin/g++
```
### 3.4 安装jdk
如果有好几个jdk版本，则需要切换到指定版本，方法如下：  
`update-alternatives --config java`  
不想切换的话就装一个，如果已经装了高版本又不想切换，删了重新装。
### 3.5 删除java
移除所有 Java相关包 (Sun, Oracle, OpenJDK, IcedTea plugins, GIJ):
```
apt-get update
apt-cache search java | awk '{print($1)}' | grep -E -e '^(ia32-)?(sun|oracle)-java' -e '^openjdk-' -e '^icedtea' -e '^(default|gcj)-j(re|dk)' -e '^gcj-(.*)-j(re|dk)' -e 'java-common' | xargs sudo apt-get -y remove
apt-get -y autoremove
```
清除配置信息:
`dpkg -l | grep ^rc | awk '{print($2)}' | xargs sudo apt-get -y purge`
清除java配置及缓存:
`bash -c 'ls -d /home/*/.java' | xargs sudo rm -rf`
手动清除JVMs:
`rm -rf /usr/lib/jvm/*`
### 3.6 重新安装
```
apt-cache search jdk
apt-get install openjdk-6-jdk
```
安装之后查看版本：
`java --version`
### 3.7 更换shell为bash
`rm /bin/sh;ln -s /bin/bash /bin/sh`
### 3.8 安装交叉编译链()
经验证此步可以跳过,最终使用的工具链并不是这个.  
交叉编译工具链路径：  
`Software/ServerInstall/arm-hisiv200-linux`  
切换至root用户后，在交叉工具链安装包目录下执行命令：  
`./cross.install`  
退出服务器登录，重新登录服务器，如果可以自动补全就成功了。  
这里虽然配置了编译链，但是在sdk中也会进行配置，而且使用的编译链是不同的，具体在mak文件中。  
### 3.9 设置最大打开文件数
在etc/security/limits.conf中添加下面的内容修改，文档中的修改方式无效。
```
*       soft    nofile  8192
*       hard    nofile  8192
root    soft    nofile  8192
root    hard    nofile  8192
```
### 3.10 确认服务器umask的值
umask决定了新建目录和文件时的初始权限，当`umask = 022`时，新建的目录权限是755，文件权限是644，修改方法如下：  
在编译服务器shell中输入umask命令，查看返回值是否为0022  
如果不是，则需要编辑/etc/profile文件，添加：  
`umask 022`  
重新登录后，确认umask值是否正确即可。  
### 3.11 给mknod chmod chown增加s权限
```
sudo chmod a+s /bin/mknod
sudo chmod a+s /bin/chmod
sudo chmod a+s /bin/chown
```
## 4.hisi3798mv200编译相关
### 4.1 源码
git仓库地址：  
`git@pms.inspur.com:scholar/HiSTBAndroid4.4.git`  
文档地址：  
`git@pms.inspur.com:scholar/hisilicon-docs.git`
### 4.2 jenkins上的编译脚本：
```
#!/bin/bash
git pull -f origin master:master
git log -3
#rm -rf out
#git clean -df

#cd vendor/inspur/carp_build ;./7251s_android_xmlparse.py ccbn.xml;cd -

#export INSPUR_COMPILE_ONPMS=TRUE

source build/envsetup.sh
lunch Hi3798MV200-userdebug
make bigfish -j4
if [ $? -ne 0 ];then
	echo "make error"
    exit 1
fi
make updatezip -j4
if [ $? -ne 0 ];then
	echo "make ota error"
    exit 2
fi


vendor/inspur/getallbin.sh
```
### 4.3 编译
在代码的根目录执行：
```
source build/envsetup.sh
lunch Hi3798MV200-userdebug
```
#### 4.3.1 完整编译
`make bigfish -j8 2>&1 | tee bigfish.log`
#### 4.3.2 编译Android内核分区镜像
`make kernel –j8 2>&1 | tee kernel.log`
会编译出kernel.img镜像。
#### 4.3.3 编译 Emmc 器件上的 Android 系统分区镜像
`make ext4fs -j8 2>&1 | tee ext4fs.log`  
会编译出  
`out/target/product/Hi3798MV200/EMMC`目录下：  
`system.ext4 userdata.ext4 cache.ext4 kernel.img`
#### 4.3.4 编译 Android recovery 小系统内核分区镜像
`make recoveryimg -j8 2>&1 | tee recovery.log`  
会编译出recovery.img镜像，编译过程文件在  
`out/target/product/Hi3708MV200/obj/RECOVERT_OBJ/`
#### 4.3.5 编译 Android recovery 升级包 update.zip
`make updatezip -j8 2>&1 | tee updatezip.log`  
输出在  
`out\target\product\Hi379XXVXXX\Emmc`目录下： update.zip
#### 4.3.6 编译 fastboot 分区镜像
`make hiboot -j8 2>&1 | tee hiboot.log`  
输出在  
`out\target\product\Hi3798MV200\Emmc`目录下： fastboot.bin
#### 4.3.7 清除编译结果
`make clean`
## 5.hisi3798mv200使用Hitool烧录
不具备USB升级功能的旧版本软件,或者各种原因导致的盒子软件异常,只要串口可用,可以尝试本方法使用Hitool工具进行烧录,工具可以选择只烧录部分分区,局部更改验证的时候可以使用此方法.
### 5.1 烧录准备
* 确保电脑及开发板在同一网络环境
* 确保串口连接,请关闭其他串口工具,本地PC配置请正确配置串口及PC的IP
* 选择传输方式为网口,板端配置如果开发板能正常获取IP请正确配置,如果拿不到IP不配置也可以,但是要确保开发板能够找到PC,PC是作为服务器的
![hitool](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171221162414292.png)
### 5.2 烧录
选择烧写EMMC项,浏览选择分区表xml文件,最后选择需要烧录的分区,勾选后点击烧写即可,这部分海思的文档也写的比较详细,可以再仔细看下,对应的操作如下图:
![hitool](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171221163203159.png)

## 6.hisi3798mv200遥控器进入Recovery及USB升级
在开机启动后**点按**，**注意不是按住** 遥控器的power键，进入recovery,进入后的界面：
![revocery](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-2017121810585550.png)  
选择对应的菜单项进入对应的功能，USB升级需要将编译的update.zip文件(可以将服务器的update-qin.zip重命名即可)存放在U盘根目录，即可使用USB升级升级。
## 7.裸片USB升级方法
如果遇到烧录失败的情况,需要擦除后使用裸片升级的方法.此时可使用Hitool的全器件擦除,然后进行下面的裸片升级过程.如果还能进入recovery,也可以用命令去擦除.
### 7.1需要准备的设备及文件
* hisi3798mv200芯片对应机顶盒一台,电源配件
* usb转串口线,串口小板
* 8G或8G以上存储U盘一个（FAT32格式,如果是4G的flash,可以使用4G的U盘）
* PMS系统的以下文件
  * ![recovery.img](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103445817.png)
  * ![fastboot.bin](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103635468.png)
  * ![bootargs.bin](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103750502.png)
  * ![update-qin.zip](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103903501.png)

将update-qin.zip重命名为update.zip
### 7.2清空flash(如果是裸片可以跳过这一步)
进入recovery后,在串口中进行下面的操作.  
**步骤1 清空emmc boot0**  
`busybox dd if=/dev/zero of=/dev/block/mmcblk0boot0`  
串口输出类似:
```
dd: writing '/dev/block/mmcblk0boot0': No space left on device
8193+0 records in
8192+0 records out
4194304 bytes (4.0MB) copied, 0.126209 seconds, 31.7MB/s
```
**步骤2 清空emmc boot1**  
`busybox dd if=/dev/zero of=/dev/block/mmcblk0boot1`  
串口输出类似:
```
dd: writing '/dev/block/mmcblk0boot1': No space left on device
8193+0 records in
8192+0 records out
4194304 bytes (4.0MB) copied, 0.126298 seconds, 31.7MB/s
```
**步骤3 清空emmc**  
`busybox dd if=/dev/zero of=/dev/block/mmcblk0`  
串口输出类似:
```
dd: writing '/dev/block/mmcblk0': No space left on device
15269889+0 records in
15269888+0 records out
7818182656 bytes (7.3GB) copied, 567.546554 seconds, 13.1MB/s
```
**注意:这一过程大概需要15分钟左右,在此期间不要断电.**

### 7.3 USB烧写fastboot,bootargs,recovery,update.zip
**步骤1** 将之前准备工作中需要的4个文件放至U盘根目录;  
**步骤2** 连接U盘至板子上的USB接口;  
**步骤3** 开机;  
**步骤4** 进入USB升级流程,完成USB升级后会重新启动进入系统  
**注意此方法只适用于裸片升级.**

## 8.内存限制方法
在新疆项目上使用1+4的配置,但是目前的开发板都是2+4或者2+8的,因此需要对内存进行限制,具体方法如下:  
**步骤1** 开机按ctrl+c进入fastboot  
**步骤2** `printenv`查看bootargs如下:  
```
bootargs_2G=mem=2G mmz=ddr,0,0,48M
```
**步骤3** 修改bootargs  
```
setenv bootargs_2G 'mem=1G mmz=ddr,0,0,48M'
saveenv
```
断电重启  
**步骤4** 启动后查看内存情况  
```
cat /proc/cmdline
cat /proc/meminfo
```
分别查看修改后的bootargs和meminfo中的MemTotal值确认是否修改成功即可.
## 9.产测软件相关
在fastboot中检测MAC,当没有MAC或者特定的MAC时会进入产测软件,对于工厂生产flash都是空的,没有MAC,此时会触发进入产测软件,对于开发和测试中使用的盒子,需要清除MAC后进入.
### 9.1 正常进入产测
如果想要模拟工厂生产的情形,可以参照第7章的内容清空flash,然后裸片升级后,进入产测软件.
如果仅需要清空MAC,可采用如下方法:  
**步骤1** 开机连续点按遥控器的POWER键进入recovery.  
**步骤2** 在recovery中输入如下命令:  
```
busybox dd if=/dev/zero of=/dev/block/platform/soc/by-name/deviceinfo
```
**步骤3** 重启进入产测软件.  
**步骤4** 完成产测.  
**步骤5** 写号工具写入序列号.  
**步骤6** 重启进入android系统.  
### 9.2 跳过产测进入系统
电脑连接机顶盒串口,机顶盒开机后使用ctrl+c进入fastboot命令行,输入命令:  
`mmc read 0 0x1FFBFC0 0x50000 0x5000; bootm 0x1FFBFC0`  
即可跳过产测进入系统.
## 10.开启串口输入及adb
默认关闭了串口输入及adb,需要使用遥控器开启,可以顺序按下`*#06#`或者`上上下下左左右右`来开启串口输入及adb,adb连接的端口为65432
## 11.标安生产镜像制作
### 11.1 需要准备的设备及文件
* hisi3798mv200芯片对应机顶盒一台,电源配件
* usb转串口线,串口小板
* 8G或8G以上存储U盘一个（FAT32格式）
* PMS编译结果中的以下几个文件(放入U盘根目录)  
![recovery.img](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103445817.png)  
![fastboot.bin](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103635468.png)  
![bootargs.bin](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103750502.png)  
![update-qin.zip](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20171218103903501.png)  
将update-qin.zip文件重命名为update.zip  
**注意:这些文件必须是同一次编译出的结果**  
### 11.2 注意事项
1. 制作镜像前机顶盒不要插入CA或TF卡，以避免进入产测时个别测试项被pass，导致工厂误判（U盘、HDMI线或串口小板不会影响）.  
2. 不要接cable线，避免软件开机时接收大网描述符，导致制作的镜像文件脱离工厂要求.  
### 11.3 镜像文件制作
#### 11.3.1 进入recovery
在开机时连续点按遥控器power键进入revocery界面:  
![revocery](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-2017121810585550.png)  
#### 11.3.2 清空flash
进入recovery后,在串口中进行下面的操作.  
**步骤1 清空emmc boot0**  
`busybox dd if=/dev/zero of=/dev/block/mmcblk0boot0`  
串口输出类似:
```
dd: writing '/dev/block/mmcblk0boot0': No space left on device
4097+0 records in
4096+0 records out
2097152 bytes (2.0MB) copied, 0.050399 seconds, 39.7MB/s
```
**步骤2 清空emmc boot1**  
`busybox dd if=/dev/zero of=/dev/block/mmcblk0boot1`  
串口输出类似:
```
dd: writing '/dev/block/mmcblk0boot1': No space left on device
4097+0 records in
4096+0 records out
2097152 bytes (2.0MB) copied, 0.050581 seconds, 39.5MB/s
```
**步骤3 清空emmc**  
`busybox dd if=/dev/zero of=/dev/block/mmcblk0`  
串口输出类似:
```
dd: writing '/dev/block/mmcblk0': No space left on device
7733249+0 records in
7733248+0 records out
3959422976 bytes (3.7GB) copied, 571.630807 seconds, 6.6MB/s
```
**注意:这一过程需要较长时间,在此期间不要断电.**  
#### 11.3.3 软件升级
完成清空flash之后，接入U盘之后断电重启.此时会自动使用U盘中的update.zip包对盒子进行升级,如果没有进入升级,串口打印内容会停在:  
```
Bootrom start
Boot Media: eMMC

Enter usb bootstrap by flash
USB(1): no found "fastboot.bin".
Boot head format invalid.
```
出现此种情况表示没有识别到U盘中的相关文件,请更换U盘重启再次尝试.  
进入升级程序的打印内容如下:
```
Bootrom start
Boot Media: eMMC

Enter usb bootstrap by flash
```
**注意:升级完成后由于清空了flash,重启会进入产测程序,此时需要使用`ctrl+c`中断启动进入fastboot命令行,看到串口如下打印后表示升级即将完成,此时就可以开始使用ctrl+c了,如果太晚了会进入产测.整个镜像制作过程不允许进入产测,否则一些自动测试项的测试结果会被包含在镜像中,影响工厂产测结果的准确性.**  
当在串口中看到如下内容后使用ctrl+c中断启动:  
```
set system symlink ok.....
# IsUpdatePackage success=0,path=/cache : /sdcard
format cache......
unmount of /cache failed; no such volume
Creating filesystem with parameters:
    Size: 838860800
    Block size: 4096
    Blocks per group: 32768
    Inodes per group: 7328
    Inode size: 256
    Journal blocks: 3200
    Label: 
    Blocks: 204800
    Block groups: 7
    Reserved block group size: 55
Created filesystem with 11/51296 inodes and 6651/204800 blocks
warning: wipe_block_device: Discard failed
# update ok.....
```
使用`ctrl+c`中断后进入fastboot命令行,串口输出类似:
```
Press Ctrl+C to stop autoboot
fastboot# <INTERRUPT>
fastboot# 
```
此时重启使用遥控器进入recovery.  
**这里需要恢复出厂设置一次,因为升级包未包含data分区,需要初始化一下data分区,否则启动会有校验错误不能跳过产测**  
恢复出厂设置后会自动重启,这个时候仍需要使用`ctrl+c`终端进入fastboot命令行,输入命令  
`mmc read 0 0x1FFBFC0 0x50000 0x5000; bootm 0x1FFBFC0`  
即可跳过产测进入Android系统.  
#### 11.3.4 导出镜像
完成Android第一次启动之后,重启使用遥控器按键进入recovery,方法和之前相同.进入recovery之后开始导出镜像.  
**步骤1 连接U盘,在串口中输入如下命令:(U盘情景较多,具体要根据实际情况应对)**  
```
busybox mkdir -p /mnt/sdcard
busybox mount -t vfat /dev/block/sda1 /mnt/sdcard
cd /mnt/sdcard/
```
**步骤2 dump boot0**  
`busybox dd if=/dev/block/mmcblk0boot0  of=/mnt/sdcard/boot0.bin`  
串口输出类似:  
```
8192+0 records in
8192+0 records out
4194304 bytes (4.0MB) copied, 0.061354 seconds, 65.2MB/s
```
**步骤3 dump boot1**  
`busybox dd if=/dev/block/mmcblk0boot1  of=/mnt/sdcard/boot1.bin`  
串口输出类似:  
```
8192+0 records in
8192+0 records out
4194304 bytes (4.0MB) copied, 0.063465 seconds, 63.0MB/s
```
**步骤4 dump镜像**  
这个分区很大,需要根据flash大小分别操作.  
**4G flash方法:**  
串口输入命令：  
`busybox dd if=/dev/block/mmcblk0  of=/mnt/sdcard/user.bin bs=4096 count=952320`  
(反向测试:使用命令:`busybox dd if=/mnt/sdcard/user.bin  of=/dev/block/mmcblk0 bs=4096 count=952320`)  
为了应对不同厂家flash的实际空间不同的情况,对于4G的flash,我们所有分区使用的大小为3700M,这里dump3720M(4096*952320),实际flash的大小要求要大于3720M.  
**8G或者更高的flash方法：**  
这里推荐直接压缩，然后再写入的方案，fat32格式U盘即可.  
`busybox cat /dev/block/mmcblk0 | busybox gzip -fc >/mnt/sdcard/alluser.bin.gz`  
然后在windows下解压到ntfs文件系统就行了.推荐的解压缩软件是7zip.  
(反向测试:请使用busybox gzip -dc alluser.bin.gz | busybox dd of=/dev/block/mmcblk0)  
### 11.4 镜像文件转换
#### 11.4.1 镜像文件裁剪
4G的flash在dump时用dd命令指定了大小,不再需要裁剪.
同样为了应对不同厂家flash的实际空间不同的情况,8G或者更大的flash,dump的是整个flash的镜像,需要对镜像文件进行裁剪,使用的工具是EditBox.  
以8G的flash为例:先计算擦除内容的起始位置,对于8G的flash,实际使用分区为7000M,这里保留7100M的数据,所以从7101M开始擦除,计算结果为:  
`7101*1024*1024`转换为16进制为`1BBD00000`  
![calc](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20180201143606353.png)  
使用EditBox软件打开alluser.bin文件，使用`ctrl+a`或者在菜单选择编辑->选择全部,然后使用`ctrl+e`或者在菜单选择编辑->选择块,由于之前执行过全选，此时可以在结束偏移的位置看到这个文件的具体大小：  
![editbox](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20180201143913586.png)  
在起始偏移中输入之前计算结果的值，这时选中的就是要删除的内容,使用delete删除这些选中的内容,然后将剩余内容全选并选择块,可以看到剩余内容的总大小:  
![datasize](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20180201144024739.png)  
计算这个大小可以看到最终保留的大小为7100M.  
```
1BBCFFFFF=7445938175(转换为10进制)
7445938175/1024/1024=7100
```
将删除后的文件另存为usr.bin,这个user.bin就是后续步骤使用的镜像文件.  
#### 11.4.2 压缩镜像文件
使用软件`EPROG 1.7.8`对镜像文件进行压缩,  
![eprog](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20180130165137336.png)  
选中镜像点击start后等待进度完成后,会在镜像同目录下生成压缩后的文件,文件格式为文件名称为user.C.IMG  
#### 11.4.3 合并镜像文件
使用软件`SianenCM.exe`合并boot0.bin，boot1.bin和user.C.IMG,  
![SianenCM](http://ovuoa7tvt.bkt.clouddn.com/markdown-img-paste-20180130165619955.png)  
按照图上的配置进行合并，合并后会在同目录生成合并后的镜像user_.C.IMG,此文件为最终烧录文件,修改为需要的文件名提供给工厂生产.  
### 11.5 镜像验证
#### 11.5.1 确保从机顶盒dump出的原始镜像正确性
提供将原始镜像反烧录回机顶盒的方法，确认原始镜像的正确性.  
U盘根目录存放之前dump出的镜像user.bin,boot0.bin,boot1.bin,连接U盘:至另一台机顶盒,进入recovery,挂载U盘把之前dump出的镜像写回另一台盒子,重启过一下简单功能即正确.  
挂载U盘:  
```
busybox mkdir -p /mnt/sdcard
busybox mount -t vfat /dev/block/sda1 /mnt/sdcard
cd /mnt/sdcard/
```
验证boot0和boot1:  
```
busybox dd if=/mnt/sdcard/boot0.bin of=/dev/block/mmcblk0boot0 
busybox dd if=/mnt/sdcard/boot1.bin of=/dev/block/mmcblk0boot1
```
mmcblk0需要对应4G和8G以上(含8G)的flash命令分别为:  
**4Gflash**:`busybox dd if=/mnt/sdcard/user.bin  of=/dev/block/mmcblk0 bs=4096 count=952320`  
**8G及以上**:`busybox gzip -dc alluser.bin.gz | busybox dd of=/dev/block/mmcblk0`  
#### 11.5.2 确保使用烧录器压缩工具和合并工具后制作的烧录文件正确性
保证至少两人使用dump出的镜像文件进行压缩,合并,制作最后烧录器需要的生产镜像,并比对**每个过程**的文件md5,需保证md5一致.
#### 11.5.3 确保烧录过程的正确性
利用烧录器将烧录后emmc的内容读取出来,与原始镜像对比,依次对比boot1,boot2,user三个分区.目前测试只要emmc容量大小一致(测试了三星和东芝两款),对比文件md5都是一致的,所以emmc容量一致时,对比md5即可.  
将导出的三个镜像的md5值发送给工厂,与烧录器读取的emmc的三个镜像的md5进行对比,一致即正确.  
对应emmc替代料容量不一致时,使用EditBox软件可以对比二进制文件,实测win10上打开和对比7G+文件速度很快,测试md5一致的文件对比结果为完全相同,文件内容一致但大小不一致的文件对比结果为文件相同但文件大小不同.  
相关软件的下载地址如下:  
`链接：https://pan.baidu.com/s/1sm3z9sL 密码：3ml8`  
