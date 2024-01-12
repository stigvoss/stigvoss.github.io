---
layout: post
title:  "Improving the Bambu Studio Update Script for Linux"
date:   2023-10-30 20:50:00 +0100
categories: fedora bambu-studio
---

**UPDATE**: Bambu Studio has now been released as a [Flatpak](https://flathub.org/apps/com.bambulab.BambuStudio) and if you run an operating system supporting Flatpak, I would highly suggest using that instead. Huge thanks to [hadess](https://github.com/hadess) and may I suggest [thanking him through his wishlist](https://github.com/flathub/com.bambulab.BambuStudio#saying-thanks).

Earlier, I wrote about [updating Bambu Studio on Fedora Linux](https://stigvoss.dk/2023/10/30/updating-bambu-studio-fedora/).
I have since then made a few improvements, such as checking if a newer version exists.

```bash
#!/bin/bash  

set -euo pipefail
IFS=$'\n\t'

URL=https://api.github.com/repos/bambulab/BambuStudio/releases
DISTRO=Fedora # or ubuntu (ubuntu is lowercase on the release page)
OUTPUT=./BambuStudio.AppImage
VERSION_FILE="$OUTPUT.ver"

RELEASES=$(curl -s $URL)
LATEST=$(jq -r .[].tag_name <<< $RELEASES | sort -Vr | head -n 1)

[[ -e $VERSION_FILE ]] && CURRENT=$(cat $VERSION_FILE) || CURRENT=""

if [[ $CURRENT != $LATEST ]]; then
	DOWNLOAD_URL=$(jq -r ".[] | select(.tag_name == \"$LATEST\") | .assets[] | select(.name | contains(\"$DISTRO\")) | .browser_download_url" <<< $RELEASES)
	wget -O $OUTPUT $DOWNLOAD_URL
	echo $LATEST > $VERSION_FILE
	
	echo "Bambu Studio is updated to $LATEST."
else
	echo "Bambu Studio is already up-to-date."
fi
```