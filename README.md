# Acer Swift 3/5 IceLake Hackintosh (OpenCore)

## Support Version
- OpenCore: `0.7.9`
- MacOS: `Big Sur 11.6.5（20G527)`

## Tested Model Spec

> All Acer Swift 3/5 IceLake laptop can possibly use the same EFI file, but we suggest to test by yourself.


- ### **Acer Swift 5 SF514-54GT**
    |  Specifications   |              Detail               |
    |-------------------|-----------------------------------|
    |Processor          |i7-1065G7                          |
    |Integrated Graphics|Intel Iris Plus Graphics           |
    |Graphic Card       |NVIDIA GeForce MX350 2G            |
    |Memory             |16GB<sup>8Gx2</sup> Sk Hynix LPDDR4X 2667MHz|
    |Sound Card         |Conexant CX20671                   |
    |Wireless Card      |AX201                              |

    
- ### **Acer Swift 3 SF314-57G**
    |  Specifications   |              Detail               |
    |-------------------|-----------------------------------|
    |Processor          |i5-1035G1                          |
    |Integrated Graphics|Intel UHD Graphics                 |
    |Graphic Card       |NVIDIA GeForce MX350 2G            |
    |Memory             |8GB Sk Hynix LPDDR4X 2667MHz       |
    |Sound Card         |Conexant CX20671                   |
    |Wireless Card      |AX201                              |
    

## What is Working?
- CPU power management
- Hardware acceleration
- Sleep/Wake
- Battery read-out
- Audio (Internal microphone, 3.5mm headphone jack) <sup>**Internal speaker is not working**</sup>
- Keyboard & trackpad/touchscreen with all macOS gestures
- Wi-Fi & Bluetooth
- USB ports
- ThunderBolt 3 <sup>**Not test yet, but the Type-C port is fine.**</sup>

## What is Not Working?
- Airdrop <sup>**Not Supported in MacOS when using Intel Wireless Card**</sup>
- FingerPrint <sup>**Not Supported in MacOS**</sup>
- NVIDIA GeForce MX350 <sup>**Disable in ACPI**</sup>

---
---
# Guidance
## Getting started with [OpenCore](https://dortania.github.io/OpenCore-Install-Guide/)

> Essentially follow the [**Laptop Icelake | OpenCore Install Guide**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html#starting-point) and incorporate the insights listed below.


## BIOS

> We accept no liability for any loss or damage howsoever changing BIOS with this guidance and cause damage on your device. Please be careful and make sure you know what you are doing.


- Due to some BIOS setting options are hidden by **Acer**, we can use several tools to help us matching [**Intel BIOS Settings | OpenCore Install Guide**](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html#intel-bios-settings), here are the steps below.

***
**Info**

Now with all that, here are the tools we need
> All tools are in [here](https://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1wrPMd3yAIhplg8v0mZhzhQ).<sup>(Password is 1234)</sup>

- [**Universal BIOS Backup ToolKit**](https://m.majorgeeks.com/mg/getmirror/universal_bios_backup_toolkit,1.html)
    - For Dumping BIOS rom
- **InsydeH2OUVE**
    - For Setting up BIOS
- [**UEFITool_NE**](https://github.com/LongSoft/UEFITool/releases)
    - For extracting and modifying UEFI firmware images
- **Universal IFR Extractor** ([**Windows**](http://bios-mods.com/pub/donovan6000/Software/Universal%20IFR%20Extractor/Universal%20IFR%20Extractor.exe) | [**MacOS**](https://github.com/LongSoft/Universal-IFR-Extractor/releases/tag/0.3.7))
    - Utility to extract the internal forms representation from both EFI and UEFI drivers/applications into human readable text file.

The details of the guildance please follow [**this**](https://zhuanlan.zhihu.com/p/266400995), which was written in Chinese, but there had some images to tell you how to do.

### **I. Dump BIOS Rom**
- Run **Universal BIOS Backup ToolKit** as administrator and dump the BIOS rom.

### **II. Extracting UEFI firmware images**
- Use **UEFITool_NE** to extract UEFI firmware files, and turn it into txt file by using **Universal IFR Extractor**.

### **III. Find the specific settings in txt file.**
- For example, you will find the information like this:
    |       Name       |Var Store|Variable|Default|Value|  Description   |
    |------------------|---------|--------|-------|-----|----------------|
    |DVMT Pre-Allocated|0x2      |0x107   |0x1    |0x2  |DVMT 32M ➜ 64M  |
    |CFG Lock          |0x3      |0x3E    |0x1    |0x0  |Disable CFG Lock|
    
    And then,
    
    ```txt
    Var Store: 0x2[555] (SaSetup) {...}
    Var Store: 0x3[570] (CpuSetup) {...}
    ```
    
### **IV. Change BIOS settings.**
- According to the information in **Step III** to fix the value in BIOS by using **InsydeH2OUVE**.

    > - You need to run **WDFInst.exe** first and them open the **InsydeH2OUVE**.
    > - Set `Low Power S0 Idle Capability` to `Disable` in BIOS or you will get a sleep problem with it.(MacOS only support `S3 Sleep State`)
---
---
# Specific Patch

## ACPI
### Add
- **SSDT-dGPU-Off**
    - More details in [**Disabling laptop dGPUs | Getting Started With ACPI**](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/laptop-disable.html#disabling-laptop-dgpus-ssdt-dgpu-off-nohybgfx)

## Booter
### MmioWhitelist
- **MMIO IceLake**
    - This will cause early kernel panics on Acer IceLake Laptop.

        |Comment|String |MMIO 0xFF600000 Icelake|
        |-------|-------|-----------------------|
        |Enabled|Boolean|True                   |
        |Address|Number |4284481536             |

## DeviceProperties
### Add
- **PciRoot(0x0)/Pci(0x2,0x0)**
   - [**Support all possible Core Display Clock (CDCLK) frequencies on ICL platforms**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#support-all-possible-core-display-clock-cdclk-frequencies-on-icl-platforms)
   - [**Fix the issue that the builtin display remains garbled after the system boots on ICL platforms**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#fix-the-issue-that-the-builtin-display-remains-garbled-after-the-system-boots-on-icl-platforms)
   - More details in [**Intel Iris Plus Graphics (Ice Lake processors) | WhateverGreen (Intel® HD Graphics FAQs)**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#intel-iris-plus-graphics-ice-lake-processors)

## NVRAM
### Add
- 7C436110-AB2A-4BBB-A880-FE41995C9F82
    - boot-args
        - `-noDC9` 	
            - This will fix sleep problem
        - `-igfxdbeo` `-igfxcdc`
            - Ignore when you place in **DeviceProperties**

---
---
# Credits
- [**Apple**](https://www.apple.com/tw/) for the macOS.
- [**Acidanthera**](https://github.com/acidanthera) for awesome kexts and first-class support for hackintosh enthusiasts.
- [**Dortania**](https://github.com/dortania) for the great guides.
- [**mfpss95134**](https://github.com/mfpss95134) for fixing bugs in Acer Swift Laptop.
- [**了了**](https://www.zhihu.com/people/xiao-zu-5-49) for setting up the BIOS.

