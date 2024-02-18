# KDE Neon on Chuwi MiniBook X N100

This repo contains my notes during installation and configuration of KDE Neon on Chuwi Minibook X N100 edition.

## Downloads and additional repos

### OS Image

https://files.kde.org/neon/images/user/20240215-0716/neon-user-20240215-0716.iso

Installation option: Erase disk.

### Kernel

https://launchpad.net/~tuxinvader/+archive/ubuntu/jammy-mainline

6.6.5 chosen for as latest as possible on Ubuntu 22.04 base, 6.7 series requires newer libc if I'm not mistaken.

## Findings and solutions (if any)

### SDDM login screen rotated

#### Analysis

This laptop uses native portrait screen. The setup utility and Windows rotate it before displaying anything, but as soon as control goes elsewhere, it returns to portrait mode.

#### Solution

Add
```
xrandr --output DSI-1 --rotate right
```
to /usr/share/sddm/scripts/Xsetup, then add
```
[X11]
DisplayCommand=/usr/share/sddm/scripts/Xsetup
```
to /etc/sddm.conf or uncomment if already there (I forgot how it was initially).

### SDDM touchscreen rotated

#### Analysis

Touch the screen left, the mouse pointer will be on top. Touch the screen bottom, the mouse pointer will be on right.

#### Solution

Not yet found, SDDM is pre-session environment, so setting `Coordinate Transformation Matrix` is unclear for which device.

### Plasma screen rotated

#### Analysis

Same as SDDM problem above, only the fix is not forwarded to Plasma.

#### Solution

Open System Settings->Display and Monitor, then select what the arrow points
![1708272703480](https://github.com/leledumbo/kde-neon-on-chuwi-minibook-x-n100/assets/270400/7236a669-edd7-4fbc-bbf3-c733f33e7fd2)
and click Apply. This also applies to touchscreen so touchscreen problem in SDDM doesn't apply here.

### Bluetooth not working

#### Analysis

```
$ sudo dmesg | grep Bluetooth
...
Bluetooth: hci0: failed to load intel firmware file intel/ibt-0040-1050.sfi (-2)
...
$ ls /usr/lib/firmware/intel/ibt-0040-1050.sfi
ls: cannot access '/usr/lib/firmware/intel/ibt-0040-1050.sfi': No such file or directory
```
The firmware fails to load because it doesn't exist in the first place.

#### Solution

```
$ sudo ln -s /usr/lib/firmware/intel/ibt-1040-4150.sfi /usr/lib/firmware/intel/ibt-0040-1050.sfi
$ sudo ln -s /usr/lib/firmware/intel/ibt-1040-4150.ddc /usr/lib/firmware/intel/ibt-0040-1050.ddc
```
This symlinks the latest firmware to the expected one.
