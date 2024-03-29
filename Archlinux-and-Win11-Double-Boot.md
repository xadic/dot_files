# ArchLinux and Win11 Double Boot 配置

## 1.制作启动盘

使用 rufus(Windows 系统)制作 archlinux 启动盘，或者是使用`dd` 命令(Linux 系统).
如果原系统是 Windows 的话，需把`Fast boot`和`Secure boot`关掉。

## 2.分区，格式化

(注意！需按实际情况更改，否则原系统可能损坏)
原 Windows 系统所在盘为`nvme0n1`，新加的硬盘为`nvme1n1`

## 双系统启动配置

在`/etc/default/grub`文件中 uncomment 掉`GRUB_DISABLE_OS_PROBER=false`这行。然后执行

```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

或者是执行命令

```sh
sudo update-grub
```

(需要安装 update-grub)

```sh
yay -S update-grub
```

以重新生成 grub 配置文件.

可以使用 grub-customizer 图形界面程序来配置 grub,同样的修改后也是要重新生成 grub 配置文件.

-------------------------------------

KDE Plasma 桌面：
参考[archlinux 简明指南](https://arch-linux.osrc.com/)
```sh
sudo pacman -S plasma-meta konsole dolphin # plasma-meta 元软件包、konsole 终端模拟器和 dolphin 文件管理器
sudo pacman -S gwenview okular  # gwenview 图片查看器、okular pdf阅读器
```

高分屏下字体太小:
System Settings -> Fonts -> Force font DPI 改成120.(根据实际情况修改)

System Settings -> Display and Monitor -> Display Configuration -> Global scale: 125%

更换主题：
System Settings -> Appearance -> Global Theme: Layan

```sh
sudo pacman -S kvantum
```
打开Kvantum Management，选择`Layan` Theme，保存.

System Settings -> Appearance -> Application Style: kvantum

毛玻璃效果：
System Settings -> Workspace Behavior -> Blur

System Settings -> Windows Management -> Kwin Scripts: Install `Force Blur`
然后把alacritty和其他需要使用毛玻璃效果的软件填写在配置中.
(注意：如果毛玻璃效果不起作用，将Global scale改成整数倍)



Krunner字体太小：
参考[如何放大Krunner的字体](https://bigshans.github.io/post/how-to-increase-krunner-font/)
先退出Krunner
```sh
kquitapp5 krunner
```

然后修改`~/.config/krunnerrc`文件如下：
```sh
[General]
font=Noto Sans,20,-1,5,50,0,0,0,0,0
[PlasmaRunnerManager]
migrated=false
```

Latte Dock：
```sh
sudo pacman -S latte-dock-git # 使用git结尾的包
```
相关配置选项
Behavior:
    Screen: Bottom
    Alignment: Center
    Visibility: Dodge Active
    Delay:
        Show: none
        Hide: none
    Always use floating gap for user interaction : uncheck this option
Appearance:
    Items:
        Absolute size: 88px
        Zoom on hover: 0%
    Margins:
        Length: 1%
        Thickness: 24%
        Floating gap: 63px
    Background:
        Thickness: 85%
        Opacity: 2%
        Radius: 16px
        Blur
        Shadows

```sh
sudo pacman -S ark # 压缩软件
```

安装小部件Widgets(Avalon Menu、Split Digital Clock、Netspeed Widget, Event Calendar)

`tela-icon-theme`主题图标：
```sh
sudo pacman -S tela-icon-theme-git
```

修改任务切换器：
System Settings -> Window Management -> Task Switcher -> Main -> Thumbnails

关掉锁屏后音乐停止：
System Settings -> Power Management -> Advanced Power Settings -> Disable Pause media players when suspending

关掉低电量降低亮度：
System Settings -> Power Management -> Energy Saving -> On Low Battery uncheck Screen brightness


字体安装：
```sh
sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei # 安装几个开源中文字体。一般装上文泉驿就能解决大多 wine 应用中文方块的问题
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra # 安装谷歌开源字体及表情
```

Windows字体安装：
```sh
sudo mkdir /usr/share/fonts/WindowsFonts
cd /path/to/C:/Windows/Fonts # 或者通过 Dolphin 在此文件夹下右键 > 点击 打开终端
sudo cp ./* /usr/share/fonts/WindowsFonts
sudo chmod 644 /usr/share/fonts/WindowsFonts/* # 设置合理的权限
fc-cache -vf # 刷新字体
```

-------------------------------------

gnome 桌面：

```sh
sudo pacman -S gnome gnome-shell-extensions
sudo pacman -S gnome-shell-extensions-simple-net-speed
yay -S gnome-shell-extension-dash-to-dack
spi gnome-shell-extension-appindicator
yay -S gnome-shell-extension-topicons-plus
yay -S gnome-shell-extensions-no-overview
yay -S gnome-shell-extension-clipboard-indicator
```

dash-to-dack 不兼容 gnome 的话可以下载安装 ubuntu dock

```sh
git clone https://github.com/micheleg/dash-to-dock.git -b ubuntu-dock
cd dash-to-dock
make
make install
```

界面字体太小:
进入 tweaks 软件把 fonts 的 Scaling Factor 调成 1.30 左右,具体数值自己调整,然后再调整 fonts 的大小.
打开夜灯过滤蓝光.
更改按键绑定
Settings-> Keyboard-> Keyboard Shortcuts-> Windows:

```
Close window: Super+Q
Maximize window: Super+K
Restore window: Super+J
```

Settings-> Keyboard-> Keyboard Shortcuts-> Custom Shortcuts:

```
Alacritty: Super+Return
```

添加 archlinux 中国仓:

```
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

添加到`/etc/pacman.conf`中。

添加 arch4edu 源(Optional)

```sh
sudo pacman -Syu
sudo pacman -S archlinuxcn-keyring
```

oh-my-zsh:
插件：web_search 可在终端直接查询，浏览器显示.
使用方法,在终端直接查询如使用 duckduckgo 查询

```sh
ddg your_problem
```

zsh theme

```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

zsh:
启用 vim 模式,在.zshrc 中添加

```
bindkey -v
export KEYTIMEOUT=1
```

fish shell:
```sh
sudo pacman -S fish fisher
chsh -s /usr/bin/fish
curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install | fish
fisher install veirt/web-search.fish
fisher install PatrickF1/fzf.fish
```

蓝牙问题：

```sh
sudo pacman -S bluez bluez-utils
sudo rfkill unblock bluetooth
systemctl restart bluetooth.service
```

开机自动连接蓝牙鼠标:

```sh
systemctl enable bluetooth.service
bluetoothctl
scan on
trust *MAC_address*
```

切换默认的 shell 为 zsh

```sh
chsh -s /usr/bin/zsh
```

vscode:
调节界面字体大小,按住`Ctrl+=`,或`Ctrl+-`,使用*Settings Sync*插件同步
下载Microsoft版
```sh
yay -S visual-studio-code-bin
```

git:

```sh
git config --global user.name "Your Name"
git config --global user.email "youremail@yourdomain.com"
git config --global init.defaultBranch main
```

废弃该项,直接使用 ssh 认证,查看该 repo 下的
git 无法 clone,需要生成 token 替代 password,然后再 cache

```sh
git config --global credential.helper cache
```

如果遇到git 无法 通过SSH方式clone的,在`~/.ssh/config`文件中添加以下内容:
```
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```


安装网络模型查看器 Netron:

```sh
git clone https://aur.archlinux.org/netron-bin.git
```

修改 PKGBUILD 文件

```sh
cd netron-bin
makepkg -s
sudo pacman -U netron-bin-5.6.6-1-x86_64.pkg.tar.zst
```

远程桌面：

```sh
yay -S todesk-bin
systemctl start todeskd.service

yay -S sunloginclient
systemctl start runsunloginclient.service
```

## 问题处理

key 相关:

```sh
sudo pacman -S archlinux-keyring
asudo pacman -S archlinuxcn-keyring
```

有些应用中无法使用中文输入法:
在`/etc/profile`文件中添加

```
export GTK_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE=fcitx
```

搜狗输入法:

```sh
yay -S fcitx-sogoupinyin
```

使用新的fcitx5输入法:
```sh
sudo pacman -S fcitx5-im # 输入法基础包组
sudo pacman -S fcitx5-chinese-addons # 官方中文输入引擎
sudo pacman -S fcitx5-anthy # 日文输入引擎
sudo pacman -S fcitx5-rime # Rime输入法
sudo pacman -S rime-cloverpinyin # 四叶草拼音输入方案
```
创建`~/.local/share/fcitx5/rime/default.custom.yaml`文件：
```yml
patch:
  "menu/page_size": 8
  schema_list:
    - schema: clover
```

更换主题
```
git clone https://github.com/fkxxyz/ssfconv.git
cd ssfconv
pip install pycryptodome
```

下载搜狗皮肤 "就这·龙小可.ssf"
```
./ssfconv -t fcitx5 就这·龙小可.ssf 就这·龙小可
cp -r 就这·龙小可 ~/.local/share/fcitx5/themes/
```


gonme 更换主题,图标:
苹果 WhiteSur 主题和图标

```sh
cd ~/Documents
git clone git@github.com:vinceliuice/WhiteSur-gtk-theme.git
cd WhiteSur-gtk-theme
./install.sh -i arch
sudo ./tweaks.sh -g -b "path/to/lockscreen/picture"

git clone git@github.com:vinceliuice/WhiteSur-icon-theme.git
cd WhiteSur-icon-theme
./install.sh
```

McSierra Compact 主题和 deepin plus all icon theme 图标
Deepin Theme

```sh
yay -S adapta-gtk-theme-git
sudo pacman -S deepin-icon-theme
```

把下载的主题解压放在~/.themes 中,
把下载的图标解压放在~/.icons 中，
使用 tweaks 工具修改主题和图标.

安装中文字体:

```sh
sudo pacman -S noto-fonts-cjk
```

终端直接查看图片:

```sh
sudo pacman -S ranger w3m
```

然后在.config/ranger/rc.conf 文件中添加

```
set preview_images true
```

更改`~/dot_files`目录下所有文件的权限，不包括文件夹:
(去掉所有用户的文件执行权限)

```sh
chmod a-x -f `find ~/dot_files -type f`
```

rCore 操作系统课程的开发环境配置:
C 开发环境：

```sh
sudo pacman -S gdb-multiarch riscv64-linux-gnu-gcc riscv64-linux-gnu-binutils
```

Rust 开发环境:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup install nightly
rustup default nightly
```

安装一些 Rust 相关的软件包:

```sh
rustup target add riscv64gc-unknown-none-elf
cargo install cargo-binutils --vers=0.3.3
rustup component add llvm-tools-preview
rustup component add rust-src
```

安装 Qemu 模拟器:

```sh
sudo pacman -S qemu qemu-arch-extra
```

GDB 调试支持:
`riscv64-unknown-elf-gdb`,调试器包含在 riscv64 gcc 工具链中.
直接安装从 Aur 中安装

```sh
yay -S riscv-gnu-toolchain-bin
sudo pacman -S gdb-dashboard
```

或者下载 PKGBUILD 文件，删掉不必要的包，自己安装

```sh
git clone https://aur.archlinux.org/riscv-gnu-toolchain-bin.git
```

可能会报错缺少 libpython3.8.so.1.0,网上下载一下或者直接把最新的包复制过去

```sh
cp /usr/lib/libpython3.10.so.1.0 /usr/lib/libpython3.8.so.1.0
```

vscode 中无法搜索到一些插件:
(Enable vscode marketplace in Code OSS)

```sh
yay -S code-marketplace
```

neovim配置:
```sh
git clone https://github.com/LunarVim/Neovim-from-scratch.git ~/.config/nvim
```
```sh
sudo pacman -S prettier xclip lazygit ripgrep fd htop
pip install pynvim
```

`Ctrl+h`: 跳到左侧菜单
`Ctrl+l`: 跳到右侧区域
`Shift+h`: 向左跳动一个文件
`Shift+l`: 向右跳动一个文件
`Space`: 自定义主键
`Space+e`: 开关菜单栏
`Space+f`: 打开文件搜索
`Space+Shift+f`: 打开项目文本搜索
`Space+l+f`: 格式化
`g+l`: 查看错误提示
`g+d`: 查看定义
`g+r`: 查看引用

Notion笔记:
```sh
sudo pacman -S notion-app-enhanced
```
(notion打开卡在加载中进不去，执行这个[链接](https://github.com/notion-enhancer/notion-repackaged/issues/116#issuecomment-1783275997)中的脚本)

Postgresql 数据库:

```sh
sudo pacman -S postgresql
sudo su - postgres
```

Before PostgreSQL can function correctly, the database cluster must be initialized:

```
[postgres]$ initdb -D /var/lib/postgres/data
```

启动服务

```sh
systemctl start postgresql.service
```

创建与 linux user 同名的 PostgreSQL role/user

```
[postgres]$ createuser --interactive
```

创建数据库

```sh
createdb your_db_name
```

postgresql数据库版本升级

1、停止服务，升级数据库，安装postgresql-old-upgrade
```sh
sudo systemctl stop postgresql.service
sudo pacman -S postgresql postgresql-libs postgresql-old-upgrade
```

2、备份数据库文件
```sh
# mv /var/lib/postgres/data /var/lib/postgres/olddata
# mkdir /var/lib/postgres/data /var/lib/postgres/tmp
# chown postgres:postgres /var/lib/postgres/data /var/lib/postgres/tmp

sudo su - postgres
[postgres]$ cd /var/lib/postgres/tmp
```

3、升级
```sh
[postgres]$ initdb -D /var/lib/postgres/data
[postgres]$ pg_upgrade -b /opt/pgsql-14/bin -B /usr/bin -d /var/lib/postgres/olddata -D /var/lib/postgres/data
```

4、
```sh
systemctl start postgresql.service
[postgres]$ vacuumdb --all --analyze-in-stages
```





安装图形化界面 pgadmin4

```sh
sudo pacman -S pgadmin4
```

tmux 配置:
配置文件`~/.tmux.conf`中添加

```
set -g prefix C-Space
unbind C-b
bind C-Space send-prefix
```
zellij:
```sh
sudo pacman -S zellij
```

u盘自动挂载（dolphin无法挂载u盘处理）:
在`/etc/udev/rules.d/99-usb-mount.rules`文件中添加:
```
ACTION=="add", SUBSYSTEMS=="usb", SUBSYSTEM=="block", ENV{ID_FS_USAGE}=="filesystem", RUN{program}+="/usr/bin/systemd-mount --no-block --automount=yes --collect $devnode /run/media/rick"
```

```sh
udo systemctl restart systemd-udevd.service
```

AI 开发环境:
anaconda

```sh
sudo pacman -S anaconda
```

在`~/.zshrc`文件中 PATH 变量添加`/opt/anaconda/bin`.
创建一个新环境

```
conda create -n pytorch pytorch torchvision cpuonly -c pytorch
```

在~/.aliases 文件中添加

```
alias activate='source /opt/anaconda/bin/activate' \
      deactivate='conda deactivate'
```

```
activate pytorch
deactivate
```

远程桌面 Remmina:

```sh
sudo pacman -S remmina
```

安装virtualbox:
```sh
sudo pacman -S virtualbox virtualbox-host-modules-arch
sudo pacman -S virtualbox-ext-oracle
sudo gpasswd -a $USER vboxusers
```
VirtualBox虚拟机与主机之间复制粘贴设置以及文件拖拽

```
Settings→General→Advanced:
Shared Clipboard: Bidirectional
Drag’n’Drop: Bidirectional

Settings→Storage→Controller: SATA→Use Host I/O Cache

Settings→Storage→Win 7.vdi→Solid-state Drive

Devices→Insert Guest Additions CD Images (下载失败的话到[https://download.virtualbox.org/virtualbox/7.0.14/](https://download.virtualbox.org/virtualbox/7.0.6/))下载VBoxGuestAdditions_7.0.14.iso，然后Devices > Optical Drives > Choose a disk file
然后进入虚拟机CD盘中安装VirtualBox Guest Additions
```
让主机和虚拟机能通讯
```
Settings→Network→Adapter 1→Attached to：把NAT改成Bridged Adapter模式
```
共享文件夹
```
Devices→Shared Folders→Add new shared folders: 选择主机一个挂载点，然后将auto-mount勾上.
```


更换默认截屏器为 flameshot:

```sh
sudo pacman -S flameshot
```

Settings-> Keyboard-> Keyboard Shortcuts-> Custom Shortcuts:
Command 框填

```
flameshot gui
```

如果要临时对换Caps键与Esc键:
```sh
setxkbmap -option "caps:swapescape" -layout "cn"
```

对换 Caps 和 Esc 键在 VSCode 中不起作用:
在`/home/rick/.config/Code - OSS/User/settings.json`文件中添加:

```
"keyboard.dispatch": "keyCode"
```

Caps键与Esc键对换：(推荐此方法)
新建文件`/etc/X11/xorg.conf.d/00-keyboard.conf`,内容如下
```sh
Section "InputClass"
        Identifier "system-keyboard"
        MatchIsKeyboard "on"
	    Option "XkbOptions" "caps:swapescape"
        Option "XkbLayout" "en"
        Option "XkbModel" "pc105"
EndSection
```
(如果不起作用，可能是被fcitx5等覆盖,查看相应的取消方法[Fcitx5 Disable overriding XKB settings](https://wiki.archlinux.org/title/Fcitx5#Disable_overriding_XKB_settings),
或者将上面`00-keyboard.conf`文件中的XkbLayout的en改成cn)


或者是使用`interception-caps2esc`:
```sh
sudo pacman -S interception-caps2esc
```
新建`/etc/interception/udevmon.yaml` 文件：
```yml
- JOB: intercept -g $DEVNODE | caps2esc | uinput -d $DEVNODE
  DEVICE:
    EVENTS:
      EV_KEY: [KEY_CAPSLOCK, KEY_ESC]
```

```sh
sudo systemctl enable udevmon
sudo systemctl restart udevmon
```

双显卡切换:

```sh
sudo pacman -S optimus-manager bbswitch
sudo pacman -S nvidia
yay -S optimus-manager-qt
```

把 gdm 替换成 gdm-prime

```
sudo pacman -R gdm
sudo pacman -S gdm-prime
```

`/etc/gdm/custom.conf`文件中注释掉这行

```
#WaylandEnable=false
```

在`/etc/optimus-manager/optimus-manager.conf`文件中添加

```
[optimus]
switching=bbswitch
pci_power_control=no
pci_remove=no
pci_reset=no
```

或

```
[optimus]
switching=nouveau
pci_power_control=yes
pci_remove=no
pci_reset=no
```

切换使用方法

```
optimus-manager --switch nvidia
optimus-manager --switch integrated
optimus-manager --switch hybrid
```

或使用 optimus-manager-qt 切换.

查看当前显卡使用模式

```sh
optimus-manager --print-mode
```

查看命令行用法 tldr

```sh
pacman -S tldr
```

npm 修改默认目录

```sh
mkdir ~/.npm-global
```

添加环境变量

```
NPM_CONFIG_PREFIX=~/.npm-global
```

llvm 和 clang

```sh
sudo pacman -S clang llvm
```

pdf 阅读器和管理工具:

```sh
sudo pacman -S zathura zotero
```

没有声音：

```sh
sudo pacman -S pipewire-pulse
```

终端默认打开chrome浏览器:
```sh
xdg-settings set default-url-scheme-handler http google-chrome.desktop
```

数据库 GUI:

```sh
sudo pacman -S dbeaver
```

dbeaver 中显示问题看不清:
Preferences => User Interface => Appearance 中 把 Enable theming 取消勾选.

程序降级:

```sh
sudo pacman -S downgrade
sudo downgrade v2ray
```

Java 版本:

```sh
sudo pacman -S jdk8-openjdk
archlinux-java status
sudo archlinux-java set java-8-openjdk
```

```sh
sudo archlinux-java set java-19-jdk
```

路径:

```
/usr/lib/jvm/java-8-openjdk
```

gnome 桌面图标以及 terminal:
安装 nemo 文件浏览器

```sh
sudo pacman -S nemo
```

设为默认文件浏览器

```sh
xdg-mime default nemo.desktop inode/directory application/x-gnome-saved-search
```

显示桌面图标

```sh
gsettings set org.nemo.desktop show-desktop-icons true
nemo-desktop
```

修改~/.config/nemo/desktop-metadata

```
[desktop-monitor-0]
nemo-icon-view-keep-aligned=true
nemo-icon-view-auto-layout=false
nemo-icon-view-layout-timestamp=1666096734
```

创建nemo.desktop文件，将文件复制到~/.config/autostart目录下，内容如下:

```
[Desktop Entry]
Type=Application
Name=Desktop Icons
Exec=nemo-desktop
OnlyShowIn=GNOME;
NoDisplay=true
X-GNOME-Autostart-Phase=Desktop
X-GNOME-Autostart-Notify=true
X-GNOME-AutoRestart=true
X-GNOME-Provides=filemanager
```

把 alacritty 设为默认终端

```sh
gsettings set org.cinnamon.desktop.default-applications.terminal exec alacritty
gsettings set org.cinnamon.desktop.default-applications.terminal exec-arg -e
```

```sh
cp /usr/share/applications/vim.desktop ~/.local/share/applications/
```

把 ~/.local/share/applications/vim.desktop 修改为

```
#TryExec=vim
#Exec=vim %F
#Terminal=true
Exec=alacritty -e vim %F
Terminal=false
```

取消 ideavim 铃声:
在~/.ideavimrc 中

```
set visualbell
set noerrorbells
```
ideavim使用剪切板:
```
set clipboard+=unnamed
```

安装idea:
配置代理：
File->Settings->System Settings->HTTP Proxy->Manual proxy configuration

配置Maven:
File->Settings->Build,Execution,Deployment->Maven

rofi配置:
```sh
sudo pacman -S rofi
```
在~/.config/rofi/config.rasi中
```
configuration {
 modes: "drun";
 font: "hack 25";
 show-icons: true;
 icon-theme: "WhiteSur-Dark";
 }
@theme "lb"

entry {
    placeholder: "C'est le seul vide que je comblerai peute";
}
element-icon {
    size: 1.4em;
}
entry, element-text {
/*    text-color: grey; */
    background-color: rgba(0,0,1,0.0);
}

