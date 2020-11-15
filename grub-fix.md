# Add Support for GRUB_EARLY_INITRD_LINUX_CUSTOM

On Ubuntu old default grub package there's seem to be a bug (https://bugs.launchpad.net/ubuntu/+source/grub2/+bug/1878705) preventing the use of `GRUB_EARLY_INITRD_LINUX_CUSTOM` option. The bug is fixed on grub-common 2.04-1ubuntu26.3 however this page is preserved for future reference.

**Option 1 - Patching the config generator**

(This is no longer necessary, see above)

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

(This is no longer necessary, see above)

Open `/boot/grub/grub.cfg` and search for the every line starting with `initrd` followed by initrd image

```
initrd /boot/initrd.img-5.8.0-050800-generic
```

Add `/boot/acpi_s3_override` in the middle

```
initrd /boot/acpi_override /boot/initrd.img-5.8.0-050800-generic
```