---
title: Measure-Object not found error with Enter-PSSession cmdlet
description: Enter-PSSession unexpectedly terminates when a network path is specified in PSModulePath.
ms.date: 09/21/2020
author: Deland-Han
ms.author: delhan 
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika, Gbrag, jerrycif
ms.prod-support-area-path: PowerShell
ms.technology: SysManagementComponents
---
# Enter-PSSession cmdlet fails when network path specified in PSModulePath environment variable

This article provides a resolution for the issue that Enter-PSSession unexpectedly terminates when a network path is specified in PSModulePath.

_Original product version:_ &nbsp; Windows Server 2016, Windows Server 2012 R2  
_Original KB number:_ &nbsp; 4076842

## Symptom

When a network path is specified in the **PSModulePath** environment variable, the Enter-PSSession cmdlet fails, and you receive the following error message:  
>Enter-PSSession : The 'Measure-Object' command was found in the module 'Microsoft.PowerShell.Utility', but the module 
could not be loaded. For more information, run 'Import-Module Microsoft.PowerShell.Utility'.  
At line:1 char:1  
\+ Enter-PSSessionservername  
\+ ~~~~~~~~~~~~~~~~~~~~~~~~  
  + CategoryInfo : ObjectNotFound: (Measure-Object:String) [Enter-PSSession], CommandNotFoundException  
  + FullyQualifiedErrorId : CouldNotAutoloadMatchingModule

## Cause

When a PS session is created and authenticates through Kerberos, the session doesn't support double hop. Therefore, the PS session can't authenticate by using network resources.  

When PowerShelltries to enumerate the modules in the network path, the operation fails with "Access Denied," and the command unexpectedly terminates

## Resolution

To fix the issue, c reate the PS session to authenticate with CredSSP. This needs to be configured in advance. On the computer that is the target of the **Enter-PSSession** command, run this command:  
 Enable-WSManCredSSP -Role Server  
 On the computer on which you run the **Enter-PSSession** command, run this command:   Enable-WSManCredSSP -Role Client -DelegateComputer Servername  

> [!NOTE]
 Servernameis the name of the computer that is the target of the **Enter-PSSession** command. 

Every time that this command is executed, the specified Servernameis added to the list. The list is stored in the following registry subkey:  
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation\AllowFreshCredentials  
To view the status of the CredSSP configuration, run the **Get-WSManCredSSP** command.  
After CredSSP is enabled, you can authenticate through CredSSP by using this command:  
 Enter-PSSessionservername-Authentication CredSSP -Credential (Get-Credentialusername)

## Workaround

To work around this issue, map the network share to a drive letter such as **S:**, and then put the drive letter in the **PSModulePath**. Having a drive letter that points to a network share will not cause the unexpected termination of **Enter-PSSession**.  

 However, inside the remote PowerShell session the mapped drive letter will not be available, and the modules on the network share will still not be available. Only the local modules will be available.  

 This workaround will only prevent the **Enter-PSSession** from crashing while allowing normal PowerShell sessions to have access to the modules that are on the network share.