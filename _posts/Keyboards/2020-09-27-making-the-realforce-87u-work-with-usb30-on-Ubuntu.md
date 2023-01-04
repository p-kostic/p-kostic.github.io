---
title: Making the Realforce 87U work with USB 3.0 on Ubuntu
date: 2020-09-27 21:34:53 +0100
categories: [Keyboards]
tags: [realforce87ub, ubuntu, issues]     # TAG names should always be lowercase
---

## Environment
OS: Ubuntu 20.04 LTS x86_64  
Host: XPS 15 9570  
Kernel: 5.4.0-33-generic  
DE: GNOME  

## Problem behavior
The Realforce 87U does not work under Ubunu / Linux, but does work under Windows, Grub, or the BIOS on the machine.

`$ dmesg | grep usb` will probably give you something like this, where it will try to read the configuration a bunch of times for your port (in my case `usb 1-2`), before it gives up.

```
[    1.375845] usb 1-2: new full-speed USB device number 3 using xhci_hcd
[    1.525302] usb 1-2: unable to read config index 0 descriptor/start: -32
[    1.525304] usb 1-2: chopping to 0 config(s)
[    1.525305] usb 1-2: can't read configurations, error -32
[    1.652127] usb 1-2: new full-speed USB device number 4 using xhci_hcd
[    1.802977] usb 1-2: unable to read config index 0 descriptor/start: -32
[    1.802979] usb 1-2: chopping to 0 config(s)
[    1.802981] usb 1-2: can't read configurations, error -32
[    1.803021] usb usb1-port2: attempt power cycle
[    2.452227] usb 1-2: new full-speed USB device number 5 using xhci_hcd
[    2.475648] usb 1-2: unable to read config index 0 descriptor/start: -32
[    2.475650] usb 1-2: chopping to 0 config(s)
[    2.475653] usb 1-2: can't read configurations, error -32
[    2.604235] usb 1-2: new full-speed USB device number 6 using xhci_hcd
[    2.628059] usb 1-2: unable to read config index 0 descriptor/start: -32
[    2.628061] usb 1-2: chopping to 0 config(s)
[    2.628062] usb 1-2: can't read configurations, error -32
[    2.628101] usb usb1-port2: unable to enumerate USB device
```

This happened to me on USB ports that are of version 3.0. Users have reported that the keyboard will work when using a USB 2.0 hub.

## Solution
I've found that turning off my laptop, unplugging the keyboard, booting into Ubuntu and plugging the keyboard in again did nothing for me. What did change something, however, is that unplugging and plugging in the keyboard a couple of times will cause this message to show op in `dmesg`

```
[  264.151488] usb usb1-port2: attempt power cycle
```

After this, The product and manufacturer are properly identified, along with the device ids

```
[  264.801787] usb 1-2: new full-speed USB device number 12 using xhci_hcd
[  264.825063] usb 1-2: New USB device found, idVendor=0853, idProduct=0111, bcdDevice= 0.01
[  264.825068] usb 1-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  264.825071] usb 1-2: Product: Realforce 87
[  264.825074] usb 1-2: Manufacturer: Topre Corporation
[  264.828792] input: Topre Corporation Realforce 87 as /devices/pci0000:00/0000:00:14.0/usb1/1-2/1 2:1.0/0003:0853:0111.0004/input/input31
[  264.889528] hid-generic 0003:0853:0111.0004: input,hidraw3: USB HID v1.11 Keyboard [Topre Corporation Realforce 87] on usb-0000:00:14.0-2/input0
```
Having this information allows us to pass usbcore as a kernelparameter using `usbcore.quirks`, and set the proper values for USB 2.0

More specifically, the following options
* `g` for `USB_QUIRK_DELAY_INIT` (device needs a pause during initialization, after we read the device descriptor);
* `k` for `USB_QUIRK_NO_LPM` (device can't handle Link Power Management);
* `i` for `USB_QUIRK_DEVICE_QUALIFIER` (device can't handle device_qualifier descriptor requests)

With the kernel parameter being
```
usbcore.quirks=<idVendor>:<idProduct>:<option><option><option>...
```

For me, `dmesg | grep "usb 1-2"` lists 
```
[  264.825063] usb 1-2: New USB device found, idVendor=0853, idProduct=0111, bcdDevice= 0.01
```
So the kernel parameter I added was
```
usbcore.quirks=0853:0111:gki
```

### Testing 
* Reboot into grub and add your kernel parameters by hitting `e` and appending it under `linux`
* Hit Ctrl+X to exit emacs and boot
* Test your keyboard, typing `dmesg | grep usb` again after booting to see if the issue with the keyboard is resolved

### Making the changes permanent
* `sudo` open `/etc/default/grub` with your favorite editor.
* Add the parameter to the very end.
    ```
    GRUB_CMDLINE_LINUX_DEFAULT="quiet slash mem_sleep_default=deep ... usbcore.quirks=0853:0111:gki"
    ```
* Update grub's configuration with `sudo update-grub`
* Reboot 


## References
* [https://qiita.com/la_float/items/fed43d540c8e2201b543](https://qiita.com/la_float/items/fed43d540c8e2201b543)
* [https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1678477](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1678477)
* [https://raw.githubusercontent.com/torvalds/linux/master/Documentation/admin-guide/kernel-parameters.txt](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1678477)