---
layout: post
title:  "Bambu Studio on Fedora Silverblue"
date:   2023-10-30 20:15:00 +0100
categories: fedora fedora-silverblue bambu-studio
---

Bambu Studio for Linux is only distributed as an AppImage, not as a Flatpak and when running on Fedora, Bambu Studio requires extra dependencies to be installed.

This fact makes it challenging to run on an immutable operating system like Fedora Silverblue.

To workaround this challenge, tools like Silverblue's built-in Toolbox or [Distrobox](https://github.com/89luca89/distrobox) can be used.

The following is a `distrobox.ini` for Distrobox Assemble which will allow you to run Bambu Studio's AppImage in a container.

```ini
[bambulabs]
image=fedora-toolbox:38
additional_packages="fuse fuse-libs"
additional_packages="mesa-libGL mesa-libGLU"
additional_packages="wayland-devel wayland-protocols-devel libxkbcommon"
additional_packages="gtk3 webkit2gtk3"
additional_packages="gstreamer1 gstreamer1-plugins-base gstreamer1-plugin-openh264"
additional_packages="mesa-libOSMesa"
```

I will suggest creating a `BambuStudio.desktop` in your `~/.local/share/applications` to launch the program from your host OS.

```
[Desktop Entry]
Name=Bambu Studio
GenericName=Bambu Studio
Comment=Bambu Labs Slicer
Categories=Utility
Exec=/usr/bin/distrobox enter  bambulabs -- ~/Applications/BambuStudio.AppImage
Icon=bambustudio
Terminal=false
NoDisplay=true
Type=Application
```

This example assumes your Bambu Studio AppImage is located as ~/Applications/BambuStudio.AppImage.