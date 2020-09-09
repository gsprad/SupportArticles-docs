---
title: Validate the OEM activation key
description: Describes how to validate the OEM activation key in Windows 10, version 1703 and later versions.
ms.data: 09/08/2020
author: Deland-Han
ms.author: Deland-Han
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-client
localization_priority: medium
ms.reviewer: kaushika, themar
ms.prod-support-area-path: Activation
ms.technology: Deployment
---
# How to validate the OEM activation key in Windows 10

This article introduces how to validate the OEM activation key in Windows 10.

_Original product version:_ &nbsp; Windows 10 - all editions  
_Original KB number:_ &nbsp; 4346763

## Background

Starting at Windows 10 Creators Update (build 1703), Windows activation behavior has changed so that the unique OA3 Digital Product Key (DPK) is not always presented as the currently installed key in the device. Instead, the system behaves as follows:

- Windows 10 (including all versions starting at Windows 10 Creators Update) is deployed to a device by having the appropriate default product key. You can run `slmgr /dli` or `slmgr /dlv` to show the partial default product key instead of the OA3 DPK as the current license in the firmware. The product ID that is displayed on the **Settings** > **System** > **About** page is not unique for the Windows 10 key that is being used.

- A device that is running any Windows 10 OEM client edition, such as Windows Home or Windows Professional, and is activated by using the OA3 DPK in the firmware is upgraded to a newer version of Windows (for example, build 1703 to build 1709). However, sometimes running `slmgr /dli` or `slmgr /dlv` does not show the OA3 DPK as the current license. Instead, these commands show the default product key.

The behavior that is described here is by design. The activation and user experience are not affected. However, OA validation in the factory may be affected as follows:

- The output of the `slmgr /dlv` or `slmgr /dli` command is not necessarily the last five (5) digits of the injected DPK. Therefore, you can no longer rely upon these commands to return the expected results.

## Recommendations for validating the product ID against the product key ID of OA3 DPK

Every OEM has a different manufacturing process that they have adopted through years of experience in manufacturing Windows. Specifically, to validate the DPK against the installed Windows 10 edition, we recommend that you not rely on the output of `slmgr /dlv` or `slmgr/dli` but, instead, use the latest OA3Tool as follows:

- OA3TOOL /Validate

  Runs a validation pass to make sure that the MSDM table exists, the MSDM table header includes all the required fields, and the MSDM table entries exist and comply with the correct formats.
- OA3TOOL /CheckEdition

  Does a cross-check between the injected DPK and the target Windows edition if they match.

## Can Microsoft ensure that the system will always activate if I do the recommended steps

The Windows activation system is designed to use the product key that is injected into the firmware of the computer during manufacturing. It automatically activates the device when the device first comes online. This operation is used daily on thousands of devices. As an additional check, OEMs are encouraged to run the complete end-to-end validation process, including activation on a subset of the devices, to validate the user experience with their PCs. If you experience any issues, engage with us through the usual channels.

## Why did Microsoft remove the ability to check the last five digits of the product by using slmgr

SLMGR is a legacy tool. Although we have not updated slmgr, and because of updates in successive system builds, the last five digits of the product key that are shown by `slmgr /dlv` or `/dli` do not match the product key that is injected into the system BIOS. This is by design. We have no intentions of validating SLMGR for every Windows 10 release or making any other changes. We are very open to feedback regarding the OA3 tool and what additional capabilities we can add to it to improve the manufacturing flow.