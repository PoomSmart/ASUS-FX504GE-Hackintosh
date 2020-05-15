## Discontinuation
The laptop has been sold. The PR owner bought a MacBook Pro and has no intention to update this anymore.

# ASUS-FX504GE-Hackintosh
Necessary configurations and instructions to get [ASUS TUF FX504GE laptop](https://www.ultrabookreview.com/19725-asus-tuf-fx504ge-review/) working with macOS Mojave 10.14.x and Catalina 10.15.x. High Sierra (10.13.x) can work on these laptops too, but Mojave or higher is strongly recommended. The following should also work with all ASUS FX504G.. laptop variants.

# Notes
1. 128 GB NVMe SSD is used for installing macOS
2. APFS partition format has to be used
3. If you are upgrading from the previous version and your partition is HFS+, better boot the installer, unmount the partition and convert it to APFS

# macOS Catalina (10.15) Kexts Requirements
1. Lilu 1.3.7+
2. WhateverGreen 1.3.0+
3. AppleALC 1.3.9+

# BIOS Settings
1. Secure Boot: Disabled
2. SATA mode: AHCI
3. DVMT-Preallocated: 64MB

## Guide for Clover users (DISCONTINUED)

[Here](Clover/README.md)

## Guide for OpenCore users

[Here](OpenCore/README.md)
