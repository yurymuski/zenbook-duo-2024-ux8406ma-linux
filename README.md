# zenbook-duo-2024-ux8406ma-linux

Features:
* brightness sync (any)
* battery limiter (any)
* touch/pen panels mapping (GNOME-specific, requires GNOME 46 or a backported Mutter patch)
* automatic bottom screen on/off (GNOME-specific)
* automatic rotation (GNOME-specific)

## panel mapping

`duo set-tablet-mapping` will set necessary dconf settings, but for them to work you need a Mutter with a patch from https://gitlab.gnome.org/GNOME/mutter/-/merge_requests/3556 and libwacom with this patch https://github.com/linuxwacom/libwacom/pull/640 . Both are merged upstream, so you can just wait.

## bottom screen toggle on GNOME

Make sure gnome-monitor-config, usbutils and inotify-tools are installed, the script relies on the `gnome-monitor-config`, `lsusb` and `inotifywait` commands from them.

Before the next steps, you may need or want to change the scaling settings or change the config at the top of `duo` based on the version of the duo that you have (1080p vs 3k display models)

For automatic screen management run `duo watch-displays` somewhere at the start of your GNOME session.

For manual screen management there are `duo top`, `duo bottom`, `duo both` and `duo toggle` (toggles between top and both) commands.

In addition there's also `duo toggle-bottom-touch` to toggle touch for the bottom screen, so you can draw with a pen while resting your hand on the screen.

## automatic rotation

Make sure iio-sensor-proxy is installed, the script relies on `monitor-sensor` command from it. Once it's installed and you followed the steps above for dualscreen setup just run `duo watch-rotation` somewhere at the start of your GNOME session.

## brightness sync

Brightness control requires root permissions. I prefer to have sudo with a password by default, so I use a hack to have a NOPASSWD sudo for /usr/bin/env which allows to execute any command. Line in /etc/sudoers looks like `%wheel  ALL=(ALL:ALL)    NOPASSWD: /usr/bin/env`. On NixOS the relevant part of the config is this:

```
  security.sudo = {
    enable = true;
    extraRules = [{
      commands = [
        {
          command = "/usr/bin/env";
          options = [ "NOPASSWD" ];
        }
      ];
      groups = [ "wheel" ];
    }];
  };
```

Once the sudo setup is done you can either run `duo sync-backlight` to sync it once (you may want to bind it to some hotkey) or you can run `duo watch-backlight` at login and it will keep syncing your brightness from the top display to the bottom one.

For most linux distros there is an included systemd service file: `brightness-sync.service` that just needs `/path/to/duo` changed before moving it to `/etc/systemd/system` to enable brightness sync in the background.

## battery limiter

Requires same sudo setup as for the brightness sync. Most likely you want to run `duo bat-limit` or `duo bat-limit 75` (where 75 is your desired threshold percentage, 80 is used if omited) once at the start of your desktop session.

## keyboard backlight control

Requires python3 and pyusb installed. `duo set-kb-backlight <0|1|2|3>` configures keyboard backlight, with 0 meaning off and 3 meaning max brightness.

## Notes concerning usage on Fedora 40

The steps described above work on Fedora 40 with the following specific changes:
Prerequisities:
`sudo dnf install lm_sensors gnome-monitor-config inotofy-tools`
Libwacom files elan-425a.tablet and elan-425b.tablet should be copied to /usr/share/libwacom
For brightness sync to work properly, line 10 of the duo.sh should be modified to `backlight=card1-eDP-2-backlight`


---
# Prerequisities ubuntu 2210
```sh
# ref: https://fostips.com/multiple-monitors-turn-off-any-ubuntu/
sudo apt install git libcairo2-dev cmake meson ninja-build libglib2.0-dev
git clone https://github.com/jadahl/gnome-monitor-config.git
cd gnome-monitor-config
meson build
cd build
meson compile
ln -s /path/build/scr/gnome-monitor-config /usr/local/bin/gnome-monitor-config


sudo apt-get install inotify-tools python3-pip
sudo pip install pyusb --break-system-packages
sudo ln -s /path/to/zenbook-duo-2024-ux8406ma-linux/duo /usr/local/bin/duo
```
---
# fix rotation

```sh
gnome-session-properties # add `duo watch-rotation`
```

---

# fix touchpad mapping
```sh
# ref: https://area-51.blog/2023/07/01/linux-on-the-asus-zenbook-duo-pro-14/

`xrandr` # get display names eDP-1 and eDP-1

`xinput` # get Touchpad ids

# map zenbook duo touchpads
`xinput map-to-output "ELAN9008:00 04F3:4259" eDP-1`
`xinput map-to-output "ELAN9009:00 04F3:42EC" eDP-2`

# add to .bashrc
```

---
# fix Duo Brightness Sync
```sh

sudo ln -s /path/to/zenbook-duo-2024-ux8406ma-linux/brightness-sync.service /etc/systemd/system/brightness-sync.service
systemctl daemon-reload
systemctl enable brightness-sync --now
systemctl status brightness-sync
```

---
# useful manual commands
```sh
duo toggle # disable 2nd monitor
duo set-kb-backlight 3 # max brightness keyboard (connected only)
duo set-kb-backlight 0 # disable brightness keyboard (connected only)

duo bat-limit 95 # battery charge limit

```
