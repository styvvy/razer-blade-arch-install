# razer-blade-arch-install
notes for installing arch on the razer blade 14 2022

```
wipefs -a /dev/nvme0n1
```

cfdisk /dev/nvme0n1

yes | mkfs.fat /dev/nvme0n1p1 && yes | mkfs.ext4 -L ROOT /dev/nvme0n1p2 && yes| mkfs.ext4 -L HOME /dev/nvme0n1p3 && fatlabel /dev/nvme0n1p1 BOOT

mount /dev/nvme0n1p2 /mnt && mkdir /mnt/{boot,home} && mount /dev/nvme0n1p1 /mnt/boot && mount /dev/nvme0n1p3 /mnt/home

timedatectl set-ntp true

reflector --country Germany,Austria --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist

pacstrap /mnt base base-devel linux linux-headers linux-firmware grub efibootmgr networkmanager ntp openssh neovim neofetch zsh iwd git curl wget rsync

genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt

systemctl enable NetworkManager.service && systemctl enable sshd.service

echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && echo "KEYMAP=de" >> /etc/vconsole.conf && echo "LANG=en_US.UTF-8" >> /etc/locale.conf && locale-gen && echo "LANG=en_US.UTF-8" >> /etc/locale.conf

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub && grub-mkconfig -o /boot/grub/grub.cfg

passwd

useradd -m stywen && usermod -G wheel stywen && passwd stywen

sed -i "s/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/g" /etc/sudoers

echo "stywen ALL=NOPASSWD: /usr/bin/poweroff,/usr/bin/reboot,/usr/bin/virsh" >> /etc/sudoers

sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf

cd /opt && sudo git clone https://aur.archlinux.org/yay-git.git && sudo chown -R $USER:$USER ./yay-git && cd yay-git && makepkg -si
