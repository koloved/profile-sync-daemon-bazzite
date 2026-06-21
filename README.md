# Profile-sync-daemon
Profile-sync-daemon (psd) is a tiny pseudo-daemon designed to manage your browser's profile in tmpfs and to periodically sync it back to your physical disc (HDD/SSD). This is accomplished via a symlinking step and an innovative use of rsync to maintain back-up and synchronization between the two. One of the major design goals of psd is a completely transparent user experience.

## Good Advice
Always backup your browser profile(s) before using psd for the first time.

## Users of eCryptFS
User of eCryptFS are encouraged not to use psd unless willing to help troubleshoot suspected browser corruption. See [#158](https://github.com/koloved/profile-sync-daemon-bazzite/issues/158).

## Supported Browsers
* Chromium
* Conkeror
* Epiphany
* Firefox (version 147 and above use freedesktop.org XDG base dir specs others use legacy `~/.mozilla`)
* Firefox flatpak (run `flatpak override --user org.mozilla.firefox --filesystem=/run/user/$UID/psd` once to give required access)
* Firefox-trunk (this is an Ubuntu-only browser: http://www.webupd8.org/2011/05/install-firefox-nightly-from-ubuntu-ppa.html)
* Google Chrome (stable, beta, and dev)
* Heftig's version of Aurora (this is an Arch Linux-only browser: https://bbs.archlinux.org/viewtopic.php?id=117157)
* Icecat (GNU version of Firefox)
* Iceweasel (Debian version of Firefox)
* Inox (https://bbs.archlinux.org/viewtopic.php?id=198763)
* Luakit
* Midori
* Opera, Opera-Beta, Opera-Developer, and Opera-Legacy
* Otter-browser
* Palemoon
* QupZilla
* Qutebrowser
* Rekonq
* Seamonkey
* Surf (http://surf.suckless.org/)
* Vivaldi-browser and Vivaldi-browser-snapshot

## Documentation
Consult the man page or the wiki page: https://wiki.archlinux.org/index.php/Profile-sync-daemon

## Installation from Source
To build from source, see the included INSTALL text document.

## Installation on Atomic/Immutable Systems (Bazzite, Fedora Silverblue, etc.)
On atomic systems where `/usr` is read-only, install to your home directory:

```bash
git clone https://github.com/koloved/profile-sync-daemon-bazzite
cd profile-sync-daemon
make
make user-install
systemctl --user daemon-reload
systemctl --user enable --now psd.service
```

This installs everything under `~/.local/` and `~/.config/systemd/user/`.

### Verifying it works

```bash
# 1. Check service status (should show "active (exited)")
systemctl --user status psd.service

# 2. Check that overlay mounts are created for your browsers
mount | grep fuse-overlayfs | grep psd

# 3. Check if your browser profile is now a symlink to tmpfs
# (for Firefox flatpak as an example)
ls -la ~/.var/app/org.mozilla.firefox/config/mozilla/firefox/*.default*

# 4. See how much RAM is being used by the overlay mounts
systemctl --user status psd.service | grep Memory

# 5. Check psd's own runtime directory
ls -la /run/user/$(id -u)/psd/
```

If step 1 shows `active (exited)` and step 2 shows your browsers listed — it's working. Your browser profiles are running in RAM. Write `BROWSERS=(firefox-flatpak)` in `~/.config/psd/psd.conf` if you only want to manage specific browsers instead of all detected ones.

## Installation from Distro Packages
### Officially Packaged
* Arch: in the extra [repo](https://archlinux.org/packages/extra/any/profile-sync-daemon/).
* Debian: in the official [repos](https://packages.debian.org/unstable/profile-sync-daemon).
* Exherbo: in the official [repos](http://git.exherbo.org/summer/packages/net-www/profile-sync-daemon).
* Gentoo: in the official [repos](http://packages.gentoo.org/package/www-misc/profile-sync-daemon).
* NixOS: in the official [repo](https://search.nixos.org/packages?channel=22.11&from=0&size=50&sort=relevance&type=packages&query=profile-sync).
* Ubuntu: in Universe [repo](https://packages.ubuntu.com/search?keywords=profile-sync-daemon).

### User Packaged
* Slackware: on [slackbuilds](http://slackbuilds.org/apps/profile-sync-daemon/).

### Other Distros
If you are interested in packaging psd for your favorite distro, please contact me.

## logcheck
Using logcheck? Here are some ways to filter out log lines:
```regexp
^\w{3} [ :0-9]{11} [._[:alnum:]-]+ profile-sync-daemon\[[0-9]+]\]: .*(google-chrome|firefox) (re|un)sync successful
^\w{3} [ :0-9]{11} [._[:alnum:]-]+ profile-sync-daemon\[[0-9]+\]: psd startup check successful$
^\w{3} [ :0-9]{11} [._[:alnum:]-]+ systemd\[[0-9]+\]: Started Timer for profile-sync-daemon
^\w{3} [ :0-9]{11} [._[:alnum:]-]+ systemd\[[0-9]+\]: psd(-resync)?\.service: Consumed [0-9\.]+s CPU time\.$
^\w{3} [ :0-9]{11} [._[:alnum:]-]+ systemd\[[0-9]+\]: psd(-resync)\.timer: Succeeded\.$
```
