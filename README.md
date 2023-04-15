# razer-blade-arch-install
notes for installing arch on the razer blade 14 2022

### wipe the nvme

```
wipefs -a /dev/nvme0n1
```
### partition the disk
Partition 1: 256 MB /boot
Partition 2: 30  GB /root
Partition 3: 1.7 TB /home

```
cfdisk /dev/nvme0n1
```

### create the right filesystem
In this case I choose ext4 for /root /home and fat for the /boot
```
yes | mkfs.fat /dev/nvme0n1p1 && yes | mkfs.ext4 -L ROOT /dev/nvme0n1p2 && yes| mkfs.ext4 -L HOME /dev/nvme0n1p3 && fatlabel /dev/nvme0n1p1 BOOT
```

### mount the partitions and create the right folders to nmount the partitions
```
mount /dev/nvme0n1p2 /mnt && mkdir /mnt/{boot,home} && mount /dev/nvme0n1p1 /mnt/boot && mount /dev/nvme0n1p3 /mnt/home
```

### enable ntp
```
timedatectl set-ntp true
```

### configure /etc/pacman.conf to do parallel downloads
### set the fastest mirrors
```
reflector --country Germany,Austria --age 12 --protocol http --sort rate --save /etc/pacman.d/mirrorlist
```

### install the base system
```
pacstrap /mnt base base-devel linux linux-headers linux-firmware grub efibootmgr networkmanager ntp openssh neovim neofetch
```

### generate the partition table
```
genfstab -U /mnt >> /mnt/etc/fstab
```

### switch into the newly created OS
```
arch-chroot /mnt
```

#### enable NetworkManager and SSH

```
systemctl enable NetworkManager.service && systemctl enable sshd.service
```
### configure the display language and the keyboard layout
```
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && echo "KEYMAP=de" >> /etc/vconsole.conf && echo "LANG=en_US.UTF-8" >> /etc/locale.conf && locale-gen && echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

### configure the bootloader grub
``` 
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub && grub-mkconfig -o /boot/grub/grub.cfg
```
### set the root password
```
passwd
```

### create a user and make it part of the group "wheel"
```
useradd -m stywen && usermod -G wheel stywen && passwd stywen
```

### MISC
```
sed -i "s/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/g" /etc/sudoers && echo "stywen ALL=NOPASSWD: ALL" >> /etc/sudoers && sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf
```
### exit and reboot
```
exit
umount -R /mnt
reboot
```

### install yay
```
cd /opt && sudo git clone https://aur.archlinux.org/yay-git.git && sudo chown -R $USER:$USER ./yay-git && cd yay-git && makepkg -si
```
