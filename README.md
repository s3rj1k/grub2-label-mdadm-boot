### Disk partitioning example:
```
parted -s -a optimal /dev/sda mklabel gpt
parted -s -a optimal /dev/sda mkpart primary 2048s 4095s set 1 bios_grub on
parted -s -a optimal /dev/sda mkpart primary ext4 4096s 50GiB set 2 raid on
```

```
$ sgdisk -p /dev/sda
...
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            4095   1024.0 KiB  EF02  boot
   2            4096       104857599   50.0 GiB    FD00  root
...

$ blkid
/dev/md0: LABEL="root" UUID="..." TYPE="ext4"
/dev/sda2: UUID="..." UUID_SUB="..." LABEL="any:root" TYPE="linux_raid_member" PARTLABEL="root" PARTUUID="..."
/dev/sda1: PARTLABEL="boot" PARTUUID="..."

```

### Create and/or Assemble RAID1 (before install):
```
mdadm --create --run --force /dev/md/0 --level=1 --raid-devices=2 --homehost=any --name=root /dev/sda2 missing
mdadm --assemble /dev/md/0 --homehost=any --name=root --update=name /dev/sda2
```

### Create ext4 partition (SSD):
```
mkfs.ext4 -FF -L root -E discard /dev/md/0
```

### Disable 'grub.d' files:
```
chmod -x /etc/grub.d/*
chmod +x /etc/grub.d/05_debian_theme
chmod +x /etc/grub.d/10_label
```

### After replacing configs:
```
update-initramfs -u -k all
update-grub

# opt: remove ssh keys
# /usr/bin/find /etc/ssh -type f -name "ssh_host_*" -delete

# opt: generate new ssh keys
# ssh-keygen -A

# opt: clean tmp files
# rm -rf /tmp/* /var/tmp/*

# opt: clean log files
# find /var/log -type f -exec truncate -s 0 {} \;
# find /var/log -type f -name *.gz -exec rm -vf {} \;

# opt: clean apt cache
# apt-get clean

# opt: remove machine id
# rm -f /var/lib/dbus/machine-id

# opt: clean dhcp leases
# rm -f /var/lib/dhcp/dhclient.leases
```

### Create image in Live media environment:
```
fsarchiver savefs /mnt/image.fsa /dev/md127
```

### Restore image in Live media environment:
```
fsarchiver restfs /mnt/image.fsa id=0,dest=/dev/md127
```

### Install grub2 boot record:
```
grub-install /dev/sda --recheck
```
