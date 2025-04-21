# Arch Linux Installation Guide
manual installation guide for arch linux

1. config partition table use `gdisk`
    * efi => 256MB
    * swap => 2GB (depends on RAM size)
    * root => the rest of disk size
2. create filesystem base on partition table
    * efi => `mk.vfat -F 32` 
    * root => `mk.ext4`
    * swap => `mkswap`
3. enable swap feature
    * activate swap => `swapon [target partition]`
    * check is swap is activated => `swapon -S`
4. mount efi and root partition
    * root => `mount [root partition] /mnt`
    * efi => `mount [efi partition] /mnt/boot`
5. install essential package
    * execute command => `pacstrap -K /mnt base linux-lts linux-headers-lts linux linux-headers linux-firmware`
6. generate fstab table list
    * execute command => `genfstab -U /mnt >> /mnt/etc/fstab`
7. change root command
    * execute command => `arch-chroot /mnt`
8. install bootloader
    * install package => `sudo pacman -S grub efibootmgr base-devel nano git`
    * install grub => `grub-install —efi-directory [target mounting efi partition]`
    * create grub config => `grub-mkconfig -o [target mounting efi partition]/grub/grub.cfg`
9. config pacman
    * find file `/etc/pacman.conf` edit like this below 

    ```bash
    Color
    ParallelDownloads=5
    ILoveCandy

    [multilib]
    include
    ```

    * download packaga reflector => `pacman -S reflector`
    * backup exisiting mirror list => `cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup`
    * add fastest mirror list => `reflector —verbose —latest 10 —protocol https —sort rate —save /etc/pacman.d/mirrorlist`
    * download pacman cache cleaner => `pacman -S pacman-contrib`
    * enable pacman cache cleaner => `systemctl enable paccache.timer`
10. install `yay`
    * clone repository => `git clone https://aur.archlinux.org/yay.git`
    * change dir to `yay` => `cd yay`
    * change config `makepkg` in file `/etc/makepkg.conf` uncomment line `MAKEFLAGS="-j2”` and edit that to be `MAKEFLAGS="-j$(nproc)”`
    * build package => `makepkg -si`
11. set timezone
    * setting timezone => `ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime`
    * sync hardware clock with system clock => `hwclock --systohc`
    * edit file `/etc/systemd/timesyncd.conf`

    ```bash
    [Time]
    NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
    FallbackNTP=0.pool.ntp.org 1.pool.ntp.org 0.fr.pool.ntp.org
    ```

    * enable systemd-timesyncd service => `systemctl enable systemd-timesyncd.service`
12. set localization
    * edit file `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8`
    * execute command => `locale-gen`
    * create file `etc/locale.conf` add `LANG=en_US.UTF-8`
13. set hostname
    * create file `/etc/hostname` add your hostname
14. set password for root
    * execute command => `passwd`
15. install `openssh`
    * execute command => `pacman -S openssh`
    * enable service => `systemctl enable sshd`
16. install `zsh` shell
    * execute command => `sudo pacman -S zsh`
17. create new user
    * create new user => `useradd -m -G wheel -s /bin/zsh [username]`
    * set password => `passwd [username]`
    * set env variable for editor => `export EDITOR=nano`
    * execute command => `visudo`
    * uncomment `%wheel ALL=(ALL:ALL) ALL`
18. install `networkmanager`
    * execute command => `pacman -S networkmanager`
    * enable `networkmanager` => `systemctl enable NetworkManager`
19. install desktop environment
    * Gnome
        a. execute command => `pacman -S xorg gnome gnome-tweaks gnome-extra`
        b. enable gnome display manager service => `systemctl enable gdm.service`
    * KDE Plasma
        a. execute command => `pacman -S xorg plasma kde-applications`
        b. enable kde display manager service => `systemctl enable sddm.service`
20. install microcode
    * check processor vendor => `lscpu | grep "Vendor ID"`
    * AMD => `pacman -S amd-ucode`
    * Intel => `pacman -S intel-ucode`
    * create grub config => `grub-mkconfig -o [target mounting efi partition]/grub/grub.cfg`
21. install graphic card driver
    * check graphic card vendor => `lspci | grep VGA`
    * AMD => `pacman -S mesa libva-mesa-driver lib32-mesa vulkan-radeon lib32-vulkan-radeon`
    * Nvidia => `pacman -S nvidia nvidia-lts nvidia-utils nvidia-settings`
    * Intel => `pacman -S mesa intel-media-driver lib32-mesa vulkan-intel lib32-vulkan-intel`
22. install bluetooth driver
    * execute command => `pacman -S bluez blueman bluez-utils`
    * load bluetooth kernel => `modprobe btusb`
    * enable bluetooth service => `systemctl enable bluetooth`
23. install firewall
    * execute command => `pacman -S ufw`
    * enable firewall service => `systemctl enable ufw`
24. install `xdg-user-dirs`
    * download package => `pacmans -S xdg-user-dirs`
25. install `preload`
    * download preload package => `yay -S preload`
    * enable preload service => `systemctl enable preload`
26. install `auto-cpufreq`
    * clone repository => `git clone https://github.com/AdnanHodzic/auto-cpufreq.git`
    * change dir => `cd /auto-cpufreq`
    * run installer => `./auto-cpufreq-installer`
    * install => `auto-cpufreq —install`
27. unmount all mounted partition
    * execute command => `umount -R /mnt`
28. after reboot and login to desktop environment set region & lang
