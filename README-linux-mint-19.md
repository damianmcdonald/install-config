# Linux Mint 19 64bit XFCE Host Installation

## Pre-install tasks

* Download ISO FILE - *linuxmint-19-xfce-64bit-v2.iso*
* Create USB boot drive with [Rufus](https://rufus.ie/en_IE.html)
* Boot into Linux Mint Live from USB
* Shred the hard drives
```bash
sudo fdisk -l
sudo shred -v -n1 -z /dev/sdX
```

## Post-install tasks
https://sites.google.com/site/easylinuxtipsproject/mint-cinnamon-first
https://sites.google.com/site/easylinuxtipsproject/3
https://sites.google.com/site/easylinuxtipsproject/ssd

### Timeshift

* Go to Update Manager and configure Timeshift
* RSYNC -> Monthly -> Keep 2
* Create an initial system snapshot

### System startup apps
* Go to Settings -> Sessions and Startup -> Application Autostart
* Prevent unwanted aps from starting

### System updates and software install
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt dist-upgrade
sudo apt-get install firejail git rclone gpodder youtube-dl imagemagick puddletag nodejs npm python3-pip
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt-get update && sudo apt-get install youtube-dlg
```

### Install AWS tools
```bash
pip3 install awscli --no-cache --upgrade --user
pip3 install localstack --no-cache --user
pip3 install amazon_kclpy --no-cache --user
pip3 install awscli-local --no-cache --user
```

### Add proprietary drivers
* Go to System -> Driver Manager
* Select proprietary drivers

### Turn off visual effects in Xfce
* Go to Menu button -> Settings -> Desktop Settings
* Window Manager: from Xfwm4 + Compositing to Xfwm4

### Tame the inode cache
Reduce the use of disk swapping.

```bash
sudo cp -v /etc/sysctl.conf /etc/sysctl.conf.backup
sudo nano /etc/sysctl.conf
```
Scroll to the bottom of that text file and add the parameters below to override the default.

 ```bash
# Improve cache management
vm.vfs_cache_pressure=50

# Sharply reduce the inclination to swap
vm.swappiness=1
```
### SSD discard and noatime
With "noatime" in /etc/fstab, you disable the write action "access time stamp", that the operating system puts on a file whenever it's being read by the operating system.

```bash
sudo cp -v /etc/fstab /etc/fstab.backup
sudo nano /etc/fstab
```
Here's an example of an adapted line, in which you can see the exact spot where the discard and noatime switch has to be put:

`UUID=xxxxxxx / ext4 discard,noatime,errors=remount-ro 0 1`

Note: there should be no space after the comma after discard or noatime entries.

### Configure TRIM on SSD
Check the SSD TRIM status:

`sudo hdparm -I /dev/sda | grep TRIM`

The SSD supports TRIM when the message below is shown:

*Data Set Management TRIM supported*

Schedule a daily trimming as follows.

```bash
sudo mkdir -v /etc/systemd/system/fstrim.timer.d
sudo touch /etc/systemd/system/fstrim.timer.d/override.conf
```
Add the following text to the file.

```bash
[Timer]
OnCalendar=
OnCalendar=daily
```
After a reboot, confirm the setting.

`systemctl cat fstrim.timer`

The output should look as shown below.

```bash
# /lib/systemd/system/fstrim.timer
[Unit]
Description=Discard unused blocks once a week
Documentation=man:fstrim

[Timer]
OnCalendar=weekly
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target

# /etc/systemd/system/fstrim.timer.d/override.conf
[Timer]
OnCalendar=
OnCalendar=daily
```

### Disable suspend to RAM
```bash
sudo touch /etc/polkit-1/localauthority/90-mandatory.d/disable-suspend.pkla
sudo nano /etc/polkit-1/localauthority/90-mandatory.d/disable-suspend.pkla
```
Add the entries to the file as shown below.

```bash
[Disable suspend (upower)]
Identity=unix-user:*
Action=org.freedesktop.upower.suspend
ResultActive=no
ResultInactive=no
ResultAny=no

[Disable suspend (logind)]
Identity=unix-user:*
Action=org.freedesktop.login1.suspend
ResultActive=no

[Disable suspend for all sessions (logind)]
Identity=unix-user:*
Action=org.freedesktop.login1.suspend-multiple-sessions
ResultActive=no
```

### Disable hibernate
```bash
sudo touch /etc/polkit-1/localauthority/90-mandatory.d/disable-hibernate.pkla
sudo nano /etc/polkit-1/localauthority/90-mandatory.d/disable-hibernate.pkla
```
Add the entries to the file as shown below.

```bash
[Disable hibernate (upower)]
Identity=unix-user:*
Action=org.freedesktop.upower.hibernate
ResultActive=no
ResultInactive=no
ResultAny=no

[Disable hibernate (logind)]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate
ResultActive=no

[Disable hibernate for all sessions (logind)]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate-multiple-sessions
ResultActive=no
```

### Uninstall unecessary software
```bash
sudo apt-get remove mono-runtime-common gnome-orca openjdk-11-jre* compiz-core
sudo apt-get purge mono-runtime-common gnome-orca openjdk-11-jre* compiz-core
```

### Install software from binaries

| Package        | Link         |
| -------------  |------------|
| Atom        | https://github.com/atom/atom/releases |
| Skype       | https://www.skype.com/en/get-skype/   |
| Chrome      | https://www.google.com/chrome/        |
| Filebot     | https://www.filebot.net/              |

`sudo dpkg -i /path/to/package-name.deb`

Configure the Oracle JDK as default.
```bash
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk-11.0.1/bin/java 0
sudo update-alternatives --config java
```
### Install VMWare Workstation
* Install VMware-Workstation-Full-15.0.2-10952284.x86_64.bundle
* Enter serial `UC7W0-6WY91-H890Y-NQQ79-M60D8`

### Install Citrix Receiver for Linux
* Download [Citrix Receiver for Linux Full Package](https://www.citrix.es/downloads/citrix-receiver/linux/receiver-for-linux-latest.html)

```bash
sudo dpkg -i /path/to/package-name.deb
sudo apt-get -f install
```

### Add non-root user and group
```bash
sudo adduser USERNAME
sudo groupadd GROUPNAME
sudo usermod -a -G GROUPNAME USERNAME
```

### Assign permissions to key file system locations
```bash
sudo chgrp -R GROUPNAME /path/to/folder
sudo chmod -R 775 /path/to/folder
```

### Configure Chrome
[Configure Chrome](https://sites.google.com/site/easylinuxtipsproject/chrome)
* Start chrome with `google-chrome --password-store=basic`

* Settings -> Appearance
 * Show Home button
 * Always show the bookmarks bar
 * Use system title bar and borders


* Settings -> Advanced -> Languages
 * Offer to translate pages that aren't in a language you read


* Settings -> Advanced -> Privacy

 Ensure that only these three useful features are enabled:
 * Use a web service to help resolve navigation errors
 * Protect you and your device from dangerous sites
 * Send a "Do Not Track" request with your browsing traffic


* Settings -> Advanced -> Content settings -> Cookies
 * Keep local data only until you quit your browser

### Configure Firefox
[Configure Firefox](https://sites.google.com/site/easylinuxtipsproject/firefox)

* Preferences -> Privacy & Security
 * History -> Firefox will: Use custom settings for history
 * Clear history when Firefox closes
 * Click the button Settings.to the right of "Clear history when Firefox closes" and tick everything, except for Site Preferences

* Item Cookies and Site Data:
 * Accept cookies -> Keep until: Firefox is closed
 * Address Bar: remove the tick for: Browsing history
 * Tracking protection: leave those settings at their defaults

Type `about:config` in the URL bar of Firefox:

* `browser.cache.disk.smart_size.enabled` set to false.
* `browser.cache.disk.capacity` set to 102400.
* `dom.webnotifications.enabled` set to false.
* `browser.urlbar.maxRichResults` set to 0
* `browser.sessionstore.interval` set to 15000000

#### Make new tab pages empty
By default, when you open a new tab page, Firefox shows tiles of websites that you've previously visited.

In the new tab, click the gear icon in the top right of the new tab and remove all checks except the one for Search.

[Add Google as a search engine](https://sites.google.com/site/easylinuxtipsproject/searchbox)

### Firejail config
https://sites.google.com/site/easylinuxtipsproject/sandbox

### Disable Java in Libre Office
* Libre Office Toolbar -> Tools -> Options -> Advanced - Java Options
* Remove the tick for: Use a Java runtime environment

### Configure bookmarks
* Add custom bookmarks to `~/.config/gtk-3.0/bookmarks`
* There is a hidden menu to hide Shortcuts in the Side Pane.
* Right click in the Side Pane where there are no shortcuts, like on the DEVICES section label. Then you will get a pop-up menu where you can uncheck items you do not want displayed.

### Customize profile picture
* Add a 210x210 PNG image to Home directory with the filename  `.face`

### Configure gpodder
* Create a file called `gpodder.sh`
* Add the content below to the file

```bash
#!/bin/bash
# sets the home folder (database and downloads) for gPodder
# https://gpodder.github.io/docs/user-manual.html#changing-the-downloads-folder-location-and-the-gpodder-home-folder
export GPODDER_HOME=/opt/storage/gpodder
```
* copy the file to /etc/profile.d/gpodder.sh (requires sudo)

### Configure rclone
* Open Terminal and type `rclone config`
* n) New remote
* name> **gdrive**
* Storage> *Goolge Drive*
* client_id> *leave blank*
* client_secret> *leave blank*
* Remote config, use auto-config> *y**
* y) Yes this is OK
* q) Quit config

### Shrink a VMware Guest disk drive to reclaim unused space
 *Linux* Guest
 * Log on to Linux Guest
 * Open a Terminal
 * Issue the following command:
 `sudo vmware-toolbox-cmd disk shrink /`

 *Windows* Guest
 * Log on to Windows Guest
 * Start -> Control Panel -> VMware Tools

### Create a Live Linux USB with persistent file system
```bash
sudo add-apt-repository ppa:mkusb/ppa
sudo apt install mkusb
sudo mkusb /path/to/ubuntu_image.iso p
```

* Choose the e) option for the GUI
* Follow defaults and allocate the desired percentage for allocation to the file system

### Create and restore a USB disk image
* make the disk image

`sudo dd if=/dev/sd* conv=sync,noerror status=progress | gzip -c > /path/to/my-disk.image.gz`

* restore disk image

`sudo gunzip -c IMAGE.HERE-GZ | dd of=/dev/OUTPUT/DEVICE-HERE`
