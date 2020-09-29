---
title: Set Partmgr Attributes registry with PowerShell
description: Describes how to set the Partmgr Attributes registry value by using PowerShell.
ms.date: 09/16/2020
author: Deland-Han
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: Data corruption and disk errors
ms.technology: BackupStorage
---
# How to set the Partmgr Attributes registry value using PowerShell

This article describes how to set the Partmgr Attributes registry value by using PowerShell.

_Original product version:_ &nbsp; Windows Server 2012 R2  
_Original KB number:_ &nbsp; 2849097

## Summary

The partmgr.sys Attributes registry key affects the onlining behavior of a given disk, and the value corresponds to the SAN policy values in VDS as documented on [VDS_SAN_POLICY enumeration](/windows/win32/api/vds/ne-vds-vds_san_policy). 

```cpp
typedef enum _VDS_SAN_POLICY {
    VDS_SP_UNKNOWN = 0x0,
    VDS_SP_ONLINE = 0x1,
    VDS_SP_OFFLINE_SHARED = 0x2,
    VDS_SP_OFFLINE = 0x3
} VDS_SAN_POLICY;
```

This value is initially set when a disk is identified based on the SAN Policy value in place at the time. The SAN policy determines whether a newly discovered disk is brought online or remains offline, and whether it is made read/write or remains read-only. When a disk is offline, the disk layout can be read, but no volume devices are surfaced through Plug and Play (PnP). This means that no file system can be mounted on the disk. When a disk is online, one or more volume devices are installed for the disk.

This policy can be viewed using the DISKPART utility as follows:

1. Click Start, click Run, type cmd, and then press ENTER.
2. In the command-line window, type diskpart, and then press ENTER.
3. Type SAN, and then press ENTER. This command will return the current set SAN policy.
4. Type exit, and then press ENTER.

Setting a value of 0 (VDS_SP_UNKNOWN) will be mapped to VDS_SP_ONLINE, so a value of 0 or 1 may be used to bring a disk online.

This script modifies the Attributes registry value located at `HKLM\System\CurrentControlSet\Enum\SCSI\<device>\<instance>\Device Parameters\Partmgr`.

For more information about how to write scripts for Windows PowerShell, visit the following Microsoft website:

[General information about how to write scripts for Windows PowerShell](https://technet.microsoft.com/scriptcenter/dd742419) 

Registry information 

> [!IMPORTANT]
> This section, method, or task contains steps that tell you how to modify the registry. However, serious problems might occur if you modify the registry incorrectly. Therefore, make sure that you follow these steps carefully. For added protection, back up the registry before you modify it. Then, you can restore the registry if a problem occurs. For more information about how to back up and restore the registry, click the following article number to view the article in the Microsoft Knowledge Base:
 [322756](https://support.microsoft.com/help/322756)  How to back up and restore the registry in Windows

The vendor string variable is used to restrict the value change to a subset of disks matching a specific vendor's identifier. Check the registry for the Disk&Ven_ string to verify what value should be used.

```powershell
$val = 0
$vendor = Read-Host "Enter Vendor String"
$devIDs = Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Enum\SCSI\Disk*Ven_$vendor*\*\Device Parameters\"

foreach ($id in $devIDs)
{
    $error.Clear()
    $regpath = $id.PSPath + "\Partmgr\"
    Set-ItemProperty -path $regpath -Name Attributes -Value $val -ErrorAction SilentlyContinue

    if ($error) # didn't find the path, create it and try again
    {
        New-Item -Path $id.PSPath -Name Partmgr
        Set-ItemProperty -path $regpath -Name Attributes -Value $val -ErrorAction SilentlyContinue
        $error.Clear()
    }

    Get-ItemProperty -Path $regpath -Name Attributes -ErrorAction SilentlyContinue | Select Attributes | fl | Out-String -Stream
}
```