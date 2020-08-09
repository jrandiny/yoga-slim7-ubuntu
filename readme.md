# Linux on Lenovo Yoga Slim 7 AMD

## About

This are various tweaks and fix to run Linux on Lenovo Yoga Slim 7. This note is validated on the following configuration
- Ubuntu 20.04.1
- Lenovo Yoga Slim 7 AMD (Ryzen 7)

## Summary

Legend:
- :heavy_check_mark: Works out of the box (Kernel 5.4)
- :white_check_mark: Works on latest kernel (Tested with Kernel 5.8)
- :hammer_and_wrench: Require tweaking
- :negative_squared_cross_mark: Not working
- :question: Unknown

| Feature                      | Status             | Description                                                                   |
| ---------------------------- | ------------------ | ----------------------------------------------------------------------------- |
| Power (battery and charging) | :heavy_check_mark: |                                                                               |
| Storage                      | :heavy_check_mark: | Disable bitlocker on windows to access windows partition from Linux           |
| Graphic                      | :white_check_mark: | Kernel update is required (see )                                              |
| USB                          | :heavy_check_mark: |                                                                               |
| Keyboard                     | :heavy_check_mark: |                                                                               |
| Speakers                     | :heavy_check_mark: |                                                                               |
| Microphone                   | :question:         | It's working on my device but there are some reports online stating otherwise |
| Audio jack                   | :heavy_check_mark: |                                                                               |
| Wifi and Bluetooth           | :heavy_check_mark: |                                                                               |
| Webcam                       | :heavy_check_mark: |                                                                               |

## Fixes

### Graphic

By default Ubuntu 20.04.1 shipped with Linux 5.4, support for AMD 4000 graphics is still experimental on 5.4. To get the best result wait for Ubuntu 20.04.2 or upgrade manually to the latest stable kernel.

Some of the ways to upgrade the kernel:
- https://github.com/pimlie/ubuntu-mainline-kernel.sh
- https://github.com/bkw777/mainline

