---
title: How to change the GDM login screen background
date: 2020-09-27 21:54:53 +0100
categories: [Linux, Gnome]
tags: [customization]  # TAG names should always be lowercase
---

## Environment
OS: Ubuntu 19.10 x86_64  
Host: XPS 15 9570  
DE: GNOME  

## Procedure

Open the following file using your favorite editor with root access (e.g. `vim`, `nano`, `gedit`, `code`).
```
sudo vim /usr/share/gnome-shell/theme/gnome-shell.css
```
Search for something like this 
```css
#lockDialogGroup {
   background: #2c001e url(resource:///org/gnome/shell/theme/noise-texture.png);
   background-repeat: repeat; 
}
```
Change the color code, or replace it with a image file 
```css
#lockDialogGroup {
   background: #2c001e url(file:///home/<username>/pictures/background.png);
   background-repeat: no-repeat; 
   background-size: cover;
   background-position: center;
}
```
Do the exact same to the following
```
sudo vim /usr/share/gnome-shell/theme/gdm3.css 
```
Done, sign out and back in again or reboot your system to see the results