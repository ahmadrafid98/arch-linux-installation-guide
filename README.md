# Arch Linux Installation Guide
manual installation guide for arch linux

1. config partition table use `gdisk`
    * efi => 1GB
    * root => the rest of disk size
2. create filesystem base on partition table
    * efi => `mk.vfat -F 32` 
    * root => `mk.btrfs`
3. create root btrfs subvolume
    * mount partition => `mount [root partition] /mnt`
    * create subvolume => `btrfs subvolume create @`
    * unmount root partition => `umount /mnt`
4. mount efi and root partition
    * root => `mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@ [root partition] /mnt`
    * efi => `mount [efi partition] /mnt/boot`
5. install essential package
    * execute command => `pacstrap -K /mnt base linux-lts linux-headers-lts linux linux-headers linux-firmware`
6. generate fstab table list
    * execute command => `genfstab -U /mnt >> /mnt/etc/fstab`
7. change root command
    * execute command => `arch-chroot /mnt`
8. install `grub` bootloader
    * install package => `sudo pacman -S grub efibootmgr base-devel vim git acpid btrfs-progs`
    * install grub => `grub-install —efi-directory /boot --bootloader-id=GRUB --target=x86_64-efi`
    * create grub config => `grub-mkconfig -o /boot/grub/grub.cfg`
9. edit `mkinitcipio`
    * open config => `vim /etc/mkinitcipio.conf`
    * edit this line => `MODULES=(btrfs)`
    * regenerate image for linux kernel => `mkinitcipio -p linux`
    * regenerate image for linux-lts kernel => `mkinitcipio -p linux-lts`
    * recreate grub config => `grub-mkconfig -o /boot/grub/grub.cfg`
10. config pacman
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
11. install `yay`
    * clone repository => `git clone https://aur.archlinux.org/yay.git`
    * change dir to `yay` => `cd yay`
    * change config `makepkg` in file `/etc/makepkg.conf` uncomment line `MAKEFLAGS="-j2”` and edit that to be `MAKEFLAGS="-j$(nproc)”`
    * build package => `makepkg -si`
12. set timezone
    * setting timezone => `ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime`
    * sync hardware clock with system clock => `hwclock --systohc`
    * edit file `/etc/systemd/timesyncd.conf`

    ```bash
    [Time]
    NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
    FallbackNTP=0.pool.ntp.org 1.pool.ntp.org 0.fr.pool.ntp.org
    ```

    * enable systemd-timesyncd service => `systemctl enable systemd-timesyncd.service`
13. set localization
    * edit file `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8`
    * execute command => `locale-gen`
    * create file `etc/locale.conf` add `LANG=en_US.UTF-8`
14. set hostname
    * create file `/etc/hostname` add your hostname
15. set password for root
    * execute command => `passwd`
16. install `openssh`
    * execute command => `pacman -S openssh`
    * enable service => `systemctl enable sshd`
17. install `zsh` shell
    * execute command => `sudo pacman -S zsh`
18. create new user
    * create new user => `useradd -m -G wheel -s /bin/zsh [username]`
    * set password => `passwd [username]`
    * set env variable for editor => `export EDITOR=vim`
    * execute command => `visudo`
    * uncomment `%wheel ALL=(ALL:ALL) ALL`
19. install `networkmanager`
    * execute command => `pacman -S networkmanager`
    * enable `networkmanager` => `systemctl enable NetworkManager`
20. install desktop environment
    * Gnome
        a. execute command => `pacman -S xorg gnome gnome-tweaks`
        b. enable gnome display manager service => `systemctl enable gdm.service`
    * KDE Plasma
        a. execute command => `pacman -S xorg plasma kde-applications`
        b. enable kde display manager service => `systemctl enable sddm.service`
21. install microcode
    * check processor vendor => `lscpu | grep "Vendor ID"`
    * AMD => `pacman -S amd-ucode`
    * Intel => `pacman -S intel-ucode`
    * create grub config => `grub-mkconfig -o [target mounting efi partition]/grub/grub.cfg`
22. install graphic card driver
    * check graphic card vendor => `lspci | grep VGA`
    * AMD => `pacman -S mesa libva-mesa-driver lib32-mesa vulkan-radeon lib32-vulkan-radeon`
    * Nvidia => `pacman -S nvidia nvidia-lts nvidia-utils nvidia-settings`
    * Intel => `pacman -S mesa intel-media-driver lib32-mesa vulkan-intel lib32-vulkan-intel`
23. install bluetooth driver
    * execute command => `pacman -S bluez blueman bluez-utils`
    * load bluetooth kernel => `modprobe btusb`
    * enable bluetooth service => `systemctl enable bluetooth`
24. install firewall
    * execute command => `pacman -S firewalld`
    * enable firewall service => `systemctl enable firewalld.service`
25. install `xdg-user-dirs`
    * download package => `pacmans -S xdg-user-dirs`
26. install `preload`
    * download preload package => `yay -S preload`
    * enable preload service => `systemctl enable preload`
27. install `zram` for swap
    * install package => `yay -S zramd`
    * enbale service => `systemctl enbale --now zramd.service`
28. install `auto-cpufreq`
    * clone repository => `git clone https://github.com/AdnanHodzic/auto-cpufreq.git`
    * change dir => `cd /auto-cpufreq`
    * run installer => `./auto-cpufreq-installer`
    * install => `auto-cpufreq —install`
29. install tools for backup snapshot
    * install packages => `pacman -S timeshift grub-btrfs inotify-tools timeshift-autosnap`
    * manually create grub snapshot entries => `/etc/grub.d/41_snapshots-btrfs`
    * recreate grub config => `grub-mkconfig -o /boot/grub/grub.cfg`
    * edit btrfsd config => `sudo systemctl edit --full grub-btrfsd`
    * edit this line => `ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto`
    * enable grub-btrfsd service => `systemctl enable grub-btrfsd`
30. unmount all mounted partition
    * execute command => `umount -R /mnt`
