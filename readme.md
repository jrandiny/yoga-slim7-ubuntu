# Linux on Lenovo Yoga Slim 7 AMD

## About

This are various tweaks and fix to run Linux on Lenovo Yoga Slim 7. This note is validated on the following configuration
- Ubuntu 20.04.1
- Lenovo Yoga Slim 7 AMD (Ryzen 7)

## Summary

### What works
- Battery
- Storage
- USB
- Keyboard
- Wifi, bluetooth
- Speakers
- Headphone jack
- Webcam

### What doesn't works 
- Microphone () 
- Graphic ()

### Unknown


## Fixes

### Graphic

By default Ubuntu 20.04.1 shipped with Linux 5.4, support for AMD 4000 graphics is still experimental on 5.4. To get the best result wait for Ubuntu 20.04.2 or upgrade manually to the latest kernel.

Some of the ways to upgrade the kernel:
- https://github.com/pimlie/ubuntu-mainline-kernel.sh
- https://github.com/bkw777/mainline

