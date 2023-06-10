# Chroot on Android using Termux
Guide originally by ELWAER-M. Modified and added to by TuralTo.

## Content

* Setting chroot up
  * [Good to Mention](#good-to-mention)
  * [Setting termux up](#setting-termux-up)
  * [Choosing a rootfs](#choosing-a-rootfs)
  * [Extracting the rootfs](#extracting-the-rootfs)
  * [Making a script to launch the chroot environment](#making-a-script-to-launch-the-chroot-environment)
  * [Troubleshooting](#troubleshooting)
    * [Fixing apt under debian based ditros](#fixing-apt-under-debian-based-ditros)
    * [Fixing network issues](#fixing-network-issues)
  * [Problems that you may encounter](#problems-that-you-may-encounter)
* [Additional Stuff](#additional-stuff)
  * [Setting audio up](#setting-up-audio-pulseaudio)

## Good to Mention

* You need a [rooted](https://en.m.wikipedia.org/wiki/Rooting_(Android)) device for chroot
* The device used here is a Samsung A20 phone running Android 11 Red Velvet Cake
* If you damage your device in any way, you are all responsible for it!

## Setting termux up

You have to add `root-repo` and install [`tsu`](https://github.com/cswl/tsu) in order to get root access under the Termux environment:

```
$ apt install root-repo
$ apt update
$ apt install tsu
```
## Choosing a rootfs

The rootfs architecture has to match with your device's architecture. In order to know what architecture your device has:

```
$ uname -m
```

(e.g. `aarch64`, `armv7l`).

You can get Debian rootfs matching your architecture from [jubinson's repo](https://github.com/jubinson/debian-rootfs) for example. The most up-to-date rootfs I could find is [Ubuntu Base](http://cdimage.ubuntu.com/ubuntu-base/) at the time of writing and to my knowledge. You should be able to use any rootfs within this format.
## Extracting the rootfs

Any location under `/data` should be good (because it is formatted as `ext4`) so you can use your termux home directory (because it's under `/data` too "***`/data`***`/data/com.termux/files/home`"), or `/data/local`

e.g.

```
$ mkdir chroot #or anything else
$ sudo tar xfp /sdcard/Download/rootfs.tar.xz -C ./chroot #to keep the files permissions
```

## Making a script to launch the chroot environment

Use `vim`, `nano` or any text editor you like:

```
$ nano run-chroot.sh
```

A simple example:
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

Then change the file permissions to executable:

```
$ chmod +x run-chroot.sh
```

Run it (as root):

```
$ sudo ./run-chroot.sh
```

## Troubleshooting

### Fixing network issues

You have to add a DNS to resolv.conf (The DNS server used here is Google's):

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

Then you need to add android root groups:

```
groupadd -g 3001 aid_bt
groupadd -g 3002 aid_bt_net
groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 3005 aid_admin
```

Add those groups to the user you are using (in this case `root`):

```
usermod -a -G aid_bt,aid_bt_net,aid_inet,aid_net_raw,aid_admin root
```

### Fixing apt under debian based ditros

```
usermod -g 3003 _apt
```
You can only use this command after you've added the groups found in this [section](#fixing-network-issues).

## Problems that you may encounter

### "Could not open session" when trying to start chroot
This error is bound to the rootfs, so it may or may not be present in the rootfs you are using. The solution usually (and in my case) is commenting out or removing this line in `/etc/pam.d/su-l`:
```
session optional pam_keyinit.so force revoke
```

Other solutions that people reported working:
- Deleting `/var/log/btmp`

# Additional Stuff

## Setting up audio (pulseaudio)

### On the server side (chroot):

Before running the script make sure pulseaudio is running. If it isn't:
```
$ pulseaudio --start
```
Use your preffered text editor (in this case `nano`):
```
$ nano pashare
```
An example script to setup the server:
```
#!/bin/sh
case "$1" in
  start)
    $0 stop
    pactl load-module module-simple-protocol-tcp rate=48000 format=s16le channels=2 source=<source_name_here> record=true port=8000
    ;;
  stop)
    pactl unload-module `pactl list | grep tcp -B1 | grep M | sed 's/[^0-9]//g'`
    ;;
  *)
    echo "Usage: $0 start|stop" >&2
    ;;
esac
```
Make it executable:
```
$ chmod +x pashare
```
Run the scipt like this:
```
$ ./pashare start # replace with "stop" if you want to stop
```

The script is from [this post](https://superuser.com/questions/605445/how-to-stream-my-gnu-linux-audio-output-to-android-devices-over-wi-fi/750324#750324).

### On the client side (your phone):
* Install [Simple protocol player](https://play.google.com/store/apps/details?id=com.kaytat.simpleprotocolplayer), [PulseDroid](https://github.com/dront78/PulseDroid) for Android.
* In the app set:
  * `IP Address/Hostname/Server` to `127.0.0.1`.
  * `Port` to `8000`(in this case. set by the script).
* And connect!

