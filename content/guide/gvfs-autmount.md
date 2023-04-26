---
title: "Automatically mounting remote shares in GNOME"
description: >
    Samba sucks, you don't need it. There's a good chance you don't need to install anything to follow this guide. Just a recent-ish version of GNOME and a text editor.
date: 2023-02-24T15:36:21-05:00
---

# Automatically mounting remote shares in GNOME

Moving between many devices and having to find and open remote shares in GNOME is annoying. Using built-in GNOME tools like *gio*, it's possible to automatically mount network shares when you log in **without** wrangling with Samba or fstab. It works for [many protocols][gvfs-protos], including *smb*, *webdav*, and *ftp*.
<!--more-->

## Requirements

This guide works with the GNOME Shell, and tested on GNOME 43 on Fedora 37. If *glib2* is installed, *gio* is available, and your desktop environment supports XDG, everything should work. Before continuing, make sure that the credentials for the network share are **saved on the GNOME Keyring**.

## Getting Started

To mount the remote location on login, create a **desktop entry** in the autostart directory. The autostart directory is usually *~/.config/autostart*. If that folder doesn't exist, create it. Give your desktop file a unique filename, like *mount-example-share.desktop*. The content should look like the example below, with *$SHARE_URL_HERE* replaced with the URL to the share.

```ini
[Desktop Entry]
Version=1.5
Name=Mount Example Share
TryExec=gio
Exec=gio mount $SHARE_URL_HERE
Terminal=false
Type=Application
X-GNOME-Autostart-enabled=true
```

On the next login, the remote share should be available in the Files app. To add more shares, create more desktop files in the autostart directory.

## Troubleshooting

Check these common issues, in order of likelihood. If one step doesn't correct the problem, move on to the next. Remember to restart the GNOME session by logging in and out to test if it worked.

1. Credentials must be in the keyring. Mount manually using the Files app. If it prompts you for a login, save the login. That should put it on the keyring.
2. Open up a terminal and run *gio*. If that command is not there then *glib2* is missing.
3. Check the URL of the server. Running *gio mount $SHARE_URL_HERE* should mount the share if it's valid.
4. Not all systems use the default autostart directory. Running *systemd-path user-configuration* or *echo $XDG_CONFIG_HOME* should print where the XDG configuration folder is. If it does, create the 'autostart' folder there and move the desktop file. If it doesn't, your system might not support XDG Autostart. See [this ArchWiki article][xdg-archwiki] for more.

## Acknowledgements

Special thanks to [Sebastian Keller][skeller] on the GNOME Matrix for [pointing this method][skeller-matrix] out to me. Somehow, this flew under the radar. Most results from searching online turn up complicated instructions for Samba. But, GNOME doesn't need Samba to connect to remote servers, it uses [Gvfs][gvfs] instead.

[gvfs]: https://wiki.gnome.org/Projects/gvfs
[gvfs-protos]: https://wiki.gnome.org/Projects/gvfs/schemes
[xdg-archwiki]: https://wiki.archlinux.org/title/XDG_Autostart
[skeller]: mailto:skeller@gnome.org
[skeller-matrix]: https://matrix.to/#/!EzCPArJyrSebvklKjn:gnome.org/$tOPrUS-mnrVF87e4EpIX-G0fp7cupkUKS3Yok-lj8xY?via=libera.chat&via=matrix.org&via=gnome.org
[related_gnome_issue]: https://gitlab.gnome.org/GNOME/nautilus/-/issues/120

