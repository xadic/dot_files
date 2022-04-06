---
title: Install and Configure Arch Linux
date: 2019-05-05 19:53:04
updated:
tags:
- Arch Linux
- Linux
categories:
- Linux
mathjax:
---

Steps to install Arch Linux and configure it for daily use and work.

<!-- more -->


## Pre-installation

### Verify signature

Use **GnuPG** to verify downloaded package.

```bash
gpg --keyserver pgp.mit.edu --keyserver-options auto-key-retrieve --verify archlinux-<version>-x86_64.iso.sig
```

### Build bootable live environment

Plug in USB and check its device name by

```bash
lsblk
```

Asume its name is **/dev/sdb**, then use **dd** to build the bootable system to usb (**warning** before press "ENTER", double check the command to prevent from destroying system)

```bash
dd if=/way/to/arch/file of=/dev/sdb
```

### Verify  the boot mode

```bash
ls /sys/firmware/efi/efivars
```

If the directory does not exist, the system may be booted in **BIOS** or **CSM** mode, otherwise, in **UEFI** mode. If it's in **UEFI** mode, go ahead.

Plug in USB, shut down computer, enter into BIOS system interface with holding **F2** then press power button. (Different motherboards diverse in entering BIOS)

If you can't see a boot option "boot from USB", go to create a boot option by select your boot file which end with **.efi**. And then put this option to number 1 boot option. OK, save change and reboot system.

--------------------------------------------------------------------------------------------------------------------

Now we had entered the Arch live system.

### Connect to the Internet

The installation image enables the **dhcpcd** daemon for wired network devices on boot. The connection can be verified with **ping**, If you use wifi, you should configure wifi with **wifi-menu**

```bash
wifi-menu
```

```bash
ping -c 3 baidu.com
```

### Update the system clock

Use **timedatectl** to ensure the system clock is accurate

```bash
timedatectl set-ntp true
```

(To check the service status, use )

```bash
timedatectl status
```

### Partition the disks

When recognized by the live system,disks are assigned to a **block device**. To  identify these devices, use **lsblk** or **fdisk -l**

```bash
lsblk
```

The following *partitions* are required for a chosen device:

- One partition for the root directory **/** .

- If **UEFI** is enabled, an  **EFI system partition** .

    (The following are alternative)

- **Swap** space can be set on a separate partition or a **swap file** .

- **/home** directory for users .

(*Note* : If you want to create any stacked block devices for **LVM**, **disk encryption**, or **RAID**, do it now.)

To modify **partition tables**, use **cfdisk**, **fdisk** or **parted**.

```bash
cfdisk /dev/sda
```

my partition plan for system with 4G RAM and 500G storage device

```
  /dev/sda1		 200M		/boot/efi  	EFI System
  /dev/sda2		 250G	    /		    Linux fs
  /dev/sda3		 remaining  /home	    Linux fs
```

### Format the partitions

Once the partitions have been created, each must be formatted with an appropriate **file system**.

```bash
mkfs.fat -F32 /dev/sda1

mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
```

### Mount the file systems

Mount the file system on the root partition to **/mnt**

```bash
mount /dev/sda2 /mnt
```

Create mount points for any remaining partitions and mount them accordingly

```bash
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi

mkdir /mnt/home
mount /dev/sda3 /mnt/home

# Assume your Windows system EFI patition /dev/sda4
mkdir -p /mnt/boot/efi-win
mount /dev/sda4 /mnt/boot/efi-win
```

**genfstab** will later detect mounted file systems and swap space.

---------------------------------

We have accomplished the work for Pre-Installation, now enter the process of Installation.

## Installation

### Select the mirrors

Packages to be installed must be downloaded from mirror servers, which are defined in **/etc/pacman.d/mirrorlist** . Use **vim** to select your country's mirror to the top.

```bash
vim /etc/pacman.d/mirrorlist
```

### Install the base packages

Use the **pacstrap** script to install the **base** and **base-devel** package groups:

```bash
pacstrap /mnt base base-devel
```
Linux with firmware:
```bash
pacstrap /mnt linux linux-firmware
```
wifi:
```bash
pacstrap /mnt dialog wpa_supplicant dhcpcd netctl networkmanager
```

------------------

## Configure the system

### Fstab

Generate an **fstab** file (use **-U** or **-L** to define by *UUID* or *labels*, respectively)

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

and check the resulting file.

### Chroot

Change root into the new system:

```bash
arch-chroot /mnt
```

### Configure wifi

