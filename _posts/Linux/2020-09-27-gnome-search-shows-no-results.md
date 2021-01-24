---
title: Gnome search shows no results, but hitting enter will actually launch the application
date: 2020-09-27 21:54:53 +0100
categories: [Linux, Gnome]
tags: [issues]  # TAG names should always be lowercase
---

## Environment
OS: Ubuntu 19.10 x86_64  
Host: XPS 15 9570  
DE: GNOME  

### Problem
When you hit the windows key and start typing to search for something within gnome, there are no results. However, when you hit enter it will in fact launch the application that was the top search result.

## Solution
This is probably caused by an update from 18.04 LTS. You were a ricing and installed a bunch of GDM themes. The script gnome-look that edits these GDM themes had some incompatibilities with the upgrade. To solve it, just reinstall the yaru theme

`sudo apt-get --reinstall install yaru-theme-gnome-shell`

Then press `alt+f2` and type `r` in the text box that shows up to reload GNOME.