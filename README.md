!!Installation Manual for AMiRo!!
=================================

For new deployments, the driver is already installed, but update the already running robots, do the following:

1. For AMiRo build with Yocto 1.7.3 and Linux 3.19.0 [this binary module](binaries/8192cu.ko) can be copied to the AMiRo: `scp 8192cu.ko root@${AMIRO_IP}:/lib/modules/3.19.0-yocto-standard/extra/`
2. Disable the old driver by adding them to the blacklist on AMiRO: `echo -e "blacklist rtlwifi\nblacklist rtl_usb\nblacklist rtl8192c_common\nblacklist rtl8192cu\n" >> /etc/modprobe.d/blacklist.conf`
3. Update the available modules on AMiRo: `depmod -a`


Further Information from Original Repo
======================================

[![Build Status](https://travis-ci.org/pvaret/rtl8192cu-fixes.svg?branch=master)](https://travis-ci.org/pvaret/rtl8192cu-fixes)

This is a repackaging of Realtek's own 8192CU USB WiFi driver for Ubuntu 13.10 and later.

This driver is not explicitly maintained.

The new `rtl8xxxu` driver initially introduced in kernel 4.4 works mostly well these days, and you should give it a try before trying this repository.

If `rtl8xxxu` gives you problems, try troubleshooting it first. Known things to look for are:
  - Make sure to blacklist the older `rtl8192cu` driver, which some distros seem to load by default otherwise.
  - Some devices require that power management be disabled in NetworkManager. Follow the instructions further down to disable power management in NetworkManager. Typical symptoms would be that the device works fine for a moment, and then becomes very slow or outright drops the connection.

In some cases, though, this driver has been known to work fine where `rtl8xxxu` doesn't. If `rtl8xxxu` doesn't work for you even after the troubleshooting steps listed above, follow the instructions below to install this one instead.

Compatibility
=============

These devices are known to work with this driver:
- ASUSTek USB-N13 **rev. B1** (0b05:17ab)
- Belkin N300 (050d:2103)
- D-Link DWA-121 802.11n Wireless N 150 Pico Adapter [RTL8188CUS]
- Edimax EW-7811Un (7392:7811)
- Kootek KT-RPWF (0bda:8176)
- OurLink 150M 802.11n (0bda:8176)
- TP-Link TL-WN821N**v4** (0bda:8178)
- TP-Link TL-WN822N (0bda:8178)
- TP-Link TL-WN823N (only models that use the rtl8192cu chip)
- TRENDnet TEW-648UBM N150

These devices are known not to be supported:
- Alfa AWUS036NHR
- TP-Link WN8200ND

As a rule of thumb, this driver generally works with devices that use the RTL8192CU chipset, and some devices that use the RTL8188CUS, RTL8188CE-VAU and RTL8188RU chipsets too, though it's more hit and miss.

Devices that use dual antennas are known not to work well. This appears to be an issue in the upstream Realtek driver.

Installation
============

Ensure you have the necessary prerequisites installed:

    sudo apt-get update
    sudo apt-get install git linux-headers-generic build-essential dkms

Clone this repository:

    git clone https://github.com/pvaret/rtl8192cu-fixes.git

Set it up as a DKMS module:

    sudo dkms add ./rtl8192cu-fixes

Build and install it:

    sudo dkms install 8192cu/1.11

Refresh the module list:

    sudo depmod -a

Ensure the native (and broken) kernel driver is blacklisted:

    sudo cp ./rtl8192cu-fixes/blacklist-native-rtl8192.conf /etc/modprobe.d/

And reboot. You're done.

Other distributions
===================

The instructions above should work in every Debian-based distribution. So long as you can install the prerequisites on your own, everything after the line that contains `apt-get` should work in other distributions as well.

On Gentoo, you can disregard the instructions above and just install the following ebuild: https://gitweb.gentoo.org/dev/maksbotan.git/tree/sys-kernel/rtl8192cu-fixes/rtl8192cu-fixes-9999.ebuild

Troubleshooting
===============

There is a known issue with power management on some hardware. If your WiFi connection drops after a few minutes, install the following module setting file to disable power management in your WiFi interface:

    sudo cp ./rtl8192cu-fixes/8192cu-disable-power-management.conf /etc/modprobe.d/

And then reboot.

Sometimes Network Manager also sets a device in a power-saving mode where it doesn't use enough power to connect. You can fix it by editing `/etc/NetworkManager/conf.d/default-wifi-powersave-on.conf` and setting `wifi.powersave` to 2. And then reboot.

Current status
==============

As it currently stands, the driver doesn't populate /proc with informational data from the driver. The API for /proc has changed in recent kernels, and the driver has not been ported to the new API.

Credits
=======

This repository was initially based on Timothy Phillips's work as published here: https://code.google.com/p/realtek-8188cus-wireless-drivers-3444749-ubuntu-1304/, though no longer.

Thanks go to Saqib Razaq (@s-razaq) for the power management workaround.

Thanks to @CGarces for the Travis configuration.

Thanks to @rburcham for the kernel 4.15 fixes.