```bash
wifi-menu
```
or
```bash
systemctl start NetworkManager
nmcli device wifi connect your_wifi_SSID password your_password
```

### Time zone

```bash
ln -sf /usr/share/zoneinfo/Asian/Shanghai /etc/localtime
```

Generate **/etc/adjtime**

```bash
hwclock --systohc
```

This command assumes the hardware clock is set to **UTC**.

### Localization

Uncomment **en_US.UTF-8 UTF-8** and other needed **locales** in **/etc/locale.gen**,and generate them

```bash
locale-gen
```

Set the **LANG** variable in **/etc/locale.conf**

```
LANG=en_US.UTF-8
```

### Network configuration

Create the hostname file **/etc/hostname**

```bash
echo 'Arch' > /etc/hostname
```

​Add matching entries to **/etc/hosts**

```bash
echo -e "127.0.0.1\tlocalhost\n::1\t\tlocalhost\n127.0.1.1\tArch.localdomain\tArch" > /etc/hosts
```
If the system has a permament IP address, it should be used instead of **127.0.1.1** .

### Initramfs

Creating a new **initramfs** is usually not required, because **mkinitcpio** was run on installation of the **linux** package with **pacstrap** .

For special configurations, modify the **mkinitcpio.conf** file and recreate the initramfs image :

```bash
mkinitcpio -p linux
```

### Set root password

```bash
passwd
```

### Install bootloader

A Linux-capable boot loader must be installed in order to boot Arch Linux. If you have an Intel or AMD CPU, enable **microcode** updates. Let's install GRUB boot loader:

#### Download grub and efibootmgr

```bash
pacman -S grub efibootmgr
```

**GRUB** is the bootloader while **efibootmgr** is used by the GRUB installation script to write boot entries to **NVRAM** .

(Mount the EFI system partition at mount point, for example, ***/boot/efi*** )

#### For dual boot with Windows
```bash
pacman -S os-prober dosfstools
```

#### Install the GRUB EFI application

Install the GRUB EFI application **grubx64.efi** to **/boot/efi*/EFI/GRUB/** and install its modules to **/boot/grub/x86_64-efi/** .

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchLinux
```

(After the above install completed the main GRUB directory is located at **/boot/grub/** . Note that **grub-install** also tries to create an entry tin the fireware boot manager, named **GRUB** in the above example.)

#### Generated grub.cfg

Use the **grub-mkconfig** tool to generate **/boot/grub/grub.cfg**

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

(*Note*:

- Remember that **/boot/grub/grub.cfg** has to be re-generated after any change to **/etc/default/grub** or files in **/etc/grub.d** .
- After installing or removing a **kernel**, you just need to re-run the above **grub-mkconfig** command.)



### Reboot

Exit the chroot environment, then manually unmount all the partitions.Finally, restart the machine.

```bash
exit
umount -R /mnt
reboot
```



--------

## Post-installation

```bash
pacman -S vi vim
```
### Add a new user

```bash
useradd -m -G wheel -s /bin/bash wjw
passwd wjw
```

To grant sudo access to user *wjw* run **visudo** and add "**wjw ALL=(ALL) ALL**"

```bash
visudo
```

### Font

See available fonts

```bash
fc-list
```

Download fonts

```bash
sudo pacman -S wqy-microhei ttf-dejavu
yay -S ttf-symbola
```

or download packages manually and put font files to the **.local/share/fonts/** directory.

If you want to change console font, put font files in the **/usr/share/kbd/consolefonts** directory. Then go to see the console fonts, and select one..

```bash
ls /usr/share/kbd/consolefonts
echo -e "FONT=sun12x22\nFONT_MAP=8859-2" > /etc/vconsole.conf
```

### Graphical user interface

### Install Xorg

```bash
sudo pacman -S xorg-server xorg-xinit xorg-xinput
```

configure **~/.xinitrc** file

```bash
[[ -f ~/.Xresources ]] && xrdb -merge -I$HOME ~/.Xresources
```



### Install  i3-gaps  and related programs

