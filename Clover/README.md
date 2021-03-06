# Clover notes
1. Clover bootloader r4954+ for macOS Catalina (10.15)
2. For Clover v2.5k, **the new UEFI drivers directory is now** `/EFI/CLOVER/drivers/UEFI`, make sure you move every UEFI driver from the old (Clover v2.4k) directory (`/EFI/CLOVER/drivers64UEFI`) to this directory after you upgraded Clover, **except for HFSPlus-64.efi** as there has been a new version of it that works with Clover v2.5k and should come pre-installed by default by the Clover v2.5k installer.

# Pre-installation
Get yourself a Mojave/Catalina USB installer with Clover installed. Important Clover settings (via Clover Configurator) are:
1. Acpi SSDT `PluginType` checked
2. Graphics: Inject Intel checked
3. Kernel Patches `Kernel LAPIC`, `KernelPM` and `AppleRTC` enabled
4. SMBIOS: MacBookPro15,2
5. UEFI Drivers
    1. **EmuVariableUefi-64** (Native UEFI won't work with macOS)
    1. **ApfsDriverLoader-64** (If your OS partition is APFS)
    1. **AptioMemoryFix-64.efi** (Required for ASUS BIOS, APTIO V)
    1. **PartitionDxe-64**
    1. CsmVideoDxe64
    1. UsbKbDxe-64
    1. UsbMouseDxe-64
    1. NvmExpressDxe-64.efi (If you use NVMe SSD)
    1. HFSPlus-64 (If you have HFS+ partitions) - **If you use Clover v2.5k, it should be named as just HFSPlus.efi**
    1. NTFS-64 (If you have NTFS partitions)
 
Mandatory kexts installed to `/EFI/CLOVER/kexts/Other`: **FakeSMC**, **VoodooPS2Controller**, **Lilu**

# Post Configuration
Any changes made to kexts installed under `/Library/Extensions` are to be followed by `sudo kextcache -i /`.
Every external kext mentioned is assumed to be the latest.
## First Steps
1. DSDT files generated by Clover to the EFI partition
2. DSDT tables to drop: `MATS` and `DMAR`
3. Drop all _DSM methods
## CPU Power Management
1. Clover ACPI `PluginType` enabled
2. Clover Kernel Patches `Kernel LAPIC`, `KernelPM` and `AppleRTC` enabled
3. ACPIBatteryManager kext installed to `/EFI/CLOVER/kexts/Other`
## ALC255 Realtek Audio
Internal speaker and microphone work. If headphone output produces weird audio, set volume balance to be either left or right. This issue has not been seen on Catalina.
1. `/System/Library/Extensions/AppleGFXHDA.kext` **must be removed** (ID matched but not actually compatible)
2. AppleALC kext installed to `/EFI/CLOVER/kexts/Other`
3. Clover Audio injection `Inject=3` (`ResetHDA` may be enabled)
## PS/2 Keyboard
**Custom-built** [VoodooPS2Controller](https://github.com/acidanthera/VoodooPS2) kext installed to `/Library/Extensions` and `/EFI/CLOVER/kexts/Other` (using keyboard in Recovery mode)

This kext has its own CAPS lock workaround, plus setting `Swap command and option` value to `false` inside `VoodooPS2Keyboard` 's `Info.plist`, you can grab it from this repository.

```
// Simplified code
// CAPS lock may not work properly with Keyboard Viewer active, might be improved in the future
if (adbKeyCode == 0x39 && version_major >= 16) // Caps Lock workaround
{
    clock_get_uptime(&now_abs);
    dispatchKeyboardEventX(adbKeyCode, goingDown, now_abs);
    return true;
}
```

## Intel UHD 630 Graphics
1. Enabled by `device-properties` injection (`PciRoot(0)/Pci(0x02,0)`) with Clover's `Inject Intel` option **unchecked**
2. WhateverGreen kext installed to `/EFI/CLOVER/kexts/Other`
3. Useful additional boot flags: `-igfxmlr enable-dpcd-max-link-rate-fix `
## Sleep and Wake
1. Applied static patching of `USB _PRW 0x6D (instant wake)` **for Skylake** (credits to [MegaStood](https://github.com/MegaStood/Hackintosh-FX504GE-ES72))
2. Patch `_PRW` method for `XDCI` device as follows:
```
Device (XDCI)
{
    ...
    Method (_PRW, 0, NotSerialized)  // _PRW: Power Resources for Wake
    {
        Return (Package (0x02)
        {
            0x6D, 
            Zero
        })
    }
    ...
}
```
3. **Remove** `_PRW` method from `CNVW` device
### Backlight Control
Via WhateverGreen. Remove AppleBacklightFixup if present.
1. Latest `SSDT-PNLF.aml` and `SSDT-PNLFCFL.aml` installed to `/EFI/CLOVER/ACPI/patched`
2. Brightness adjustment keys working by modifying `/EFI/CLOVER/ACPI/patched/DSDT.aml`
   ```
   Scope (_SB.PCI0.LPCB.EC0) {
        ...
        Method (_Q11, 0, NotSerialized)  // _Qxx: EC Query
        {
            Notify (PS2K, 0x0405) // Brightness down
        }
        Method (_Q12, 0, NotSerialized)  // _Qxx: EC Query
        {
            Notify (PS2K, 0x0406) // Brightness up
        }
        ...
   }
   ```
### HDMI Port + Audio
No thorough test on this.
1. Disable WhateverGreen's HDMI injection by adding a boot flag `-igfxnohdmi`
2. `device-properties` combination of `framebuffer-con1-type`, `framebuffer-con1-pipe` and `AAPL01,override-no-connect` based on [this post](https://www.tonymacx86.com/threads/uhd-630-no-hdmi-audio.265490/page-2#post-1858289)
## USB 2.0/3.1 Ports
1. Clover USB injection `Inject=false`
2. USBInjectAll, XHCI-300-series-injector and XHCI-unsupported kexts installed to `/EFI/CLOVER/kexts/Other`
3. `SSDT-XHC.aml` installed to `/EFI/CLOVER/ACPI/patched` for better USB support
4. Disable unused USB ports via `/EFI/CLOVER/APCI/patched/SSDT-UIAC.aml`
5. Override AppleBusPowerController profile (and fix **bootloop on Catalina beta 5**) via `/EFI/CLOVER/ACPI/patched/SSDT-EC.aml`
## Realtek LAN
1. [RealtekRTL8111](https://www.insanelymac.com/forum/topic/287161-new-driver-for-realtek-rtl8111/) kext installed to `/Library/Extensions` and `/EFI/CLOVER/kexts/Other` (using internet in Recovery mode)
## SATA controller
1. SATA-300-series-unsupported kext installed to `/EFI/CLOVER/kexts/Other`
## I2C ELAN1200 Precision TouchPad (`pci8086,a368`)
### ** Caution: Latest unofficial VoodooI2C may cause kernel panic whenever patched DSDT.aml is used, there's no way around that yet **
1. VoodooI2C kexts [version 2.2](https://github.com/alexandred/VoodooI2C) or later (VoodooI2C + VoodooI2CHID)
2. DSDT patch: **\[Windows\] Windows 10 Patch**
3. Patch `TPD0` inside `/EFI/CLOVER/ACPI/patched/DSDT.aml` as follows:
```
Device (TPD0)
{
    ...

    Name (SBFG, ResourceTemplate ()
    {
        GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,
            "\\_SB.PCI0.GPI0", 0x00, ResourceConsumer, ,
            )
            {   // Pin list
                0x0000
            }
    })

    ...

    Method (_CRS, 0, NotSerialized)  // _CRS: Current Resource Settings
    {
        Return (ConcatenateResTemplate (SBFB, SBFG)) // ** MODIFIED **
    }
```

Trackpad works okay but with minor stuttering.

## Miscellaneous
1. NoTouchID kext installed to `/EFI/CLOVER/kexts/Other` (MacBookPro15,2 has Touch ID)

# Things that do not work
## NVIDIA Geforce 1050 Ti (Optimus)
Discrete graphic, we probably never see the day. For now, use `SSDT-DDGPU.aml` (in `/EFI/CLOVER/ACPI/patched`) to power it off.
## Intel Wi-Fi AC 9560
Intel built-in Wi-Fi chipset, we again probably never see the day.
## Intel Bluetooth
It never works if Wi-Fi doesn't work.

![Screenshot](FX504GE-SS.png?raw=true)