element-text {
    vertical-align: 0.5;
}
```

xclip:
复制到剪切板
```sh
cat "your_file" | xclip
```

粘贴到X应用程序以外
```sh
cat "your_file" | xclip -selection clipboard
```

vim粘贴格式错乱:
进入paste模式
```
:set paste
```

快速搜索:
fzf,ripgrep,skim,bat
在~/.zshrc中添加
```
# load fzf function
if [ -f /usr/share/fzf/key-bindings.zsh ]; then
    source /usr/share/fzf/key-bindings.zsh
fi
if [ -f /usr/share/fzf/completion.zsh ]; then
    source /usr/share/fzf/completion.zsh
fi
```
在~/.my_env中添加
(窗口预览)
```
# fzf config within vim(neovim)
export FZF_DEFAULT_COMMAND='rg --files --hidden'
export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --preview '(highlight -O ansi {} || cat {}) 2> /dev/null | head -500'"
```

## Dev

rustdesk 开发环境:

```sh
sudo pacman -Syu --needed unzip git cmake gcc curl wget yasm nasm zip make pkg-config clang gtk3 xdotool libxcb libxfixes alsa-lib pulseaudio
```

vcpkg:

```sh
cd ~/Build
git clone https://github.com/microsoft/vcpkg
cd vcpkg
./bootstrap-vcpkg.sh -disableMetrics
export VCPKG_ROOT=$HOME/Build/vcpkg
```

```sh
./vcpkg install libvpx libyuv opus
```

把`export VCPKG_ROOT=$HOME/Build/vcpkg`放到`~/.zshrc`中

安装微信 企业微信
```sh
yay -S deepin-wine-wechat
yay -S com.qq.weixin.work.deepin-x11
```
在~/.my_env中添加
```
DEEPIN_WINE_SCALE=1.25
```
修改DPI
```
/opt/apps/com.qq.weixin.deepin/files/run.sh winecfg
```

安装nvm管理node版本
```sh
sudo pacman -S nvm
nvm install 16
nvm use 16
```

fish 使用nvm
```
fisher install jorgebucaran/nvm.fish
```

安装pnpm
```sh
sudo pacman -S pnpm
```

Rss 阅读器
```sh
sudo pacman -S fluent-reader
```

法语词典
```sh
yay -S frhelper
```

grex语法生成:
```sh
sudo pacman -S grex
```

sql server数据库：
```
yay -S mssql-server
sudo /opt/mssql/bin/mssql-conf setup
yay -S azuredatastudio-bin
```
无法还原数据库时无法在/home 下面找到时，将备份的数据库复制到/tmp 下.

clion终端使用fish时报错：

创建脚本 /home/your_name/scripts/launch_fish.sh
```bash
#!/bin/sh
if [ -n "$OLD_XDG_CONFIG_HOME" ]; then
  export XDG_CONFIG_HOME="$OLD_XDG_CONFIG_HOME"