Install i3-gaps and configure to start from **~/.xinitrc** . (**rofi** for replacing d-menu, **feh** for setting wallpaper, **light** for brightness control, **scrot** for screenshot, **i3blocks** for replacing i3status, **acpi** for i3blocks's battery module)

for "light":
```bash
usermod -aG video wjw
```
```bash
sudo pacman -S i3-gaps i3blocks i3locks rofi feh light scrot acpi termite bc
echo 'exec i3' > ~/.xinitrc
```

You can start up i3 manually by typing command **startx** or set it to automatically.If you use **zsh** ,put the following into the **~/.zprofile** to auto start X at login

```bash
if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
	exec startx
fi
```



### Install ssr

```bash
sudo pacman -S git wget python
wget http://www.djangoz.com/ssr
sudo mv ssr /usr/local/bin
sudo chmod 744 /usr/local/bin/ssr
ssr install
ssr config
```
### Install v2ray
```sh
sudo pacman -S v2ray
yay -S qv2ray
```

### Install trojan

(server deploy:
```bash
wget --no-check-certificate https://raw.githubusercontent.com/V2RaySSR/Trojan/master/Trojan.sh && chmod +x Trojan.sh && ./Trojan.sh
```
)

Download the file display in the server,then
```bash
sudo pacman -S trojan
unzip trojan-cli.zip
cd usr/src/trojan-cli
sudo cp config.json fullchain.cer /etc/trojan
systemctl start trojan
```

### Wireless configure

Install **dialog**, **wpa_supplicant** and **dhcpcd** for using **wifi-menu** . (or optionally Install **networkmanager** )

```bash
sudo pacman -S dialog wpa_supplicant
```

Enabling wifi auto connecting when boot. (use **netctl**)

```bash
sudo wifi-menu -o
```

use command above to generate the profile in **/etc/netctl** for use of **netctl**.

```bash
sudo netctl enable your_profile
```

### Sound

If no sound because of Master set to **HDMI**, go to change it to **PCM**.

```bash
sudo echo -e "defaults.pcm.card 1\ndefaults.pcm.device 0\ndefaults.ctl.card 1" > /etc/asound.conf
```

Download **pulseaudio** and related control command utils.

```bash
sudo pacman -S pulseaudio pulsemixer playerctl
```
And download **alsa-utils** for controling alsa
```
sudo pacman -S alsa-utils
```

### Useful program

```bash
sudo pacman -S firefox neofetch
```

### yay

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### lxappearance & qt5ct

qt5ct for qt5 based programs, while lxappearance for gtk based programs.

```bash
sudo pacman -S lxappearance qt5ct
```

gtk related configure files:

```
~/.gtkrc-2.0
~/.config/gtk-3.0/settings.ini
```
GTK Theme
```sh
sudo pacman -S adapta-gtk-theme
```
Icon theme
```sh
git clone https://github.com/horst3180/arc-icon-theme --depth 1 && cd arc-icon-theme
./autogen.sh --prefix=/usr
sudo make install
```

add the following environment variable to **~/.pam_environment**

```
QT_QPA_PLATFORMTHEME=qt5ct
```


### laptop power saving

```bash
sudo pacman -S tlp
```

then configure it:

```bash
systemctl enable tlp.service tlp-sleep.service
systemctl mask systemd-rfkill.service systemd-rfkill.socket
```

(*Note*: **tlp.service** starts **NetworkManager.service** if it is available. If you use a different network manager, mask **NetworkManager.service** or edit **tlp.service** and remove the service out of line **Wants=**)

### System proxy (Privoxy)

```bash
sudo pacman  -S privoxy
```

then edit **/etc/privoxy/config** file and add the following line the 5.2 section (note the **.** at the end)

```bash
forward-socks5 / 127.0.0.1:1080 .
```

It means that transact all http requests to socks5 and redirect to localhost's 1080 port (change this value to your ssr listening port)

then you should go to start and enable **privoxy.service** .

```bash
systemctl start privoxy.service
systemctl enable privoxy.service
```

Privoxy's default listening port is 8118, so your should set your http proxy address to **127.0.0.1:8118** .

```bash
export http_proxy="http://127.0.0.1:8118"
```

Configure git proxy

```bash
git config --global http.proxy 127.0.0.1:8118
```

### File manager

ranger (use w3m to preview image) and pcmanfm
```bash
sudo pacman -S ranger highlight w3m
sudo pacman -S pcmanfm
```

### tmux & tmux-resurrect

```bash
sudo pacman -S tmux
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

### Build package

use **asp** or **pbget**

```bash
sudo pacman -S asp
```

### Docker

Download docker, then create a group named docker and add user to this group for using docker without "sudo" command for regular user.

```bash
sudo pacman -S docker
sudo groupadd docker
sudo usermod -aG docker $USER
```

Then enable and start docker.service

```bash
systemctl enable docker.service
systemctl start docker.service
```

Setting proxy for docker:

```bash
mkdir /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/proxy.conf
```

put following content to the file above

```
[Service]
Environment="HTTP_PROXY=192.168.1.1:8118"
Environment="HTTPS_PROXY=192.168.1.1:8118"
```

Then reload units

```bash
systemctl daemon-reload
```

Check whether setting successfully

```bash
systemctl show docker --property Environment
```

### Tensorflow

Download **nvidia-docker**

```bash
yay -S nvidia-docker
```

Pull latest **tensorflow-gpu**

```bash
docker pull tensorflow/tensorflow:latest-gpu-py3
```

```bash
docker run --runtime=nvidia -it tensorflow/tensorflow:latest-gpu-py3 bash
```

(**Note:** `nvidia-docker` v1 uses the `nvidia-docker` alias, where v2 uses `docker --runtime=nvidia`.)

Docker Usage

```bash
docker ps -a	#check all containers
docker rm $(docker ps -aq) 	#delete all containers

docker images	#check all images
docker rmi $(docker images -q) #delete all images
docker rmi node_name	#delete node(image)

docker info
docker version

# create a container named tf and run it
# -it : interactive
# --name : assign the name "tf" to this container
# -p : mapping ports of container to host, first pair for Jupyter notebook, the second one for Tensorboard
# -v host_folder:container_folder : enables sharing a folder between the host and the container. The host folder should be inside your home directory. This folder is seen as notebooks directory in the container which is used by Ipython/Jupyter Notebook.
docker run -it -p 8888:8888 -p 6006:6006 -v $(pwd)/tensorflow:/notebooks --name tf tensorflow/tensorflow

docker start -i tf	#start container named tf with interactive mode
```

### Anaconda and Pytorch

Installing anaconda3 firstly, then create an environment named "pytorch" for pytorch compatible with cuda 10, lastly, enter this environment.

```bash
wget https://repo.anaconda.com/archive/Anaconda3-2020.07-Linux-x86_64.sh
./Anaconda3-2020.07-Linux-x86_64.sh

#conda create -n pytorch pytorch torchvision cuda100 -c pytorch
conda create -n pytorch pytorch torchvision cpuonly -c pytorch
source activate pytorch
```

Install packages in the `pytorch` environment we created before.
```
source /opt/anaconda/bin/activate pytorch
conda install opencv
conda install -c conda-forge opencv=4.2 (or this)
conda install scikit-learn
conda install -c conda-forge matplotlib
```

or use **pip** package manager to install pytorch

```bash
$ pip3 --no-cache-dir install torch
$ pip3 --no-cache-dir install torchvision
```

### xclip

```bash
 sudo pacman -S xclip
```

### github

Add ssh key :

```bash
ssh-keygen -t rsa -b 4096 -C "youremail@gmail.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
xclip -sel clip < ~/.ssh/id_rsa.pub

# change your origin remote to ssh url from https url
git remote set-url origin git@github.com:<Username>/<Project>.git
```

Go to github add a new key, then paste the content of the clipboard to it.

(If count problem "ssh: connect to host github.com port 22: Connection refused", go to create a config file for ssh)

```
Host github.com
Hostname ssh.github.com
Port 443
```

Put the content above to the file `~/.ssh/config`.

### Media

mpd + ncmpcpp + cantata + mps-youtube + youtube-dl + mpv

### Fuzzy Finder
```bash
sudo pacman -S fzf
```

### Text Editor
#### Vim & Neovim & as IDE
vim plugin manager: vim-plug
code-searching tool: the_silver_searcher (**ag**)
Plugins for making a IDE: coc.vim and its extensions (coc-python, coc-snippets, coc-vimtex)

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

```bash
sudo pacman -S vim neovim xclip python-pip
sudo pacman -S the_silver_searcher
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
sudo pip3 install python-language-server
yay -S ccls-git
#pip3 install --user --upgrade pynvim
python -m pip install --user --upgrade pynvim
```
```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

```sh
sudo pacman -S ctags
```

chemacs
```bash
git clone https://github.com/plexus/chemacs.git
cd chemacs
./install.sh
```
Use Doom-Emacs with chemacs
```bash
touch ~/.emacs-profiles.el
mkdir -p ~/Git/doom
```
put the content below to *~/.emacs-profiles.el*
```
(("default" . ((user-emacs-directory . "~/.emacs.d")))
 ("doom" . ((user-emacs-directory . "~/Git/doom")
            (env . (("DOOMDIR" . "~/.doom.d"))))))
```
```bash
git clone -b develop https://github.com/hlissner/doom-emacs ~/Git/doom
~/Git/doom/bin/doom -p ~/.doom.d quickstart
make install
emacs --with-profile doom &
```

Use Spacemacs with chemacs
```bash
mkdir -p ~/Git/spacemacs
git clone https://github.com/syl20bnr/spacemacs.git ~/Git/spacemacs
```

Doom-Emacs
```bash
git clone -b develop https://github.com/hlissner/doom-emacs ~/.emacs.d
~/.emacs.d/bin/doom quickstart
cd .emacs.d
make install
```

Emacs
Configure lsp-mode and pyls-ms (Microsoft Python Language Server)
```bash
sudo pacman -S dotnet-sdk
mkdir ~/build
cd build
git clone https://github.com/Microsoft/python-language-server.git
cd python-language-server/src/LanguageServer/Impl
dotnet build -c Release
```

### Openssh
```bash
sudo pacman -S openssh
```

Enable ssh daemon:
The *sshd*'s config file located in */etc/ssh/sshd_config*,
we can change the port from 22 to other port:
```bash
sudo systemctl start sshd
sudo systemctl enable sshd
```
(connect from another machine:
```bash
ssh -p SSH_PORT USERNAME@IP_ADDRESS
```
)

### Book Reader
```bash
sudo pacman -S zathura zathura-pdf-poppler zathura-pdf-mupdf
sudo pacman -S calibre
```
### Latex
```bash
sudo pacman -S texlive-most
```
### Sync tool
```bash
sudo pacman -S syncthing
```

### Input Method
fcitx
```bash
sudo pacman -S fcitx-im fcitx-configtool fcitx-mozc fcitx-sunpinyin
```
then add input method using *fcitx-configtool*.

(*OR* ibus
```bash
sudo pacman -S ibus ibus-rime ibus-anthy
```
Change the default chinese input method to simplified, within the file *~/.config/ibus/rime/build/luna_pinyin.schema.yaml* put *reset: 1* underneath the *name: simplification*.
)

### SwitchyOmega
In auto switch profile, in Rule list confifug, select AutoProxy, and put the following content into Rule list url.
```
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```
### Mount usb
See the usb's device name
```bash
dmesg | tail
```
said, */dev/sdb*, then go to make a mount point, and mount it
for example
```bash
sudo mkdir -p /mnt/media/usb
mount /dev/sdb1 /mnt/media/usb
```
```bash
umount /dev/sdb1
```
### Cron
```bash
sudo pacman -S cronie
```
### Draw figures
```bash
sudo pacman -S inkscape
```
### Color Schedule
```bash
sudo pacman -S python-pywal
```
### X hotkey daemon
```bash
sudo pacman -S sxhkd
```
### Menu
```bash
sudo pacman -S dmenu
```
### Notification
```bash
sudo pacman -S dunst
```

### Enable to connect to android
```bash
sudo pacman -S mtpfs gvfs-mtp gvfs-gphoto2
```
### Package Manager
```bash
sudo pacman -S yarn
```

### Compositor with blur
```sh
git clone https://github.com/tryone144/compton.git
sudo pacman -S libconfig asciidoc
make
make docs
make install
```
### Compositor
```bash
sudo pacman -S picom
```

### Swap Caps and Esc key
For console:
```bash
sudo mkdir -p /usr/local/share/kbd/keymaps
sudo echo -e "keycode 1 = Caps_Lock\nkeycode 58 = Escape" > /usr/local/share/kbd/keymaps/personal.map

sudo echo "KEYMAP=/usr/local/share/kbd/keymaps/personal.map" >> /etc/vconsole.conf
```

For X:
add a new file named .Xmodmap (for details see my dot files), then
```bash
xmodmap ~/.Xmodmap
```
(note: if you want to attach an external keyboard, you also need to execute the
above command)

### Touchpad

#### Permanent change
Put following content to file `/etc/X11/xorg.conf.d/40-touchpad.conf`
```
Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "on"
        Driver "libinput"
        Option "Tapping" "on"
EndSection
```

#### Runtime change
replace the following command with appropriate values.
```sh
sudo pacman -S xorg-xinput
xinput
xinput --list-props your_device_id
xinput set-prop your_device_id "want_to_change_prop" changed_number
```

### Network

#### networkmanager
Install networkmanager
```sh
sudo pacman -S networkmanager
nmcli device wifi list
nmcli device wifi connect your_wifi_SSID password your_password
```
#### netctl
- wireless
```sh
sudo pacman -S dialog wpa_supplicant dhcpcd
```
- wired
```sh
sudo pacman -S ifplugd
```
Find your ethernet interface using `ip link` command, and
start/enable the `netctl-ifplugd@interface.service` systemd unit. (note: change the above *interface* with your ethernet interface)

### Dict
```sh
sudo pacman -S goldendict translate-shell
```

### ADB
```sh
sudo pacman -S android-tools
```

### Nvidia Driver
```sh
sudo pacman -S nvidia nvidia-utils bbswitch
yay -S nvidia-xrun
```
and go to configure .nvidia-xinitrc

### VirtualBox

(note: while installing select *virtualbox-host-modules-arch*)
```sh
sudo pacman -S virtualbox
systemctl start systemd-modules-load.service
sudo modprobe vboxdrv
yay -S virtualbox-ext-oracle
sudo usermod -aG vboxusers $USER
```
Enable **Auto-resize Guest Display**:
*Devices* -> *Insert Guest Additions CD image*,
then go to guest system (i.e. your installed windows) to install the downloaded file *CD Drive VirtualBox Guest Additions*.
(select amd64)

Enable recognize USB:
- Install *virtualbox-ext-oracle*
- `sudo usermod -aG vboxusers $USER`
- go to the USB setting button belong to which virtual machine you want to use,
before start the machine, add your USB by clicking the first button in the rigtht side.

Update extension pack
```sh
sudo vboxmanage extpack install --replace Oracle_VM_VirtualBox_Extension_Pack-6.1.26.vbox-extpack
```

### Misc

GNU Debugger:
```sh
sudo pacman -S gdb
```
Memory Debugging:
```sh
sudo pacman -S valgrind
```
Oh-my-zsh:
```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestion

cd ~/dot_files/.oh-my-zsh/custom
cp my_custom.zsh ~/.oh-my-zsh/custom
cp themes/my_lambda.zsh-theme ~/.oh-my-zsh/custom/themes
```
Screen Recorder:
```sh
sudo pacman -S obs-studio

Screen Shot:
sudo pacman -S flameshot
```

Snap:
```sh
yay -S snapd
systemctl start snapd.socket
```

Godot game engine
Scons build tool:
```sh
sudo pacman -S scons
```
```sh
yay -S godot-git
```
Sticky notes:
```sh
yay -S qtpad
```

Image Viewer
```sh
sudo pacman -S sxiv
```
Unrar:
```sh
sudo pacman -S unrar
```

Weather
```sh
curl wttr.in
```
Static site generator
```sh
sudo pacman -S hugo
```

# ntfs format
```sh
sudo pacman -S ntfs-3g
```
### Bluetooth
```sh
sudo pacman -S bluez bluez-utils
systemctl start bluetooth.service
bluetoothctl
```
### MySQL/MariaDB
```sh
sudo pacman -S mariadb
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
systemctl start mariadb.service
sudo pip install mysql-connector
```

```sh
sudo mysql -u root -p
```
after enter into MariaDB, assign a password for root user, so you can enter MariaDB without 'sudo':

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password'
```

### Java
```sh
sudo pacman -S jre-openjdk jdk-openjdk
```

### tldr
```sh
sudo pacman -S tldr
```
### rclone
```sh
sudo pacman -S rclone
rclone config
rclone lsd remote:
rclone copy ~/Notes GoogleDrive:ArchNote
```
### Git GUI
```bash
yay -S gitkraken
```
###
```bash
sudo pacman -S nodejs npm
```

### Useful command
```bash
# extract .gz file with striping first two dir
tar xzvf other.tar.gz --strip-components=2

# start chromium with proxy
chromium --proxy-server="socks://localhost:1080"

# change shell to zsh
chsh -s /bin/zsh $USER
```
### References

1. [Installation guide](https://wiki.archlinux.org/index.php/Installation_guide)
2. [How do I start tensorflow docker jupyter notebook](https://stackoverflow.com/questions/33636925/how-do-i-start-tensorflow-docker-jupyter-notebook#)
3. [Configure Emacs, lsp-mode and pyls-ms](https://vxlabs.com/2018/11/19/configuring-emacs-lsp-mode-and-microsofts-visual-studio-code-python-language-server/)
4. [How I draw figures for my mathematical lecture notes using Inkscape](https://castel.dev/post/lecture-notes-2/)
5. [Dot files](https://github.com/wjwrobot/dot_files)

6. [hugo zzo](https://zzo-docs.vercel.app/zzo/gettingstarted/quickstart/)
