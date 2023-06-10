# Chroot on termux
Guide originally by ELWAER-M

## Content

* [Good to Mention](README.md#good-to-mention)
* [Setting termux up](README.md#setting-termux-up)
* [Choosing a rootfs](README.md#choosing-a-rootfs)
* [Extracting the rootfs](README.md#extracting-the-rootfs)
* [Making a script to launch the chroot environment](README.md#making-a-script-to-launch-the-chroot-environment)
* [Troubleshooting](README.md#troubleshooting)
  - [Fixing apt under debian based ditros](README.md#fixing-apt-under-debian-based-ditros)
  - [Fixing network issues](README.md#fixing-network-issues)

## Good to Mention

* You need a [rooted](https://en.m.wikipedia.org/wiki/Rooting_(Android)) phone for that
* I've used here a samsung A20 phone running Android 11
* If any damage happens to your phone, you are all responsible for it!

## Setting termux up

You have to add root-repo and install [tsu](https://github.com/cswl/tsu) to get the root access under termux environment

```
$ apt install root-repo
$ apt update
$ apt install tsu
```
## Choosing a rootfs

the rootfs architecture have to match with your device architecture too, to know what architecture your phone have run:

```
$ uname -m
  aarch64
```

(e.g. `aarch64`, `armv7l`).
## Extracting the rootfs

Any location under `/data` should be good (because it formatted as `ext4`) so you can doing that under your termux home directory (because its under `/data` too "***`/data`***`/data/com.termux/files/home`"), or `/data/local`

e.g.

```
$ mkdir chroot #or anything else
$ sudo tar xfp /sdcard/Download/rootfs.tar.xz -C ./chroot #to keep the files permissions
```

## Making a script to launch the chroot environment

Use `vim` or any text editor you like for that:

```
$ vim run-chroot.sh
```

simple example:
```
#!/data/data/com.termux/files/usr/bin/sh

# fix /dev mount options
mount -o remount,dev,suid /data

mount --bind /dev ./chroot/dev
mount --bind /sys ./chroot/sys
mount --bind /proc ./chroot/proc
mount --bind /dev/pts ./chroot/dev/pts

# disable termux-exec
unset LD_PRELOAD

export PATH=/bin:/sbin:/usr/bin:/usr/sbin
export TERM=$TERM
export TMPDIR=/tmp

chroot ./chroot /bin/su - root
```

then change the file permissions to executable:

```
$ chmod +x run-chroot.sh
```

you have to run it as root:

```
$ sudo ./run-chroot.sh
```

## Troubleshooting
### Fixing apt under debian based ditros

```
usermod -g 3003 _apt
```

### Fixing network issues

you have to add a DNS to resolv.conf:

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

then you need to add android root groups:

```
groupadd -g 3001 aid_bt
groupadd -g 3002 aid_bt_net
groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 3005 aid_admin
```

then add those groups to the user you using:

```
usermod -a -G aid_bt,aid_bt_net,aid_inet,aid_net_raw,aid_admin root
```
