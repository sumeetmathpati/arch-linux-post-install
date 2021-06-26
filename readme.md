# Install yay

If you want to install `yay` as a aur helper, enter the following commands.

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

# Improve IO Performace

## Using the BFQ scheduler

Create the file /etc/udev/rules.d/60-ioschedulers.rules and fill it in as below:

```bash
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"
# set scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"
```

## Setting ext4 commit frequency to 60

Arch Wiki reference: https://wiki.archlinux.org/index.php/Ext4#Improving_performance

Another way to improve performance on not-so-fast SSDs is increasing the ext4 commit frequency from every 5 seconds up to 60. Just add commit=60 to the mount options for the boot partition in /etc/fstab:

```bash
/dev/sda5    /    ext4    rw,relatime,commit=60    0    1
```

# Systemd configuration

Arch Wiki reference: https://wiki.archlinux.org/index.php/Systemd/

## Taming the journal's size

Systemd's system journal's size can go out of control. There are some things you can do to keep it in control:

```bash
# journalctl --vacuum-size=100M
# journalctl --vacuum-time=2weeks
```

## Forwarding the journal to /dev/tty12

This is very simple. Just create the file /etc/systemd/journald.conf.d/fw-tty12.conf and fill it like this:

```bash
[Journal]
ForwardToConsole=yes
TTYPath=/dev/tty12
MaxLevelConsole=info
```
Then, restart the service:

```bash
# systemctl restart systemd-journald.service
```

# Improving performance on Intel + Intel graphics

Install necessary packages:

```bash
# pacman -S libva-intel-driver libvdpau-va-gl lib32-vulkan-intel vulkan-intel intel-ucode
```

Arch Wiki reference: https://wiki.archlinux.org/index.php/Microcode https://en.wikipedia.org/wiki/Intel_Microcode

The first thing you should do on Arch Linux if you have an Intel CPU is setting up intel-ucode.

The first thing to do to set up the microcode on my Intel laptop was to install the intel-ucode package:

```bash
# pacman -S intel-ucode
```

The second thing was to configure my bootloader, GRUB, with microcode early-loading. I just let grub-mkconfig handle this:

```bash
# grub-mkconfig -o /boot/grub/grub.cfg
```

Reboot and you're done.




