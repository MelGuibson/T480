GPT : 
```
gdisk /dev/nvme0n1
    o
    n
    1
    ... (first sector of boot partition)
    +512MB (last sector of boot partition)
    EF00 (EFI)
    n
    2
    ... (use all of disk or partition further)
    8E00 (LVM)
    w
```

Setting UP LVM : 

```
cryptsetup luksFormat /dev/nvme0n1p2
cryptsetup open --type luks /dev/nvme0n1p2 foo
pvcreate /dev/mapper/foo
vgcreate foo_group /dev/mapper/foo
lvcreate -L32G foo_group -n swap
lvcreate -L40G foo_group -n root
lvcreate -l 100%FREE foo_group -n home
```

Format : 

```
mkfs.fat /dev/nvme0n1p1
mkfs.ext4 /dev/mapper/foo_group-root
mkfs.ext4 /dev/mapper/foo_group-home
mkswap /dev/mapper/foo_group-swap
```

Mount : 

```
mount /dev/mapper/foo_group-root /mnt
mkdir /mnt/home
mount /dev/mapper/foo_group-home /mnt/home
swapon /dev/mapper/foo_group-swap
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

Fstab generation / archrooting into mnt : 

```
pacstrap -i /mnt base base-devel
genfstab -U /mnt > /mnt/etc/fstab
arch-chroot /mnt /bin/bash
```

Setup : 

```
pacman-key --init
pacman-key --populate archlinux
pacman -Syu vim sudo intel-ucode linux linux-firmware mkinitcpio lvm2 dhcpcd netctl wpa_supplicant dialog
vim /etc/locale.gen # uncomment fr_FR.UTF-8 UTF-8
locale-gen
ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc --utc
echo hostname > /etc/hostname
passwd # set root password
```

Bootloader : 

```
vim /etc/mkinitcpio.conf
HOOKS=(base udev autodetect keyboard keymap modconf block encrypt lvm2 filesystems fsck)
mkinitcpio -p linux
bootctl --path=/boot install
vim /boot/loader/loader.conf
# default arch
# editor 0
blkid # note UUID of crypto_LUKS drive (primary)
```

