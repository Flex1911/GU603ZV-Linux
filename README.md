# ASUS ROG Zephyrus G16 GU603ZV Linux setup resources

Fedora KDE is used as the basis of this guide. My laptop is a 12700H/QHD+ variant, but this should generally apply for any GU603ZV.

## General setup

Follow the [ASUS Linux guide](https://asus-linux.org/wiki/fedora-guide/) for Fedora.

## Sound

As of kernel 6.5.9, cs35l41 amp requires DSDT patch to work properly.
* Copy `etc/dracut.conf.d` contents to `/etc/dracut.conf.d`
* Regenerate initramfs: `sudo dracut -f`

Recommended, but not essential - enable full preemption to improve pipewire latency:
* Add `preempt=full` to `GRUB_CMDLINE_LINUX` in `/etc/default/grub`
* Regenerate grub config: `sudo grub2-mkconfig -o /etc/grub2-efi.cfg`

## CPU

To properly leverage CPU boost it is necessary to install thermald:
```
sudo dnf install thermald
sudo systemctl enable --now thermald.service
```

## Make TTY usable

Default console fonts are too small for a 16" 1600p screen.
* Put `FONT="latarcyrheb-sun32"` to `/etc/vconsole.conf` (or replace the existing `FONT` entry)

## Automatically lower refresh rate on battery

KDE:
* Go to System Settings -> Power Management -> Energy Saving
* In "On AC Power" enable "Run Script" and put `kscreen-doctor output.eDP-1.mode.2560x1600@240`
* In "On Battery" enable "Run Script" and put `kscreen-doctor output.eDP-1.mode.2560x1600@60`

## Hardware video decoding

For HW video acceleration to work properly you need to install [extra codecs from RPM Fusion](https://rpmfusion.org/Howto/Multimedia)
If Firefox still doesn't use HW decoding, set `media.ffmpeg.vaapi.enabled` in `about:config` to true.

## Display scaling

Fractional scaling with KDE on Wayland mostly works fine, but here are a few tips:
* For Steam, you have to force a scaling factor with an env variable: `STEAM_FORCE_DESKTOPUI_SCALING=1.5`
* Chromium-based browsers detect a scaling factor automatically. If it's blurry, try to force a scaling factor: `--force-device-scale-factor=1.5`
* Plasma panels aren't scaled on X11 by default, but can be forced with an env variable: `PLASMA_USE_QT_SCALING=1`
* Blur effects are buggy and should be disabled.

## Ethernet

Due to whatever reason r8169 driver is unstable and causes kernel hangs, even when ethernet is not used.
Realtek's r8168 fixes that:
```
sudo dnf copr enable sunwire/dkms-r8168
sudo dnf install dkms-r8168
sudo dracut -f
```

## Notes

* From my testing, gaming performance in different modes mostly scales like this: MUX X11 > PRIME Wayland > PRIME X11 > MUX Wayland