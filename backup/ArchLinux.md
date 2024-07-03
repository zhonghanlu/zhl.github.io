---
title: ArchLinux install

date: 2022-9-19 16:34:03

description: WIN11 完整刷机 ArchLinux

keywords: ArchLinux

tags: 
  - ArchLinux
  - System

categories: ArchLinux
---

### 切换之前

​	为啥需要换做ArchLinux系统呢，作为一个开发就应该用属于开发的系统，好不好！！！ 其实也是因为自己买不起MAC。在众多的Linux系统中最终我选择了Arch。Ubuntu我装了一次很方便，从制镜像到安装完成，共花费一小时。但是这个Ubuntu一直出毛病，要不然是显卡不能用了，要不然是连不上网了，网卡都给我干丢了，反正就是一直出问题，然后就想着换一个系统。选了大蜥蜴和Arch，网上评论众多，想了想像我这种爱折腾的Java小开发，那不得整个Arch。果不其然整了一天，终于整出来了，下面细细听我道来   （部分转载别人安装教程）

### 制作镜像

​	制作镜像我选择的是U盘制作，U盘格式成FAT32形式的，如果U盘是FATex的也不是不可以。用这个玩意就可以轻松制作启动盘了

![image-20220919165841611](https://cdn.zenless.top/gh/zhonghanlu/PicGo/img/image-20220919165841611.png)

镜像下载直接搜ArchLinux官网即可

选择中国的镜像源 下载iso文件即可

会清空磁盘的嗷谨慎操作嗷，盘符里的小电影注意点嗷

### U盘启动基础安装

​	插上U盘进行重启，狂按12进行启动选择 ，我的拯救者狂按F12即可，选择U盘启动，U盘启动失败的就重新制启动盘，这个没办法。

​	进行U盘启动，启动完成选择第一个UEFI形式安装  （可以提前看下自己的BIOS是什么模式，记住将安全启动关闭，进入BIOS将Ser那个玩意关闭再次进行U盘启动）

![image-20220919170331056](https://cdn.zenless.top/gh/zhonghanlu/PicGo/img/image-20220919170331056.png)

等待黑屏刷日志

刷完即可进入安装程序

首先更新时间（*这一步也是检查你网络是否通常，记住WIFI如果连不上就别连了，我整了整整四五个小时WIFI没成功，最后找到我们运维大哥给我插根网线，唉。放心刷完系统之后进去是可以使用WIFI的）

```linux
timedatectl set-ntp true
```

列出磁盘：

```linux
fdisk -l
```

/dev/sda1 : EFI 系统分区，大小为 1024 MB，FAT32 格式。这为存储引导加载程序和引导所需的其他文件提供了空间。

/dev/sda2 ： 交换分区，4GB 大小。交换空间用于将虚拟内存扩展到已安装的物理内存 (RAM) 之外，或用于挂起磁盘支持。

/dev/sda3 ： Linux分区，剩余可用磁盘空间大小，EXT4格式。这是存储我们的 Arch Linux 操作系统、文件和其他信息的根 ( / ) 分区。

创建 EFI 系统分区

```linux
cfdisk /dev/sda 
```

选择 GPT 标签类型并点击 Enter 。

然后 从底部菜单中选择 Free Space 并点击 New 。您可以使用 Tab 或 箭头键浏览菜单选项。

(如果你不想要之前的数据，直接delete完，硬干)

以 MB ( 1024M ) 为单位键入分区大小，然后按 Enter 键。

在 /dev/sda1 分区仍然被选中的情况下，从底部菜单中选择 Type 并选择 EFI System 分区类型。 

现在，您已完成 EFI 系统分区的配置。

创建交换分区

现在让我们使用相同的过程创建 Swap 分区。再次选择剩余的 Free space 和 并点击 New 。

以 GB ( 4G ) 为单位键入分区大小，然后按 Enter 键。

在 /dev/sda2 分区仍然被选中的情况下，从底部菜单中选择 Type 并选择 Linux swap 分区类型。

现在，您已经完成了 Swap 分区的配置。

创建根分区

最后，您需要创建根 ( / ) 分区。再次选择剩余的 Free space 并点击 New 。

对于 ( / ) 大小，保留默认大小值。这意味着，所有剩余的可用空间。按 Enter 键。

在 /dev/sda3 分区仍然被选中的情况下，从底部菜单中选择 Type 并选择 Linux filesystem 分区类型。

现在，您已经完成了根分区的配置。

将更改写入磁盘

接下来，您需要保存所做的更改。选择 Write 从底部菜单和命中 Enter 。

键入 yes 并按下 Enter 键。

我们到此结束。选择 Quit 并按下 Enter 即可。

 创建文件系统

现在您已准备好磁盘分区，是时候在其上创建文件系统了。但是让我们首先通过运行来查看分区表摘要：

```linux
fdisk -l
```

该 /dev/sda 磁盘应该有三个分区（ /dev/sda1 ， dev/sda2 ，和 /dev/sda3 ）

**EFI分区格式化**

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda3
```

**创建swap分区**：

```linux
mkswap /dev/sda2
swapon /dev/sda2
```

**挂载分区：**

```linux
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

**查看磁盘分区情况**

```linux
lsblk -f
```

**更新为国内镜像源**

```linux
reflector --country China --age 72 --sort rate --protocol https --save /etc/pacman.d/mirrorlist
```

将最新的镜像源更新为国内的，保存在/etc/pacman.d/mirrorlist目录下

也可以手动替换到“/etc/pacman.d/mirrorlistg”中

```linux
Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.dgut.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
Server = https://mirror.redrock.team/archlinux/$repo/os/$arch
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.xjtu.edu.cn/archlinux/$repo/os/$arch
```

如果到这一步 大哥畅通无阻  那只能说牛批  ， 开始安装ARCH

### ArchLinux 安装

安装基本系统（包括linux内核以及基础软件包），这里相比参考文章多给了几个软件包，因为这些对用户来说还是比较重要的 ，有几种内核可以安装：

普通内核(linux linux-headers)
lts稳定版内核(linux-lts linux-lts-headers)
zen内核(linux-zen,linux-zen-headers)
按自己的需求安装就可以

这里需要提前说一下，linux-zen 内核不支持 nvidia 显卡，有这个需求的就别装了，如果是原版 linux 内核的话，就要做好随时滚挂的准备，最近的 5.18 内核更新就会导致 nvidia-5.15 版本驱动失效无法开机，如果你希望稳定使用，就选择 linux-lts 内核和linux-lts-headers，并安装相应的 nvidia-lts 驱动（后面会有详细说明），不过不用太担心，即便是系统安装完成，你也可以随时切换自己想要的内核版本。

建议安装LTS内核

```linux
pacstrap /mnt base linux-lts linux-lts-headers linux-firmware base-devel （LTS内核）
或者
pacstrap /mnt base linux linux-headers linux-firmware base-devel （普通内核）
```

如安装报错 ERROR: Failed to install packages to new root 

尝试以下命令再继续安装：

```linux
pacman -Sy archlinux-keyring
```

写入分区表：

```linux
genfstab -U /mnt >> /mnt/etc/fstab
```

 使用如下命令进入新系统，进入后会显示[root@archiso /]#

```linux
arch-chroot /mnt
```

配置系统
我解释一下这都是干嘛的。

vim：文本编辑器，可替代有neovim，nano,其中nano对新手比较友好，推荐经验较少或者刚入坑的同志使用。
iwd,networkmanager：用iwd作为nm的backend进行使用。（但是我这样使用在i3下会出现不少问题，比如网络经常自动断，且短时间无法重连等(KDE也会出现，但感觉没有i3频繁，我觉得可能是命令行的原因，安装完成之后卸载掉networkmanager问题解决。另外，如果需要使用网线和usb网络共享，networkmanager必须安装，最好加装一个dhcpcd）
ttf-dejavu：开源字体
sudo：用于非root用户暂时获取root权限
bluez：蓝牙模块
usbmuxd：参考文章没给这个。现在系统中使用的网络来自于live系统，不装这个的话，重启是无法通过usb连接手机共享网络的，根据个人情况选择，建议安装。
wqy-zenhei：中文字体，避免进入系统后无法显示中
dhcpcd：连网线用
pacman -S neovim iwd ttf-dejavu sudo bluez usbmuxd networkmanager dhcpcd wqy-zenhei
neovim和vim的启动命令是不一样的，neovim为“nvim”，vim是“vim”我一般会通过软链接

```linux
ln -s /bin/nvim /bin/vim
ln -s /bin/nvim /bin/vi
```

 将他们链接起来看，这样的话，通过“vim”“vi”命令也可以启动neovim了。

设置时区和时间

依次输入下面的命令：

```linux
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

hwclock --systohc
```

 设置语言

输入“vim /etc/locale.gen”，删除前面的“#”，保存。

![image-20220919171722856](https://cdn.zenless.top/gh/zhonghanlu/PicGo/img/image-20220919171722856.png)

输入

```linux
locale-gen

echo LANG=zh_CN.UTF-8 >> /etc/locale.conf
```

设置root用户的密码

输入“passwd”，再输入密码，密码不会显示。

设置主机名

主机名的开头和结尾字符不允许是连字符。 主机名强烈建议不要用数字开头,尽管这一条不是强制的。 用小写字母而不用大写字母。

echo 主机名 >> /etc/hostname
设置网络

编辑 “vim /etc/hosts”

```linux
127.0.0.1 localhost
::1 localhost
127.0.1.1 主机名.localdomain 主机名
```

安装引导并重启系统
pacman -S grub efibootmgr   (安装grub)
grub-install /dev/sda    (注意：选择的是整个硬盘)
创建grub配置文件

```linux
grub-mkconfig -o /boot/grub/grub.cfg
```

 重启系统

exit    (退出新系统，回到live系统)
umount -R /mnt    (递归卸载 /mnt 中的磁盘)
reboot    (重启)
重启后登录root，密码是刚才设置的

打开联网服务和蓝牙
1. 首先激活服务

```linux
systemctl start iwd.service
systemctl enable iwd.service
systemctl start systemd-resolved.service
systemctl enable systemd-resolved.service
systemctl enable bluetooth.service
systemctl enable NetworkManager
systemctl enable dhcpcd
```

2. 配置网络连接和DNS

输入“vim /etc/iwd/main.conf”，把下面的文字打进去，保存。

```linux
[General]
EnableNetworkConfiguration=true
NameResolvingService=systemd
```

3. 安装了netwokmanager的配置

如果安装了networkmanager，则需要将backend服务修改为iwd，总体上iwd比wpa好用很多。

输入“vim /etc/NetworkManager/NetworkManager.conf”，把下面的文字打进去，保存。（vim的光标移动、删除和保存退出等命令请另行百度。）

```linux
[device]
wifi.backend=iwd
```

如果只安装networkmanager而不安装iwd的话，是不需要这一步的，nm会使用默认的wpa。（猜测）

安装一些硬件设备
	1.CPU编码

```linux'
pacman -S intel-ucode    (intel的cpu装这个)
pacman -S amd-ucode    (amd的cpu装这个)
```

注意是 CPU 编码，不是显卡

2. 显卡驱动

```linux
pacman -S xf86-video-intel（Intel核心显卡驱动，用Intel核显就装，否则不用装）
pacman -S mesa nvidia(-lts) nvidia-settings nvidia-dkms nvidia-utils nvidia-prime（nvidia显卡驱动，用nvidia显卡就装，否则不用装）
pacman -S xf86-video-amdgpu (AMD显卡驱动，用amd显卡的就装)
这里举两个例子，我的笔记本，i7-11代，搭配intel核显以及3050显卡，所以安装前两个。我的台式机，e3-1230垃圾CPU，搭配HD6950显卡，所以装第三个。nvidia-dkms 与 nvidia-lts 不兼容，如果装lts驱动的话无需安装dkms 。

注意：nvidia驱动的安装与前面选择的内核有关，如果你安装的是linux-lts内核，那么需要将nvidia更换为nvidia-lts，linux-zen不支持nvidia显卡（务必对号入座）
```

新建一个用户
```linux
useradd -m -G wheel -s /bin/bash 用户名    (添加进入wheel用户组，并将bash作为启动命令)
```

passwd 用户名
然后输入

visudo
如果报错的话(应该不会)，就改为

```linux
vim /etc/sudoers
```

 找到如下内容，取消掉“# %wheel ALL=(ALL:ALL) ALL”前面的“# ”

```linux
## User privilege specification
##
root ALL=(ALL:ALL) ALL
 
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL:ALL) ALL
 
## Same thing without a password
# %wheel ALL=(ALL:ALL) NOPASSWD: ALL
```

这里说一下，取消“# %wheel ALL=(ALL:ALL) NOPASSWD: ALL”前的“# ”也是可以的，区别就在于，取消这一行后，wheel组的用户使用 sudo 就不用输密码了，如果你是新手，不建议这么做，如果你是老鸟，可以考虑取消NOPASSWD 所在的这一行。（我取消的是NOPASWD这行，图方便）

重启系统
```linux
reboot
```

此时的系统已经基本可以使用了，但是还没有配置图形界面，如果你不需要图形界面，就只需登陆用户名就可以使用了。

### ArchLiux 桌面安装

！！！注意：从这里开始，如果登陆的是普通用户，则所有的pacman和systemctl都需要 sudo ， 如果嫌麻烦，可以先在 ~/.bashrc 中添加 “alias pacman='sudo pacman'和alias systemctl='sudo systemctl'”，我这里就不多写sudo了。如果提示需要权限，同样加sudo即可。所以这部分安装建议登陆root用户。

首先需要选择X11或者是Wayland，现在来看Wayland是比较先进的，但为了方便和兼容性还是用X11吧。

```linux
pacman -S xorg
```

这个是必须安装的，后面的DE和WM都是基于x服务。

**安装KDE桌面**

特点：美观，比较稳定，自定义功能强大，配置方便（最推荐）

```linux
pacman -S plasma sddm konsole dolphin kate ark okular spectacle yay
```

重启

```linux
systemctl enable sddm
```

**安装Gnome桌面**

特点：自定义功能丰富（但是被阻隔了），

```bash
pacman -S gnome
systemctl enable gdm
```

我安装的是Gnome桌面，不要问我为什么不装第一个，第一个装不上去，焯！！！！

**中文输入法**

推荐使用 fcitx5

```undefined
sudo pacman -S fcitx5 fcitx5-chinese-addons fcitx5-gtk fcitx5-qt fcitx5-configtool
```

然后添加环境变量

```linux
sudo vim /etc/environment

GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
INPUT_METHOD=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

'然后设置开机启动即可（桌面环境不同，方法也不同），这里提供一个简单的思路。

tty 下面是不需要中文输入法的（也用不了），所以修改 ~/.xinitrc ，添加代码

```linux
fcitx5 -d &    (后台运行fcitx5，且开机自启)
```

wiki描述

```linux
注意：
如果您使用的桌面环境是兼容 XDG 的（例如 KDE、GNOME、Xfce、LXDE等），则 无需 此步骤（添加自启）。
如果使用i3、awesome等窗口管理器，需要在其脚本中添加 Fcitx5 以实现自启动。例如，如果您使用 i3 或 sway ,可以在配置文件中添加exec --no-startup-id fcitx5 -d
如果使用dwm，则需要添加 autostart 补丁。在 ~/.dwm/autostart.sh 中添加fcitx5 -d
```

### 系统切换完成

开始安装环境吧

![image-20220919172627761](https://cdn.zenless.top/gh/zhonghanlu/PicGo/img/image-20220919172627761.png)









写在最后：

​	组成了众多文章：

1. CSDN

```url
https://blog.csdn.net/love906897406/article/details/126109464?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166357655316800180660005%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166357655316800180660005&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-126109464-null-null.142^v47^body_digest,201^v3^control&utm_term=archlinux%E5%AE%89%E8%A3%85&spm=1018.2226.3001.4187
```

2.知乎

```url
https://zhuanlan.zhihu.com/p/433920079
```