else
  unset XDG_CONFIG_HOME
fi
exec fish
```
修改clion的terminal的路径为上面的脚本。

设置文件绑定打开程序
```bash
xdg-mime query filetype ~/Documents/test.txt
xdg-mime query default text/plain
xdg-mime default lvim.desktop text/plain
xdg-open ~/Document/test.txt
```


## 参考资料

[How to Dual Boot Arch Linux and Windows 10 on UEFI System](https://www.linuxtechi.com/dual-boot-arch-linux-windows-10/)

[How to Update GRUB on Arch Linux](https://linuxhint.com/update_grub_arch_linux/)

[Using the Zsh shell in Vim mode](https://koenwoortman.com/zsh-vim-mode/)

[Manjaro Update fails: signature is unknown trust](https://unix.stackexchange.com/questions/518432/manjaro-update-fails-signature-is-unknown-trust)
[remote: Support for password authentication was removed](https://namespaceit.com/blog/remote-support-for-password-authentication-was-removed-on-august-13-2021-please-use-a-personal-access-token-instead)
[WhiteSur-gtk-theme](https://github.com/vinceliuice/WhiteSur-gtk-theme)

[Add arch4edu to your Archlinux](https://github.com/arch4edu/arch4edu/wiki/Add-arch4edu-to-your-Archlinux)
[tmux](https://github.com/tmux/tmux/wiki/Getting-Started)
[How to make Flameshot the default Linux screenshot app](https://www.addictivetips.com/ubuntu-linux-tips/flameshot-default-linux-screenshot-app/)

[optimus-manager](https://github.com/Askannz/optimus-manager)
[optimus-manager FAQ](https://github.com/Askannz/optimus-manager/wiki/FAQ,-common-issues,-troubleshooting#after-trying-to-switch-gpus-i-am-stuck-with-a-black-screen-or-a-black-screen-with-a-blinking-cursor-or-a-tty-login-screen)

[Permission denied when installing npm modules](https://stackoverflow.com/questions/47252451/permission-denied-when-installing-npm-modules-in-osx)

[rCore-Tutorial 实验环境配置](https://rcore-os.github.io/rCore-Tutorial-Book-v3/chapter0/5setup-devel-env.html#id2)
[riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain)

[Powerlevel10k](https://github.com/romkatv/powerlevel10k#configuration-wizard)
[Nerd Fonts](https://github.com/ryanoasis/nerd-fonts)

[Zotero, LaTeX, Sublime/VS Code, Git, Vim, Markdown, & Pandoc](http://www.benbrastmckie.com/tools)

[](https://www.pling.com/p/1013741)
[](https://www.pling.com/p/1261886/)

[How to make st the default terminal in linuxmint cinnamon](https://unix.stackexchange.com/questions/594242/how-to-make-st-the-default-terminal-in-linuxmint-cinnamon)

[web-search.fish](https://github.com/Veirt/web-search.fish)
[](https://zaiste.net/posts/shell-commands-rust/)

[Can I use Vim to edit the notes in Zotero directly?](https://forums.zotero.org/discussion/64893/can-i-use-vim-to-edit-the-notes-in-zotero-directly)
[Arch wiki:nemo](https://wiki.archlinux.org/title/Nemo)
[gnome3.28及以后桌面图标显示方法](https://www.jianshu.com/p/b4ece1c0acec)

[安装KDE Plasma桌面环境](https://arch-linux.osrc.com/rookie/desktop-env-and-app.html#_4-%E5%AE%89%E8%A3%85-kde-plasma-%E6%A1%8C%E9%9D%A2%E7%8E%AF%E5%A2%83)
[udev: Mounting drives in rules](https://wiki.archlinux.org/title/Udev)
[ssh: connect to host github.com port 22: Connection timed out](https://stackoverflow.com/questions/15589682/ssh-connect-to-host-github-com-port-22-connection-timed-out)
