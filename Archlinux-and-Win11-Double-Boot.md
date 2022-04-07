# ArchLinux and Win11 Double Boot 配置

## 1.制作启动盘
使用rufus(Windows系统)制作archlinux启动盘，或者是使用`dd` 命令(Linux系统).
如果原系统是Windows的话，需把`Fast boot`和`Security boot`关掉。

## 2.分区，格式化
(注意！需按实际情况更改，否则原系统可能损坏)
原Windows系统算在盘为`nvme0n1`，新加的硬盘为`nvme1n1`

## 双系统启动配置
在`/etc/default/grub`文件中uncomment掉`GRUB_DISABLE_OS_PROBER=false`这行。然后执行
```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
或者是执行命令
```sh
sudo update-grub
```
(需要安装update-grub)
```sh
yay -S update-grub
```
以重新生成grub配置文件.

可以使用grub-customizer图形界面程序来配置grub,同样的修改后也是要重新生成grub配置文件.

---

gnome桌面：
```sh
sudo pacman -S gnome gnome-shell-extensions
sudo pacman -S gnome-shell-extensions-simple-net-speed
yay -S gnome-shell-extension-dash-to-dack 
yay -S gnome-shell-extension-topicons-plus
```
界面字体太小:
进入tweaks软件把fonts的Scaling Factor调成1.30左右,具体数值自己调整,然后再调整fonts的大小.
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

添加archlinux中国仓:

```
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

添加到`/etc/pacman.conf`中。

添加arch4edu源(Optional)

```sh
sudo pacman -Syu
sudo pacman -S archlinuxcn-keyring
```

oh-my-zsh:
插件：web_search 可在终端直接查询，浏览器显示.
使用方法,在终端直接查询如使用duckduckgo查询
```sh
ddg your_problem
```
zsh theme
```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
zsh:
启用vim模式,在.zshrc中添加
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

切换默认的shell为zsh
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

废弃该项,直接使用ssh认证,查看该repo下的
git 无法clone,需要生成token替代password,然后再cache
```sh
git config --global credential.helper cache
```

安装网络模型查看器Netron:
```sh
git clone https://aur.archlinux.org/netron-bin.git
```
修改PKGBUILD文件
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

key相关:
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

gonme更换主题,图标:
苹果WhiteSur主题和图标
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
McSierra Compact主题和deepin plus all icon theme图标
Deepin Theme
```sh
yay -S adapta-gtk-theme-git
sudo pacman -S deepin-icon-theme
```
把下载的主题解压放在~/.themes中,
把下载的图标解压放在~/.icons中，
使用tweaks工具修改主题和图标.

安装中文字体:
```sh
sudo pacman -S noto-fonts-cjk
```
终端直接查看图片:
```sh
sudo pacman -S ranger w3m
```
然后在.config/ranger/rc.conf文件中添加
```
set preview_images true
```


更改`~/dot_files`目录下所有文件的权限，不包括文件夹:
(去掉所有用户的文件执行权限)
```sh
chmod a-x -f `find ~/dot_files -type f`
```

rCore操作系统课程的开发环境配置:
C开发环境：
```sh
sudo pacman -S gdb-multiarch riscv64-linux-gnu-gcc riscv64-linux-gnu-binutils
```
Rust开发环境:
```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup install nightly
rustup default nightly
```
安装一些Rust相关的软件包:
```sh
rustup target add riscv64gc-unknown-none-elf
cargo install cargo-binutils --vers=0.3.3
rustup component add llvm-tools-preview
rustup component add rust-src
```
安装Qemu模拟器:
```sh
sudo pacman -S qemu qemu-arch-extra
```

GDB调试支持:
`riscv64-unknown-elf-gdb`,调试器包含在 riscv64 gcc 工具链中.
直接安装从Aur中安装
```sh
yay -S riscv-gnu-toolchain-bin
sudo pacman -S gdb-dashboard
```
或者下载PKGBUILD文件，删掉不必要的包，自己安装
```sh
git clone https://aur.archlinux.org/riscv-gnu-toolchain-bin.git
```
可能会报错缺少libpython3.8.so.1.0,网上下载一下或者直接把最新的包复制过去
```sh
cp /usr/lib/libpython3.10.so.1.0 /usr/lib/libpython3.8.so.1.0
```

vscode中无法搜索到一些插件:
(Enable vscode marketplace in Code OSS)
```sh
yay -S code-marketplace
```

Postgresql数据库:
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
创建与linux user同名的PostgreSQL role/user
```
[postgres]$ createuser --interactive
```
创建数据库
```sh
createdb your_db_name
```
安装图形化界面pgadmin4
```sh
sudo pacman -S pgadmin4
```

tmux配置:
配置文件`~/.tmux.conf`中添加
```
set -g prefix C-Space
unbind C-b
bind C-Space send-prefix
```

AI开发环境:
anaconda
```sh
sudo pacman -S anaconda
```
在`~/.zshrc`文件中PATH变量添加`/opt/anaconda/bin`.
创建一个新环境
```
conda create -n pytorch pytorch torchvision cpuonly -c pytorch
```
在~/.aliases文件中添加
```
alias activate='source /opt/anaconda/bin/activate' \
      deactivate='conda deactivate'
```
```
activate pytorch
deactivate
```

远程桌面Remmina:
```sh
sudo pacman -S remmina
```

更换默认截屏器为flameshot
```sh
sudo pacman -S flameshot
```
Settings-> Keyboard-> Keyboard Shortcuts-> Custom Shortcuts:
Command框填
```
flameshot gui
```

对换Caps 和Esc键在VSCode中不起作用:
在`/home/rick/.config/Code - OSS/User/settings.json`文件中添加:
```
"keyboard.dispatch": "keyCode"
```

双显卡切换:
```sh
sudo pacman -S optimus-manager bbswitch
sudo pacman -S nvidia
```
把gdm替换成gdm-prime
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

查看命令行用法tldr
```sh
pacman -S tldr
```

npm修改默认目录
```sh
mkdir ~/.npm-global
```
添加环境变量
```
NPM_CONFIG_PREFIX=~/.npm-global
```

llvm和clang
```sh
sudo pacman -S clang llvm
```

pdf阅读器和管理工具:
```sh
sudo pacman -S zathura zotero
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
