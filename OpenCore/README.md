# OpenCore notes
Configurations here are considered experimental. Use at your own risk.

Tested version: 0.5.4

# Configurations

## ACPI
### Add
1. `DSDT.aml` (Patched one)
2. `SSDT-DDGPU.aml` (Disable discrete graphics card)
3. `SSDT-PLUG.aml` (Alternative for Clover's `PluginType`)
4. `SSDT-PNLF.aml` (Backlight support)
5. `SSDT-PNLFCFL.aml` (Backlight support for Coffee Lake machines)
6. `SSDT-EC.aml` (Override AppleBusPowerController profile for Catalina beta 5)
7. `SSDT-UIAC.aml` (Disable unused USB ports)
8. `SSDT-XOSI.aml` (TBD)
9. `SSDT-XHC.aml` (USB3)

### Patch
1. `Rename EC0 to EC` (Really need it?)
2. `Rename _OSI to XOSI`
3. `Rename HDAS to HDEF` (Audio)

### Quirks
All `NO`.

## Booter
### Quirks
Enabled:
1. `AvoidRuntimeDefrag`
2. `DevirtualiseMmio`
3. `DisableVariableWrite` (NVRAM)
4. `SetupVirtualMap`
5. `ShrinkMemoryMap`

## DeviceProperties
### Add
Only the important configurations are shown here:
1. `PciRoot(0x0)/Pci(0x1f,0x3)` (Audio - ALC255)
   1. `layout-id` as `0x03000000` (which is 3)
2. `PciRoot(0x0)/Pci(0x2,0x0)` (Graphics - Intel UHD 630)
   1. `AAPL,ig-platform-id` as `0x00009b3e` (which is 0x3E9B0000)
   2. `device-id` as `0x9b3e0000` (which is 0x00003E9B)
   3. `disable-external-gpu` as `0x01000000` (which is 1, disabling discrete graphics card)
   4. `framebuffer-unifiedmem` as `0x00000080`, `framebuffer-stolenmem` as `0x00003001` and `framebuffer-fbmem` as `0x00009000` (increase VRAM from 1536 MB to 2048 MB)

## Kernel
### Add
Orders matter. Think about which kexts should load before which.
1. **Lilu** (First of all)
2. **VirtualSMC** (Second always)
3. **CPUFriend** (CPU power management)
4. **CPUFriendDataProvider** (CPU power management profile)
5. SMCProcessor
6. SMCSuperIO (maybe not)
7. SMCLightSensor (maybe not)
8. **SMCBatteryManager**
9. **WhateverGreen** (Graphics)
10. **AppleALC** (Audio)
11. NoTouchID
12. **VoodooPS2Controller** (PS/2 keyboard) (Must also be installed at `/Library/Extensions`)
13. **USBInjectAll** (USB ports, built-in webcam)
14. RealtekRTL8111 (LAN internet)
15. **XHCI-unsupported** (USB3)
16. SATA-300-series-unsupported (SATA drives)

### Quirks
1. `AppleCpuPmCfgLock` enabled (maybe not)
2. `AppleXcpmCfgLock` enabled
3. `DisableIoMapper` **enabled** (Disable VT-d)
4. `ThirdPartyDrives` **enabled** (TRIM for SSD)
5. `XhciPortLimit` **disabled** (no need)

## Misc
### Boot
1. `ConsoleBehaviourOs` as `ForceGraphics`
2. `ConsoleBehaviourUi` as `ForceText`
3. `HibernateMode` as `None`
4. `Resolution` as `1920x1080` (Or your actual screen size)
5. `ShowPicker` and `UsePicker` enabled

### Security
1. `AllowNvramReset` enabled (If you want to)
2. `ExposeSensitiveData` as `3` or `4`

## NVRAM
### Add
1. `7C436110-AB2A-4BBB-A880-FE41995C9F82`
   1. `boot-args` containing:
      1. `darkwake=0` (Sleep/wake)
      2. `-lilubetaall -vsmcbeta -alcbeta -wegbeta` (If you are on a newer beta)
      3. `-v debug=0x100 keepsyms=1` (Debugging, optional)
      4. `-no_compat_check` (In case OS doesn't support your machine)
      5. `-igfxmlr enable-dpcd-max-link-rate-fix` (Read WhateverGreen README)
   2. `LegacyEnable` enabled (Emulating NVRAM)

## PlatformInfo
1. `Automatic` **enabled**
2. `Update*` options **enabled**

### Generic
This is where you put SMBIOS data, make sure you provide the following:
1. `MLB`
2. `ROM` (MAC address)
3. `SpoofVendor` **enabled**
4. `SystemProductName` as `MacBookPro15,2` or similar
5. `SystemSerialNumber`
6. `SystemUUID`
**These values are masked from the provided config file, make sure you enter your own before testing!**

## UEFI
1. `ConnectDrivers` **enabled**

### Drivers (must-have)
1. `UsbKbDxe.efi`
2. `VboxHfs.efi`
3. `ApfsDriverLoader.efi`
4. `FwRuntimeServices.efi` (UEFI + AptioMemoryFix)
5. `VirtualSMC.efi`
6. `NvmExpressDxe.efi` (NVMe SSD)
7. `XhciDxe.efi` (Peripherals)

### Input
1. `KeySupport` disabled (already use `UsbKbDxe.efi`)

### Protocols
1. `ConsoleControl` enabled

### Quirks
1. `ClearScreenOnModeSwitch` enabled
2. `ProvideConsoleGop` enabled
3. `RequestBootVarRouting` enabled
4. `SanitiseClearScreen` enabled

![Screenshot](FX504GE-SS-OC.png?raw=true)