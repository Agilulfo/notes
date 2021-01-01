# Ubuntu dual boot with encrypted partitions

The following guide is a work in progress. It probably does contain mistakes!
If you decide to fo follow it, do it at your own risk.

Encrypting partitions in this way is not a standard flow offered by linux distributions.
I've decided to write some notes so that next time I'll not have to restart from scratch.

## Rationale/Goal
### the problem
I have a laptop provided by the company I work for. Since it's a quite powerfull machine and I do not like the idea of bringing around 2 laptops I've decided to use the same machine both for *personal* and *work* use.

Using one system to perform both things is not a so great for at least two reasons.
1. *Stability*: you do not want something you need for personal use to compromise the stability of your working environment making you unable to work. 
2. *Security*: you do not want a vulnerability on something you need for personal use to compromise sensitive data of your working environment (and viceversa) 

An easier way to address the 2 points would be to use 2 separate machines but still you should nowadays use encryption anyway. 

### the plan
The solution I've decided to put in place is to have 2 separate os (ubuntu linux in my case) instances on the same machine (a.k.a. dual boot) to solve the first point.
You also want to encrypt both partitions so that one installation cannot temper with data in the other. This should address the 2nd concern.

## basics
You might want to refresh you knowledge on how [boot process] works on linux

## references
The process described in this document is a mix of the following guides/articles

- [encrypting disks on ubuntu]
- [Arch wiki on how to encrypt using LVM on LUKS][arch lvm on luks]

## tools
TODO

LVM and LUKS

commands:
- lsbkl
- cryptsetup
- pvcreate
- vgcreate
- lvcreate



## warning
TODO: backup warning
TODO: disk wiping warning
TODO: disclaimer on uefi/gpt
TODO: disclaimer on tested only on ubunt 20.04
TODO: disclaimer on the fact that boot partitions are not encrypted

## instructions
First you want to partition your drive


### [Step 1] partitioning
Assuming you use uefi(todo:link to uefi) you want to have:
 - one ESP patition (todo: link to esp) (around 200/300 mb)
 - one partition **for each** ubuntu installation that will be mounted as */boot* (around 500mb), this partition will contain the bootloader (grub) and the kernels. (Such partition will not be encrypted) 
 - one partition **for each** ubuntu installation thzat we will encrypt using luks that will contain our */*, *swap* (and */home* if you like)

![starting point](01_starting_point.png)

you can use tools like *gparted* or *partitionmanager* (on kde) to edit your partition table (gpt on uefi) to reach the result above.

### [Step 2] encryption setup

#### setup luks

Our next step is to encrypt the partitions that will contain the data we would like to protect.

``` bash
sudo cryptsetup luksFormat --hash=sha512 --key-size=512 /dev/nvme0n1p4
sudo cryptsetup luksFormat --hash=sha512 --key-size=512 /dev/nvme0n1p5
sudo cryptsetup open --type=luks /dev/nvme0n1p4 work
sudo cryptsetup open --type=luks /dev/nvme0n1p5 personal
```

#### setup lvm

create a pysical volume and volume gruop and then create logical volumes as you please (in my case one for root and the other for the swap partition)
``` bash
sudo pvcreate /dev/mapper/work 
sudo vgcreate wvg /dev/mapper/work 
sudo lvcreate -L 8G wvg -n swap
sudo lvcreate -l 100%FREE wvg -n root
```
TODO: I was not sure if the swap could be shared safely so I've opted to inclue the swap as a logical volume

repeat the same for other partitions
``` bash
sudo pvcreate /dev/mapper/personal
sudo vgcreate pvg /dev/mapper/personal
sudo lvcreate -L 8G pvg -n swap
sudo lvcreate -l 100%FREE pvg -n root
```
tip: if you make mistakes, commands such as lvrename are available 
tip: use regularly lsblk to check if you are getting the result you would 

### [Step 3] Perform the installation

Now is time to perform the installation of ubuntu. chose to manually select the partitions and set accordingly the partitions to use as */boot*, *swap* and */* 

### [Step 4] configure kernel to load the partitions

Use lsblk to get the uuid of the parition 
for instance in my case the uuid I have to take note is `48b7b2e4-4c03-4339-98f8-672e897ea5fc`

``` bash
kubuntu@kubuntu:~$ sudo  lsblk -o name,uuid,mountpoint
NAME           UUID                                   MOUNTPOINT
loop0                                                 /rofs
sda            2020-04-23-07-59-42-00                 
├─sda1         2020-04-23-07-59-42-00                 /cdrom
├─sda2         1AC3-20ED                              
└─sda3         c278ed1f-6ac6-41a1-9074-7db516723945   /var/crash
nvme0n1                                               
├─nvme0n1p1    6BFB-F958                              
├─nvme0n1p2    1b0a7b5a-d724-4727-9ebc-06cd099ac37d   /mnt/boot
├─nvme0n1p3    5702f183-eb74-49ff-9e8b-79515e103e1b   
├─nvme0n1p4    48b7b2e4-4c03-4339-98f8-672e897ea5fc   
│ └─work       DpOYw5-Pivu-edGh-Ngqh-UTSN-jXL7-uXlP1H 
│   ├─wvg-swap 9cdc59eb-9e97-4e7b-a1fa-56eced8f83f1   
│   └─wvg-root 22c6b822-f9be-40e0-84b7-def2548466c0   /mnt
└─nvme0n1p5    614c65f4-821a-47ba-853e-c49e1921631a   
kubuntu@kubuntu:~$ 
```

Now we need to chroot(TODO: link) in the installed ubuntu and configure crypttab 
``` bash
sudo mount /dev/mapper/wvg-root /mnt
sudo mount /dev/nvme0n1p2 /mnt/boot
sudo mount --bind /dev /mnt/dev
sudo chroot /mnt
mount -t proc proc /proc
mount -t sysfs sys /sys
mount -t devpts devpts /dev/pts
sudo nano /etc/crypttab
```
TODO: not sure if all the mount commants are necessary.

the content of your */etc/cypttab* should look like this:

```
work UUID=48b7b2e4-4c03-4339-98f8-672e897ea5fc none luks,discard
```

last but not least

``` bash
update-initramfs -k all -c
```

tip: this should scream some warnings if there is a mistake with crypttab. 
In such case check again and fix the content of /etc/crypttab.

Once finished restart. 
You sould be promted for the password of your volume and once you type it you should be able to start your pc.

If this does not happen you have likelly messedup something in the /etc/crypttab. 
You should be able to fix it by starting again from ubuntu live and checking, chrooting and repeating the last steps. 

then you can repeat from the installation step for the remaining instances of ubuntu you want to install.

## GRUB issues:
TODO: [link][grub does not detect other partition]

[encrypting disks on ubuntu]: https://medium.com/@chrishantha/encrypting-disks-on-ubuntu-19-04-b50bfc65182a
[arch lvm on luks]: https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS
[arch wipe disk]: todo
[boot process]: https://linuxhint.com/understanding_boot_process_bios_uefi/
[cypttab]: todo
[grub does not detect other partition]: https://ubuntuforums.org/showthread.php?t=2455975 
