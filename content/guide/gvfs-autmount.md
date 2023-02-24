---
title: "Automatically mounting remote shares in GNOME"
date: 2023-02-24T15:36:21-05:00
---

# Automatically mounting remote shares in GNOME

When moving between many devices, or share files in a large group or organization, it's annoying to find and open remote shares in GNOME. Using built-in GNOME tools like *gio*, it's possible to automatically mount network shares when you log in **without** wrangling with Samba or fstab. It works for [many protocols][gvfs-protos], including *smb*, *webdav*, and *ftp*.
<!--more-->

## Requirements

This guide works with the GNOME Shell, and tested on GNOME 43 on Fedora 37. If *glib2* is installed and *gio* is available, this guide should work on most freedesktop compliant environments. Before continuing, ensure that any credentials for the network share are **saved on the GNOME Keyring**.

## Getting Started

To mount the remote location on login, create a **desktop entry** in the autostart directory. The autostart directory is usually in *~/.config/autostart*. If the folder doesn't exist, create it. Give your desktop file a unique filename, such as *mount-example-share.desktop*. The content should look like the example below, with *$SHARE_URL_HERE* replaced with the URL to the share.

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

On the next login, the remote share should be available in the Files app. To add more automatically mounted shares, create more desktop files in the autostart directory.

## Troubleshooting

Check these common issues, in order of likelihood. If one step doesn't correct the problem, move on to the next. Remember to restart the GNOME session by logging in and out to test if it worked.

1. The credentials must be in the keyring. Connect manually using the Files app. If it prompts you for a login, they aren't in the keyring.
2. Open up a terminal and type *whatis gio*. If it's nothing appropriate, *glib2* is missing. See the [requirements](#requirements) for this guide.
3. Check the URL of the server. Ensure running *gio mount $SHARE_URL_HERE* mounts the share.
4. Not all systems use the default autostart directory. Running *systemd-path user-configuration* or *echo $XDG_CONFIG_HOME* should print where the XDG configuration folder is. If it does, create the 'autostart' folder there and move the desktop file. If it doesn't, your system might not support XDG Autostart. See [this ArchWiki article][xdg-archwiki] for more.

## Acknowledgements

Special thanks to [Sebastian Keller][skeller] on the GNOME Matrix for [pointing this method][skeller-matrix] out to me. Somehow, this flew under the radar. Most results from searching online turn up complicated instructions for Samba. However, GNOME doesn't need Samba to connect to remote servers, it uses [Gvfs][gvfs] instead.

[gvfs]: https://wiki.gnome.org/Projects/gvfs
[gvfs-protos]: https://wiki.gnome.org/Projects/gvfs/schemes
[xdg-archwiki]: https://wiki.archlinux.org/title/XDG_Autostart
[skeller]: mailto:skeller@gnome.org
[skeller-matrix]: https://matrix.to/#/!EzCPArJyrSebvklKjn:gnome.org/$tOPrUS-mnrVF87e4EpIX-G0fp7cupkUKS3Yok-lj8xY?via=libera.chat&via=matrix.org&via=gnome.org
