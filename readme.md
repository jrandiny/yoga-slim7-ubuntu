# Linux on Lenovo Yoga Slim 7 AMD

## About

This are various tweaks and fix to run Linux on Lenovo Yoga Slim 7. This note is validated on the following configuration
- Ubuntu 20.04.1
- Lenovo Yoga Slim 7 AMD (Ryzen 7)

## Summary

Legend:
- :heavy_check_mark: Works out of the box (Kernel 5.4)
- :hammer_and_wrench: Require tweaking
- :negative_squared_cross_mark: Not working
- :question: Unknown

| Feature                      | Status              | Description                                                           |
| ---------------------------- | ------------------- | --------------------------------------------------------------------- |
| Power (battery and charging) | :heavy_check_mark:  |                                                                       |
| Storage                      | :heavy_check_mark:  | Disable bitlocker on windows to access windows partition from Linux   |
| Graphic                      | :hammer_and_wrench: | Kernel update is required (see [below](#Graphic))                     |
| USB                          | :heavy_check_mark:  |                                                                       |
| Keyboard                     | :heavy_check_mark:  |                                                                       |
| Speakers                     | :heavy_check_mark:  |                                                                       |
| Microphone                   | :heavy_check_mark:  | It seems there's a bug on kernel 5.7, please use other kernel version |
| Audio jack                   | :heavy_check_mark:  |                                                                       |
| Wifi and Bluetooth           | :heavy_check_mark:  |                                                                       |
| Webcam                       | :heavy_check_mark:  |                                                                       |
| External display (HDMI)      | :hammer_and_wrench: | Kernel update is required (see [below](#Graphic))                     |
| Suspend                      | :hammer_and_wrench: | See detail [below](#Suspend)                                          |

## Fixes

**DISCLAIMER** I am not responsible for any damage and negative consequences to your system

### Graphic

By default Ubuntu 20.04.1 shipped with Linux 5.4, support for AMD 4000 graphics is still experimental on 5.4. To get the best result wait for Ubuntu 20.04.2 or upgrade manually to the latest stable kernel (5.8 by the time of publication).

Some of the ways to upgrade the kernel:
- https://github.com/pimlie/ubuntu-mainline-kernel.sh
- https://github.com/bkw777/mainline

### Suspend

#### Background

In recent times Microsoft has introduced something called "[Modern Standby](https://docs.microsoft.com/en-us/windows-hardware/design/device-experiences/modern-standby)" which is essentially a new way to suspend with the advantage of allowing the system to do some task while suspending (e.g. fetching emails). In order to support this mode, the BIOS must not advertise support for the traditional suspend (S3) <sup>[Citation needed]</sup> 

```bash
$ dmesg | grep ACPI:\ \(
[    0.383096] ACPI: (supports S0 S4 S5)

```

By not advertising support for S3, the kernel will only support s2idle sleep mode which is also supported by Linux

```bash
$ cat /sys/power/mem_sleep 
[s2idle]
```

However there seems to be a problem with the amdgpu driver on resuming from suspend in this mode.

```
amdgpu 0000:03:00.0: [drm:amdgpu_ring_test_helper [amdgpu]] *ERROR* ring gfx test failed (-110)
[drm:amdgpu_device_ip_resume_phase2 [amdgpu]] *ERROR* resume of IP block <gfx_v9_0> failed -110
[drm:amdgpu_device_resume [amdgpu]] *ERROR* amdgpu_device_ip_resume failed (-110).
```

#### Possible Solutions

There are three solutions:
- Wait until the problem in amdgpu driver is fixed in newer kernel version
- Wait until Lenovo adds an option in the UEFI to advertise S3 support (similar to the options available in Thinkpad)
- Modify the system to advertise S3 support (see below)

#### Advertise S3

All of the following commands assume root shell

1. Get the required tools

Download and compile from https://www.acpica.org/downloads
```bash
# Install some required dependencies
apt install m4 build-essential bison flex

# Download and extract
wget https://acpica.org/sites/acpica/files/acpica-unix-20200717.tar.gz
tar -xvf acpica-unix-20200717.tar.gz
cd acpica-unix-20200717

# Make
make clean
make

# Add to PATH for easy access
PATH=$PATH:$(realpath ./generate/unix/bin/)
```

2. Dump the ACPI files and decompile the DSDT table

```bash
# Create new working directory
mkdir acpi
cd acpi

# Dump acpi table to binary files
acpidump -b

# Decompile dsdl.bat with all of the .dat as external symbol
iasl -e *.dat -d dsdt.dat
```

3. Apply patch

Copy dsdt.patch from this repo and patch dsdt.dsl
```bash
patch <dsdt.patch
```

4. Recompile the modified table

```bash
# Recompile dsdt to new hex asml table (ignore warning)
iasl -ve -tc dsdt.dsl
```

5. Make override archive
```bash
mkdir -p kernel/firmware/acpi
cp dsdt.aml kernel/firmware/acpi
find kernel | cpio -H newc --create > acpi_s3_override

# Copy to /boot
cp acpi_s3_override /boot/
```

6. set the default sleep type to S3 (deep)

Open `etc/default/grub` and add `mem_sleep_default=deep` to `GRUB_CMDLINE_LINUX_DEFAULT` then run `update-grub`

Example:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash mem_sleep_default=deep"
```

7. Set grub to use the override 

Open `etc/default/grub` and add `acpi_s3_override` to `GRUB_CMDLINE_LINUX_DEFAULT` then run `update-grub`

Example:
```
GRUB_EARLY_INITRD_LINUX_CUSTOM="acpi_s3_override"
```

**NOTE**

There's seem to be a problem with the default grub package on Ubuntu (https://bugs.launchpad.net/ubuntu/+source/grub2/+bug/1891131) preventing the use of `GRUB_EARLY_INITRD_LINUX_CUSTOM` option. In the meantime, you can either patch the config generator or adding the required modification to grub.cfg manually. 

**Option 1 - Patching the config generator**

Download the 10_linux.patch from this repo and move it to `/etc/grub.d`
```bash
cd /etc/grub.d
# Backup original
cp 10_linux ../10_linux.orig

# Patch
patch <10_linux.patch

# Update grub
update-grub
```

**Option 2 - Manually edit grub.cfg**

Open `/boot/grub/grub.cfg` and search for the every line starting with `initrd` followed by initrd image

```
initrd /boot/initrd.img-5.8.0-050800-generic
```

Add `/boot/acpi_s3_override` in the middle

```
initrd /boot/acpi_override /boot/initrd.img-5.8.0-050800-generic
```


## Thanks
- @SteveImmanuel for the information regarding microphone on kernel 5.7
- https://www.reddit.com/r/linuxhardware/comments/i28nm5/ideapad_14are05_s3_sleep_fix/ 
- https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X1_Yoga_(Gen_3)#Enabling_S3_(before_BIOS_version_1.33)