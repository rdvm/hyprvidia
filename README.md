# hyprvidia
A repo with some resources for setting up Hyprland WM on a system with Nvidia

## Arch minimal install with archinstaller

* Grab the Arch ISO from [teh internet](https://geo.mirror.pkgbuild.com/iso/2023.06.01/) and load it onto a bootable USB
* Connect to WiFi using `iwd` if you don' that have a wired connection (#iwd)
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
git clone https://github.com/rdvm/hyprvida
```

* The `hyprdesk_pkglist.txt` file contains a bunch of useful and necessary packages that can all be installed like this:

```bash
sudo pacman -S --needed - < hyprvidia/hyprdesk_pkglist.txt
```

* There is also a `hyprdesk_pkglist_aur.txt` that has AUR packages, but I've found that piping a text file to `yay` doesn't really work too well so you may want to just `cat` out the file to read the packages and type them into a `yay -S ...` command


### iwd

* Run `iwctl`
* Inside iwd you can run `device list` to see available networking devices, assuming your WiFi interface is `wlan0`
  * `station wlan0 connect <SSID>`
  * Enter your WiFi password when prompted
* `exit`
