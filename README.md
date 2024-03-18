# Arch Linux and ZSH Installation and Configuration
## Prerequisites
- Secure Boot disabled
- Arch Linux Iso file on bootable USB

## Installing Arch Linux
1. Use `localectl list-keymaps` to list keyboard layouts, and `loadkeys @layout` to set desired layout
2. Use `cat /sys/firmware/efi/fw_platform_size` to verify boot mode. It should return **64**
3. Use `ping archlinux.org` to check internet connection
4. Use `timedatectl list-timezones` to list available time zones, and `timedatectl set-timezone @timezone` to set desired time zone
5. Partition your device using the following commands:
   - `lsblk`: list devices and partitions
   - `cfdisk` or `cfdisk @device`: open partitioning GUI to delete and add partitions. You should add an **EFI** partition(1GB), a **swap** partition(4GB - 8GB), a **root** partition and, optionally, a **home** partition
   - `mkfs.ext4 @root-partition` and `mkfs.ext4 @home-partition`: format root and home partitions to ext4 filesystem
   - `mkfs.fat -F32 @boot-partition`: format efi partition to fat32 filesystem
   - `mkswap @swap-partition`: Change partition to swap partition
6. Mount partitions using the following commands:
   - `mount @root-partition /mnt`: mount root to /mnt
   - `mount --mkdir @boot-partition /mnt/efi`: make directory /mnt/efi and mount boot to it
   - `mount --mkdir @home-partition /mnt/home`: make directory /mnt/home and mount home to it
   - `swapon @swap-partition`: turn on the swap partition
   
   ***Caution: the root partition must be mounted first!***
7. Use `pacstrap /mnt base linux linux-firmware @package-list` to install Arch Linux to your PC. These three packages are all that is currently needed, but we can add a few more basic ones: `sof-firmware base-devel linux-headers grub os-prober efibootmgr git wget man nano networkmanager`
8. Use `genfstab /mnt` to generate a list of mounted devices and check if all partitions are mounted and saved correctly. If so, use `genfstab /mnt > /mnt/etc/fstab` to redirect the output to the fstab file
9. Use `arch-chroot /mnt` to enter installed system
10. Use `ln -sf /usr/share/zoneinfo/@zone /etc/localtime` to set local time to desired time zone. After that, use `hwclock --systohc` to synchronize the system clock
11. Use `nano /etc/locale.gen`, find the line **en_US.UTF-8 UTF-8** and uncomment it. Use `locale-gen` to generate locale and, lastly, use `echo LANG=en_US.UTF-8 > /etc/locale.conf` to set language
12. Use `echo @hostname > /etc/hostname` to set hostname
13. Use `passwd` to set root password
14. Use `useradd -m -G wheel -s /bin/bash @username` to add user
15. Use `passwd @username` to set user password
16. Use `EDITOR=nano visudo`, find the line **%wheel ALL=(ALL:ALL) ALL** and uncomment it
17. Use `systemctl enable NetworkManager` to enable network manager
18. Use `grub-install --efi-directory=/efi @disk-name`, where **disk-name** is the device where your EFI partition resides. Use `grub-mkconfig -o /boot/grub/grub.cfg` to generate the grub configuration file
19. Use `umount -a` to unmount all partitions
20. Use `exit` to return to the USB system. Use `reboot` to reboot the system
21. Use `ping archlinux.org` to check internet connection
22. Use `sudo pacman -S plasma sddm` to install KDE Plasma Desktop Environment
23. Use `sudo pacman -S konsole kate firefox` to install basic programs
24. Use `sudo systemctl enable --now sddm` to enable Plasma on startup, and also enable it now

## Dual-booting
1. Use `mount @windows-efi-partition /mnt` to mount the windows boot partition
2. Use `sudo nano /etc/default/grub`, find the line **GRUB_DISABLE_OS_PROBER=false** and uncomment it
3. Use `sudo grub-mkconfig -o /boot/grub/grub.cfg` to reconfigure grub
4. Use `reboot` to reboot the system

## Installing Yay
1. Use `sudo pacman -Syu` to update the system
2. Use `cd ~` to move to home directory, use `git clone https://aur.archlinux.org/yay.git` to download yay, use `cd yay` to move to yay directory and, finally, use `makepkg -si` to build yay

## Installing Nvidia Graphics Drivers
For a more in-depth guide that supports older grapgics cards, check [here](https://github.com/korvahannu/arch-nvidia-drivers-installation-guide)

1. Use `sudo nano /etc/pacman.conf`, find the lines **[multilib]** and **Include = /etc/pacman.d/mirrorlist** and uncomment them
2. Use `yay -S nvidia nvidia-utils lib32-nvidia-utils nvidia-settings` to download driver and needed packages
3. Use `sudo nano /etc/default/grub`, find the line **GRUB_CMDLINE_LINUX_DEFAULT** and append `nvidia-drm.modeset=1` inside the quotes
4. Use `sudo grub-mkconfig -o /boot/grub/grub.cfg` to reconfigure grub
5. Use `sudo nano /etc/mkinitcpio.conf`, find the line **MODULES=()** and append `nvidia nvidia_modeset nvidia_uvm nvidia_drm` inside the parantheses. Find the line **HOOKS=(... kms ...)** and remove `kms`
6. Use `sudo mkinitcpio -P` to regenerate the initramfs
7. Use `cd ~` to move to home directory, use `wget https://raw.githubusercontent.com/korvahannu/arch-nvidia-drivers-installation-guide/main/nvidia.hook` to download **nvidia.hook**. Use `sudo mkdir /etc/pacman.d/hooks/` to create hooks directory, and use `sudo mv nvidia.hook /etc/pacman.d/hooks/` to move nvidia.hook to hooks directory
8. Use `source .zshrc` to load the plugins
9. Use `reboot` to reboot the system

## Installing ZSH
1. Use `sudo pacman -S zsh zsh-completions` to install zsh
2. Use `zsh` and follow the instructions

## Installing Oh My Zsh
1. Use `sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"` to install Oh My Zsh

## Installing OMZ Plugins
1. Use `git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions` to install autosugesstions
2. Use `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting` to install syntax highlighting
3. Use `git clone --depth 1 -- https://github.com/marlonrichert/zsh-autocomplete.git $ZSH_CUSTOM/plugins/zsh-autocomplete` to install autocomplete
4. Use ` git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search` to install history substring search
5. Use `nano .zshrc`, find the line **plugins=(...)** and append `archlinux zsh-autosuggestions zsh-syntax-highlighting zsh-history-substring-search` inside the paranthesis
6. Use `reboot` to reboot the system

## Installing PowerLevel10k Theme
1. Use `yay -S ttf-meslo-nerd-font-powerlevel10k` to install necessary font
2. Choose the downloaded font inside your terminal
3. Use `git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k` to download the theme
4. Use `nano .zshrc`, find the line **theme=(...)** and set it to powerlevel10k/powerlevel10k
5. Open your terminal and customize the theme
