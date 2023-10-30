---
layout: post
title:  "Update Bambu Studio on Fedora Linux"
date:   2023-10-30 20:50:00 +0100
categories: fedora bambu-studio
---

To avoid having to visit GitHub to download new versions of Bambu Studio for Linux, use the below script to download and update your AppImage.

```bash
#!/bin/bash

URL=https://api.github.com/repos/bambulab/BambuStudio/releases
DISTRO=Fedora # or ubuntu (ubuntu is lowercase on the release page)
OUTPUT=./BambuStudio.AppImage

wget -O $OUTPUT $(curl -s $URL | jq -r ".[0].assets[] | select(.name | contains (\"$DISTRO\")) | .browser_download_url")

chmod +x $OUTPUT
```