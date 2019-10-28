### Disk partitioning:
```
$ sgdisk -p /dev/sda
...
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            4095   1024.0 KiB  EF02  boot
   2            4096        39065599   20.0 GiB    FD00  root
...

$ blkid
/dev/md0: LABEL="root" UUID="..." TYPE="ext4"
/dev/sda2: UUID="..." UUID_SUB="..." LABEL="any:root" TYPE="linux_raid_member" PARTLABEL="root" PARTUUID="..."
/dev/sda1: PARTLABEL="boot" PARTUUID="..."

```

### Assemble RAID1 (before install):
```
mdadm --assemble /dev/md/0 --homehost=any --name=root --update=name /dev/sda2
```

### Disable 'grub.d' files:
```
chmod -x /etc/grub.d/*
chmod +x /etc/grub.d/05_debian_theme
chmod +x /etc/grub.d/10_label
```

### After replacing configs:
```
update-initramfs -u
update-grub

# optionally remove ssh keys
# /usr/bin/find /etc/ssh -type f -name "ssh_host_*" -delete
```
