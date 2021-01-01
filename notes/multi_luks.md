# Use LUKS to encrypt ubuntu partitions
TODO intro

## rationale
TODO

## basics
TODO

## references
TODO

## tools
TODO

## instructions
TODO: backup warning

First you want to partition your drive

Assuming you use uefi(todo:link to uefi) you want to have:
 - one ESP patition (todo: link to esp) (around 200/300 mb)
 - one partition **for each** ubuntu installation that will be mounted as */boot* (around 500mb)
 - one partition **for each** ubuntu installation that we will encrypt using luks that will contain our */*, *swap* (and */home* if you like)
![starting point](01_starting_point.png =250x)



[encrypting disks on ubuntu]: https://medium.com/@chrishantha/encrypting-disks-on-ubuntu-19-04-b50bfc65182a
[arch lvm on luks]: https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS
[arch wipe disk]: todo
[boot process]: https://linuxhint.com/understanding_boot_process_bios_uefi/
