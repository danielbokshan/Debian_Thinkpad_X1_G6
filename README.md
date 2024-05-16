# My Debian Thinkpad X1 Carbon Config
## Purpose of This Document
I'm putting this all here so I can recreate this setup if necessary, including Debian-specific instructions for which I could only find guides on Ubuntu/Arch. Debian is my Linux Distribution of choice because I find that it provides the best mix of DIY and Presets. It also doesn't have Snap installed and baked in by default. 

# System Info
This is a Thinkpad X1 Carbon Gen 6 from 2018 with 8GB ram and the core i5 processor.
```
$ sudo dmidecode -s system-version
ThinkPad X1 Carbon 6th
```

```
$ uname -a | cut -d ' ' -f 3
6.1.0-17-amd64
```
```
$ cat /etc/*-release
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```
```
$ lspci
00:00.0 Host bridge: Intel Corporation Xeon E3-1200 v6/7th Gen Core Processor Host Bridge/DRAM Registers (rev 08)
00:02.0 VGA compatible controller: Intel Corporation UHD Graphics 620 (rev 07)
00:04.0 Signal processing controller: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor Thermal Subsystem (rev 08)
00:08.0 System peripheral: Intel Corporation Xeon E3-1200 v5/v6 / E3-1500 v5 / 6th/7th/8th Gen Core Processor Gaussian Mixture Model
00:14.0 USB controller: Intel Corporation Sunrise Point-LP USB 3.0 xHCI Controller (rev 21)
00:14.2 Signal processing controller: Intel Corporation Sunrise Point-LP Thermal subsystem (rev 21)
00:16.0 Communication controller: Intel Corporation Sunrise Point-LP CSME HECI #1 (rev 21)
00:1c.0 PCI bridge: Intel Corporation Sunrise Point-LP PCI Express Root Port #1 (rev f1)
00:1c.4 PCI bridge: Intel Corporation Sunrise Point-LP PCI Express Root Port #5 (rev f1)
00:1d.0 PCI bridge: Intel Corporation Sunrise Point-LP PCI Express Root Port #9 (rev f1)
00:1f.0 ISA bridge: Intel Corporation Sunrise Point LPC Controller/eSPI Controller (rev 21)
00:1f.2 Memory controller: Intel Corporation Sunrise Point-LP PMC (rev 21)
00:1f.3 Audio device: Intel Corporation Sunrise Point-LP HD Audio (rev 21)
00:1f.4 SMBus: Intel Corporation Sunrise Point-LP SMBus (rev 21)
00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection (4) I219-V (rev 21)
02:00.0 Network controller: Intel Corporation Wireless 8265 / 8275 (rev 78)
04:00.0 Non-Volatile memory controller: Intel Corporation SSD Pro 7600p/760p/E 6100p Series (rev 03)
```

```
$ neofetch
       _,met$$$$$gg.          daniel@thinkpad-x1 
    ,g$$$$$$$$$$$$$$$P.       ------------------ 
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 12 (bookworm) x86_64 
 ,$$P'              `$$$.     Host: 20KGS4BQ00 ThinkPad X1 Carbon 6th 
',$$P       ,ggs.     `$$b:   Kernel: 6.1.0-17-amd64 
`d$$'     ,$P"'   .    $$$    Uptime: 4 days, 5 hours, 24 mins 
 $$P      d$'     ,    $$P    Packages: 1958 (dpkg) 
 $$:      $$.   -    ,d$$'    Shell: bash 5.2.15 
 $$;      Y$b._   _,d$P'      Resolution: 1920x1080 
 Y$$.    `.`"Y$$$$P"'         DE: GNOME 43.9 
 `$$b      "-.__              WM: Mutter 
  `Y$$                        WM Theme: Adwaita 
   `Y$$.                      Theme: Adwaita [GTK2/3] 
     `$$b.                    Icons: Adwaita [GTK2/3] 
       `Y$$b.                 Terminal: gnome-terminal 
          `"Y$b._             CPU: Intel i5-8250U (8) @ 3.400GHz 
              `"""            GPU: Intel UHD Graphics 620 
                              Memory: 5147MiB / 7698MiB 

                                                      
                                                      
```


# Fixes For Full Functionality

## Battery Life
```
sudo apt-get install tlp tlp-rdw acpi-call-dkms tp-smapi-dkms acpi-call-dkms acpitool
```
## Throttled
This program stops the laptop from randomly locking up due to cpu temps
- Repository: [https://github.com/erpalma/throttled](https://github.com/erpalma/throttled)
Install and Enable:
```
sudo apt install git virtualenv build-essential python3-dev libdbus-glib-1-dev libgirepository1.0-dev libcairo2-dev python3-venv
```
```
git clone https://github.com/erpalma/lenovo-throttling-fix.git
```
```
sudo ./install.sh
```
```
sudo systemctl enable --now lenovo_fix.service
```

## Fingerprint Reader
This was a real pain to get working because all guides are written for Arch or Ubuntu, but I got it working!
- Original Guide: [https://blog.horner.tj/mint-fingerprint-auth-x1c7/](https://blog.horner.tj/mint-fingerprint-auth-x1c7/)
### My Steps:
- Remove OS's built-in fprintd
```$ sudo apt remove fprintd```

- Add the following to /etc/apt/sources.list:
Yes, I know that the source points to a branch for Ubuntu but it works fine on Debian.
I have also uploaded the entire file to this repo.
```
# python-validity
deb http://ppa.launchpad.net/uunicorn/open-fprintf/ubuntu jammy main
```

- Update sources
```
$ sudo apt-get update
```

- Install python-validity
```
$ sudo apt install open-fprintd fprintd-clients python3-validity
```
- Update PAM Module to tell OS to accept fingerprint as auth; use space bar to check the box within the editor program.
```
$ sudo pam-auth-update
```

- Enroll fingerprints
```$ fprintd-enroll```

- Add the following into a service file and enable it. This fixes a known bug within fprintd which makes the fingerprint scanner not come back to life after hibernating (which therefore makes it useless)
```
[Unit]
Description=Restart services to fix fingerprint integration with fprintd
After=suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target

[Service]
Type=oneshot
ExecStart=systemctl restart open-fprintd.service python3-validity.service

[Install]
WantedBy=suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target

```
# My Desktop Environment
I'm using Gnome as my DE, with some tweaks to make it better :)

## Tweaks & Extensions
- Hack Regular Font
### From extensions.gnome.org:
- Control Blur Effect on Lock Screen
- Hide Activities Button
- Transparent Top Bar (Adjustable Transparency) - opaque bar when a window touches it
- Vitals (showing fan RPM, CPU %, CPU Temp, RAM usage, and battery time estimate)

### Built-In:
- Dash to Dock (left side, shrink the dash, dynamic opacity)
- User Themes
- Removable Drive Menu

### Startup Sound
- Download the sound you want as your startup sound in the form of a .ogg
    - I used Audacity to export as .ogg
- Install mpv media player
- Create a script that plays the .ogg file with mpv
```
#!/bin/bash

mpv --no-video /home/daniel/Music/RunForTheHills.ogg
exit
```
- Add the script to the Gnome Launchpad as a program using this tutorial: [https://askubuntu.com/questions/141229/how-to-add-a-shell-script-to-launcher-as-shortcut](https://askubuntu.com/questions/141229/how-to-add-a-shell-script-to-launcher-as-shortcut)
- Set the program up under "Startup Apps" within Gnome Tweaks app.

### Spicetify
- Link: [https://spicetify.app/](https://spicetify.app/)
- Theme: Sleek / Cherry

### Terminal/VSCode
- Theme: Gruvbox Dark Medium
