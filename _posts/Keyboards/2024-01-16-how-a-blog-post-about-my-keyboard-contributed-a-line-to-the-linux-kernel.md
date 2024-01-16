---
title: How a blog post about my keyboard contributed a line to the Linux kernel
date: 2024-01-16 22:42:53 +0100
categories: [Keyboards]
tags: [realforce87ub, ubuntu, linux]     # TAG names should always be lowercase
image:
  path: /assets/img/posts/keyboards/keyboard.png
---

## Context

Back in 2020, I had some problems with the USB interface of my Realforce 87UB keyboard on Ubuntu. Despite my initial struggles, I conducted some research and documented my findings in a small blog post, which you can find [here](/posts/making-the-realforce-87u-work-with-usb30-on-Ubuntu/).

According to Google Search Console, the post didn't really get any traction. It's not a popular keyboard anyway, so I didn't expect it to.

Then, out of seemingly nowhere, on the 27th of October 2022, I was added as a CC to of a patch in the Linux Kernel Mailing List. The patch was titled `[PATCH] usb: add NO_LPM quirk for Realforce 87U Keyboard`. The corresponding details can be found here:

* [lore.kernel.org](https://lore.kernel.org/all/20221109122946.706036-1-ndumazet@google.com/) (has more info)
* [github.com/torvalds/linux](https://github.com/torvalds/linux/commit/181135bb20dcb184edd89817831b888eb8132741) (generally easier to read, especially if you're occasionally on GitHub anyway)

I honestly didn't know that some devices had their quirks hardcoded in the kernel. I didn't really expect my post to be of any use to anyone, let alone a permanent addition to the Linux kernel. I was pleasantly surprised to see that it was.

## The patch

The patch adds the following line to `drivers/usb/core/quirks.c`:

```c
    /* Realforce 87U Keyboard */
    { USB_DEVICE(0x0853, 0x011b), .driver_info = USB_QUIRK_NO_LPM },	
```

This line is added to the `usb_quirk_list` array, which is used to identify devices that need to be treated differently by the kernel. In this case, the `USB_QUIRK_NO_LPM` flag is set, which disables the *USB 3.0 Link Power Management* for this device. This renders my initial blog post obsolete, as the kernel now automatically handles the LPM issue.

The previous blog post will still be available if people experience similar issues with other keyboards, or if they want to know how to add a quirk through grub themselves. Nevertheless, I added a note to the top of the post with a short explanation, and a link to this post. 

## Release information
The patch was made available in all of the current longterm, stable, and mainline releases of the Linux kernel

| Kernel version | Patch/Release + Changelog link                                                     | Date        | 
| :------------- | ---------------------------------------------------------------------------------- | ----------- | 
|  v6.x          | 6.1 [(release)](https://cdn.kernel.org/pub/linux/kernel/v6.x/ChangeLog-6.1)        | 12-Dec-2022 | 
|  v6.x          | 6.0.10 [(patch)](https://cdn.kernel.org/pub/linux/kernel/v6.x/ChangeLog-6.0.10)    | 26-Nov-2022 | 
|  v5.x          | 5.15.80 [(patch)](https://cdn.kernel.org/pub/linux/kernel/v5.x/ChangeLog-5.15.80)  | 26-Nov-2022 |
|  v5.x          | 5.4.225 [(patch)](https://cdn.kernel.org/pub/linux/kernel/v5.x/ChangeLog-5.4.225)  | 25-Nov-2022 |
|  v4.x          | 4.19.267 [(patch)](https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.19.267)| 25-Nov-2022 |
|  v4.x          | 4.9.334 [(patch)](https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.9.334)  | 25-Nov-2022 |

> Tip: Do not click on the `6.1 (release)` link, as it will open the entire changelog in your browser, which is quite big for a release (and will probably crash your browser). 
{: .prompt-tip }


## Acknowledgement

I'd like to express my gratitude towards the individual who submitted the patch, mentioning me in the mailing list, and the commit description. Additionally, I'd like to thank the people mentioned in the Mailing List, who reviewed and accepted the patch.