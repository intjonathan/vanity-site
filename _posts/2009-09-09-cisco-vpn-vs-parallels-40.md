---
title: "Cisco VPN vs. Parallels 4.0"
date: 2009-09-09
author: Jonathan Owens
---

If you're using Cisco VPNClient on OS X 10.5, and you have Parallels 4.0 installed, you may be treated to the following error when attempting to start a VPN connection over ppp - a typical use case for an on-call developer using a 3G modem like the USBConnect Mercury:

```text
305 10:28:16.787 05/22/2008 Sev=Warning/2 CVPND/0x83400011
Error -28 sending packet.
...
Output size mismatch. Actual: 0, Expected: 237. (DRVIFACE:1319)
```

The fix for this is to uninstall Parallels and buy VMWare fusion instead because Parallels is slow and it sucks. If there's a workaround I don't have the patience to find it.
