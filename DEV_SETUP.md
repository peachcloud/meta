# PeachCloud

## Development Environment

### Background

In order to be flexible in supporting single-board computers beyond the Raspberry Pi alone, and to take full advantage of 64-bit support on ARMv8 devices, development for PeachCloud is being targeted at Debian ARM64.

No official image has yet been released by Debian for the Raspberry Pi 3, though a preview image of Debian 10 (Buster) is available [from their wiki](https://wiki.debian.org/RaspberryPi3). The major known issue with that preview image is broken wireless (the built-in wireless interface does not work). Fortunately, Michael Stapelberg has posted [an updated version of the preview image](https://people.debian.org/~stapelberg/2018/01/08/raspberry-pi-3) with WiFi that works out of the box.

### Operating System

Debian 10 (Buster). Kernel version: 4.14.0-3-arm64.

### Hardware

Raspberry Pi 3B+.

### Packages

List of all packages manually-installed during development environment setup. 

`man-db, locales, sudo, vim, git, build-essential, wget, network-manager, net-tools, curl, hostapd, dnsmasq, openssh-server, iptables-persistent, ufw, bridge-utils, ntp, ssl-cert, wpasupplicant, avahi-daemon, libssl-dev, pkg-config, i2c-tools, nmtui`

_Note: This list may be incomplete and will be checked during the next phase of development._

### Software

Install Rust using [rustup](https://rustup.rs/):

`curl https://sh.rustup.rs -sSf | sh`

_Note: This method of installation will fail with SSL cert errors if the datetime is not correctly set on the Pi and / or the `ssl-cert` package is not installed._

Set Rust to nightly (necessary for compiling [Rocket](https://rocket.rs)):

`rustup default nightly`

### Hardware

**RTC over I²C**

_Tested with DS1338 RTC module._

Ensure `i2c-tools` Debian package has been installed. Then run the following to ensure the RTC is correctly wired and connected to the Pi:

`sudo modprobe i2c-dev`  
`sudo i2cdetect -y 1`

The final command in the sequence prints an array to the console, with 68 denoting the presence of the RTC module. This is a sign that the device is properly wired and connected.

Append the following two lines to `/boot/firmware/config.txt`:

`dtoverlay=i2c-rtc,ds1307`  
`dtparam=i2c_arm=on`

Add required kernel modules:

`sudo modprobe i2c-bcm2835`  
`sudo modprobe rtc-ds1307`

Append the following three lines to `/etc/modules`:

`i2c-dev`  
`i2c-bcm2835`  
`rtc-ds1307`

Apply the `dtb` patch detailed in the **Bug Fixes** section of this document (below).

Initialize the device by running the following commands:

`sudo bash`  
`echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device`  
`exit`

Run the `i2cdetect` tool to ensure the RTC kernel driver is working:

`sudo i2cdetect -y 1`

A `UU` entry in the output arrays denotes success. If you still see `68` or similar, the module is not being controlled by the kernel driver.

_Note: This Scuttlebutt dev-diary post by @glyph documents the process step-by-step (%aEVy1gyTSl4qrbazrwrgnrLs4pRVobELwQjH/TUtsAc=.sha256)._

**GPIO**

GPIO appears to be working out-of-the-box, but be aware that all pin numbers are offset by 458 for Debian on the Raspberry Pi 3 (as mentioned in the [Debian RaspberryPi3 wiki](https://wiki.debian.org/RaspberryPi3)).

### Bug Fixes

The clock frequency property is not set for I²C in the device tree of the Debian image we are working with. The `dtb` (device tree blob) must be decompiled, patched, and recompiled.

Copy the existing `dtb` to a temporary working directory (exact location is up to you):

`cp /boot/firmware/bcm2837-rpi-3-b.dtb /tmp/`  
`cd /tmp`

Install the compiler / decompiler:

`sudo apt-get install device-tree-compiler`

Generate a human-readable decompiled device tree (`dts`):

`dtc -I dtb -O dts > bcm2837-rpi-3-b.dts`

Open the `dts` and add the clock-frequency property (line 570):

`clock-frequency = <0x186a0>;`

Save and exit the document. Recompile the patched `dts` to binary blob format:

`dtc -O dtb -o bcm2837-rpi-3-b.dtb bcm2837-rpi-3-b.dts`

Replace the existing `dtb` with the newly-patched version (you may wish to backup the old `dtb` first):

`cp bcm2837-rpi-3-b.dtb /boot/firmware/bcm2837-rpi-3-b.dtb`

Reboot and ensure no `i2c could not read clock-frequency property` errors persist in the kernel logs (`/var/log/kern.log`).
