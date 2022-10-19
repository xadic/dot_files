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

---

gnome 桌面：

```sh
sudo pacman -S gnome gnome-shell-extensions
sudo pacman -S gnome-shell-extensions-simple-net-speed
yay -S gnome-shell-extension-dash-to-dack
spi gnome-shell-extension-appindicator
yay -S gnome-shell-extension-topicons-plus
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

更换默认截屏器为 flameshot

```sh
sudo pacman -S flameshot
```

Settings-> Keyboard-> Keyboard Shortcuts-> Custom Shortcuts:
Command 框填

```
flameshot gui
```

对换 Caps 和 Esc 键在 VSCode 中不起作用:
在`/home/rick/.config/Code - OSS/User/settings.json`文件中添加:

```
"keyboard.dispatch": "keyCode"
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
archlinux-java set java-8-openjdk
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
