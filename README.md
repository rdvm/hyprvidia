# hyprvidia
A repo with some resources for setting up Hyprland WM on a system with Nvidia

## Arch minimal install with archinstaller

* Grab the Arch ISO from [teh internet](https://geo.mirror.pkgbuild.com/iso/2023.06.01/) and load it onto a bootable USB
* Connect to WiFi using `iwd` if you don' that have a wired connection ([see iwd section](#iwd)
* Run `archinstall`
* Refer to the [official documentation](https://python-archinstall.readthedocs.io/en/latest/installing/guided.html#description-individual-steps) for recommendations
* Here are my recommendations:
  * *Archinstall language*: English
  * *Keyboard layout*, *Locale language*, and * `Locale encoding` should automatically be set correct for US users as well
  * *Mirror region*: leave blank for `auto detect best mirror`
  * *Drives*: you're on your own
  * *Disk encryption*: also your call
  * *Bootloader*: doesn't really matter but systemd-boot is the new default if enabled
  * *Swap*: Default is `zswap` an what I went with [zswap on ArchWiki](https://wiki.archlinux.org/title/zswap)
  * *Hostname*: up to you, but I would recommend putting something in there just to make it clear
  * *Root password*: leave blank and create a `sudo` further on
  * *User account*: create your user account and choose the option to have it be a super user
  * *Profile*: select the `minimal` option to get a bunch of basic software you're going to need regardless
  * *Audio*: deafult is `pipewire` and is what I'd recommend
  * *Kernels*: stick with standard `linux` kernel unless you have reason to do otherwise
  * *Additional packages*: up to you, but if you use the `pkglist.txt` from this repo this step may be superfluous
  * *Network configuration*: go with good ol' NetworkManager
  * *Timezone*: select your timezone (e.g. `America/Chicago`)
  * *Automatic time sync (NTP)*: leave as `True`
  * *Optional repositories*: leave blank
* Proceed to the install phase and follow any prompts to reboot etc.
* Log in with your newly created user account
* Install `yay` (or another AUR helper, but I use `yay`)

```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd
rm -rf yay #this is just to remove the yay repo now that it's installed
```

* Install `hyprland-nvidia-git` (or `hyprland-nvidia` if you want to stay further from the bleeding edge...but then why Hyprland at all?)
  * `yay -S hyprland-nvidia-git`
* Clone this repo to get a couple of package lists that can be used to bulk install

```bash
git clone https://github.com/rdvm/hyprvida.git
```

* The `hyprdesk_pkglist.txt` file contains a bunch of useful and necessary packages that can all be installed like this:

```bash
sudo pacman -S --needed - < hyprvidia/hyprdesk_pkglist.txt
```

* There is also a `hyprdesk_pkglist_aur.txt` that has AUR packages, but I've found that piping a text file to `yay` doesn't really work too well so you may want to just `cat` out the file to read the packages and type them into a `yay -S ...` command

### Nvidia shenanigans

You can refer to the [Nvidia section of the Hyprland Wiki](https://wiki.hyprland.org/Nvidia/), but here are some consolidated notes from my experience.

#### Bootloader stuff

* Install `nvidia-dkms` (*if you installed the whole `pkglist.txt` earler you already have it*)

If you're using grub as your bootloader skip systemd-boot section and go to [Grub](#grub)

##### systemd-boot

* Add `nvidia_drm.modeset=1` to the end of the file `/boot/loader/entries/arch.conf` (*For me the file was empty or maybe even non-existent so I had to create it. That line is the only thing in mine as of today*)

[/boot/loader/entries/arch.conf]
```
nvidia_drm.modeset=1
```

##### Grub

* Add `nvidia_drm.modeset=1` to the end of `GRUB_CMDLINE_LINUX_DEFAULT=` in the file `/etc/default/grub`
* Run `grub-mkconfig -o /boot/grub/grub.cfg`

### Mkinitcpio

* Add `nvidia nvidia_modeset nvidia_uvm nvidia_drm` to the `MODULES` in `/etc/mkinitcpio.conf`

[/etc/mkinitcpio.conf]
```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
BINARIES=()
FILES=()
HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block filesystems fsck)
```

* Run the following command, making sure you have `linux-headers` installed first (*if you installed the whole `pkglist.txt` earlier you already have it*)

```
mkinitcpio --config /etc/mkinitcpio.conf --generate /boot/initramfs-custom.img
```

* Add `options nvidia-drm modeset=1` to `/etc/modprobe.d/nvidia.conf` (*You may need to create this file. I did, and this line is the only thing in it.

[/etc/modprobe.d/nvidia.conf]
```
options nvidia-drm modeset=1
```

* Install `qt5-wayland`, `qt5ct` and `libva` (*if you installed the whole `pkglist.txt` earlier you already have these*)

### Hyprland config environment variables

There are some variables you'll want to include in your `hyprland.conf`, but you may not have one yet if you haven't written your own/imported any personal dotfiles. The default `hyprland.conf` won't show up until the first boot after the `hyprland` package has been installed. At any rate, you'll want/need to have these in your configuration file at whatever point you have one:

```conf
env = LIBVA_DRIVER_NAME,nvidia
env = XDG_SESSION_TYPE,wayland
env = GBM_BACKEND,nvidia-drm
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
env = WLR_NO_HARDWARE_CURSORS,1
```

### If you get a "loop" on the login screen

I think this might have as much to do with the specific CPU as it does with anything having to do with Nvidia, but I was getting a weird behavior initially. On the SDDM login screen I'd enter my password, it would seem to be accepted, the screen would go black for a second or two, and then return me to the login screen. I also could not switch to another `TTY` to do anything, so I had to boot from the installation USB again, chroot into my install, and then added this `block-i915.conf` file to `/etc/modprobe.d`

[/etc/modprobe.d/block-i915.conf]
```
blacklist i915

install i915 /usr/bin/false
install intel_agp /usr/bin/false
```

### iwd

* Run `iwctl`
* Inside iwd you can run `device list` to see available networking devices, assuming your WiFi interface is `wlan0`
  * `station wlan0 connect <SSID>`
  * Enter your WiFi password when prompted
* `exit`
