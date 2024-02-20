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

### Linux framebuffer rotated

#### Analysis

In a splash, the logs will be visibly written in portrait mode.

#### Solution

Modify `GRUB_CMDLINE_LINUX_DEFAULT` in /etc/default/grub to include `fbcon=rotate:1` followed by executing:
```
$ sudo update-grub
```
Reboot to activate.

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
to /etc/sddm.conf or uncomment if already there (I forgot how it was initially). Restart SDDM or reboot to activate.

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

Optionally, while you're there, change scale to 125% for more convenient UI element size.

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
Note that this is different from other guide such as [this](https://www.reddit.com/r/Chuwi/comments/15x8n4l/linux_on_chuwi_minibook_x_2023_with_intel_alder/) or [this](https://www.reddit.com/r/Chuwi/comments/1714l7g/fedora_linux_on_minibook_x_n100/) where the files are .xz compressed, in KDE Neon they're uncompressed.

### With the lid closed, fans are still spinning and battery is still draining

#### Analysis

```
$ sudo dmesg | grep S3
...
ACPI: PM: (supports S0 S3 S4 S5)
...
$ cat /sys/power/mem_sleep
[s2idle] deep
```
Intel changes the default sleep state to s2idle since Tiger Lake, making s3 the alternative. This might be a pressure from Microsoft because they want Windows to still be able to connect to the internet and update devices accordingly even when "sleeping", a.k.a. "Windows modern standby".

#### Solution

Modify `GRUB_CMDLINE_LINUX_DEFAULT` in /etc/default/grub to include `mem_sleep_default=deep` followed by executing:
```
$ sudo update-grub
```
Reboot to activate.

### Flipping the display doesn't disable keyboard

#### Analysis

Open a Konsole, type:
```
$ sudo dmesg -w
```
then press Ctrl+Shift+), type:
```
$ sudo journalctl -f
```
basically opening 2 monitoring logs. Now flip the display to see... nothing. This means whatever sensor is used isn't recognized by any part of the Linux stack. To be sure, iio-sensor-proxy is preinstalled in KDE Neon.

#### Solution

Not yet found, probably in some newer kernel version in the future. I've yet to report it to https://bugzilla.kernel.org/, gotta gather information first should be available on Windows since it works there.

## Tips and tricks

### Regulate power usage and save battery

1. Install TLP: `$ sudo apt install tlp`, should be enable automatically but if not: `$ sudo systemctl enable --now tlp`, I don't configure anything but feel free to do so if you don't use something (e.g. bluetooth) and want to save more battery.
2. Install [autocpu-freq](https://github.com/AdnanHodzic/auto-cpufreq)https://github.com/AdnanHodzic/auto-cpufreq: Just follow [the guide](https://github.com/AdnanHodzic/auto-cpufreq?tab=readme-ov-file#installing-auto-cpufreq), pick the `auto-cpufreq-installer` route and disable Intel P-State.
3. Reboot for all to work

### Auto switch audio codec on conference call apps (e.g.: Google Meet, Zoom, Slack's Huddle)

By default, PulseAudio and PipeWire server are both installed AND running because there's a partial functionality only PipeWire has.
Likewise, this profile auto switch feature is only implemented by PipeWire (as of this time of writing).

Optionally, if you need AptX and AAC codecs, execute the following **before** executing below commands:
```
$ sudo add-apt-repository ppa:aglasgall/pipewire-extra-bt-codecs
$ sudo apt update
```
Execute:
```
$ sudo apt install pipewire-media-session- pulseaudio-module-bluetooth- wireplumber pipewire-audio-client-libraries libldacbt-{abr,enc}2 libspa-0.2-bluetooth 
$ sudo cp /usr/share/doc/pipewire/examples/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d/
$ systemctl --user --now enable wireplumber
```
to uninstall pipewire-media-session and PulseAudio's bluetooth module (so PipeWire can take over) while installing wireplumber, its ALSA plugin and bluetooth libraries at the same time, copying ALSA configuration as well as enabling wireplumber session manager.
