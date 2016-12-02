---
title: "Add a Windows Driver to a Boot Image Package | Microsoft Docs"
ms.custom: ""
ms.date: "09/20/2016"
ms.prod: "configuration-manager"
ms.reviewer: ""
ms.suite: ""
ms.technology:
  - "configmgr-other"
ms.tgt_pltfrm: ""
ms.topic: "article"
applies_to:
  - "System Center Configuration Manager (current branch)"
ms.assetid: 5887b585-618e-42ed-a707-d374d0da4b0b
caps.latest.revision: 9
author: "shill-ms"
ms.author: "v-suhill"
manager: "mbaldwin"
---
# How to Add a Windows Driver to a Configuration Manager Boot Image Package
In System Center Configuration Manager, you add a Windows driver to an operating system deployment boot image package by adding a reference to the required driver in the [SMS_BootImagePackage Server WMI Class](../../develop/reference/osd/sms_bootimagepackage-server-wmi-class.md)[ReferencedDrivers](assetId:///ReferencedDrivers?qualifyHint=False&autoUpgrade=True) array property.  

> [!NOTE]
>  The assetId:///ReferencedDrivers?qualifyHint=False&autoUpgrade=True property is an array of an embedded [SMS_Driver_Details](assetId:///SMS_Driver_Details?qualifyHint=False&autoUpgrade=True) object, and you can add more than one driver to the package. The objects in the array are added to the boot image package each time it is updated on the distribution point.  

 The location of the driver content is usually obtained from the [SMS_Driver Server WMI Class](../../develop/reference/osd/sms_driver-server-wmi-class.md) object `ContentSourcePath` property, but this can be overridden if the original driver location is not available.  

 It might be necessary to add network or storage drivers to a boot image package so that a task sequence can access the network and disk resources while in WinPE.  

 Drivers are added to the image only when the boot image is refreshed by calling the [RefreshPkgSource Method in Class SMS_BootImagePackage](../../develop/reference/osd/refreshpkgsource-method-in-class-sms_bootimagepackage.md) method.  

 Drivers are added to the image by using Windows Package Manager.  

### To add a Windows driver to a boot image package  

1.  Set up a connection to the SMS Provider. For more information, see [About the SMS Provider in Configuration Manager](../../develop/core/understand/about-the-sms-provider-in-configuration-manager.md).  

2.  Get the [SMS_BootImagePackage](assetId:///SMS_BootImagePackage?qualifyHint=False&autoUpgrade=True) object for the boot image package that you want to add the driver to.  

3.  Create and populate an embedded assetId:///SMS_Driver_Details?qualifyHint=False&autoUpgrade=True object to contain the driver details.  

4.  Add the assetId:///SMS_Driver_Details?qualifyHint=False&autoUpgrade=True object to the assetId:///ReferencedDrivers?qualifyHint=False&autoUpgrade=True array property of the assetId:///SMS_BootImagePackage?qualifyHint=False&autoUpgrade=True object.  

5.  Commit the assetId:///SMS_BootImagePackage?qualifyHint=False&autoUpgrade=True object changes.  

## Example  
 The following example method adds a Windows driver to a boot image package. The package is identified by its [PackageID](assetId:///PackageID?qualifyHint=False&autoUpgrade=True) property, and the driver is identified by its [CI_ID](assetId:///CI_ID?qualifyHint=False&autoUpgrade=True) property.  

 For information about calling the sample code, see [Calling Configuration Manager Code Snippets](../../develop/core/understand/calling-code-snippets.md).  

```vbs  
Sub AddDriverToBootImagePackage(connection, driverId,packageId)  

    Dim bootImagePackage  
    Dim driver   
    Dim referencedDrivers  
    Dim driverDetails  

    ' Get the boot image package and referenced drivers.  
    Set bootImagePackage = connection.Get("SMS_BootImagePackage.PackageID='" & packageId &"'" )  
    referencedDrivers = bootImagePackage.ReferencedDrivers  

    ' Get the driver.  
    Set driver = connection.Get("SMS_Driver.CI_ID=" & driverId )  

    ' Create and populate the driver details.  
    Set driverDetails = connection.Get("SMS_Driver_Details").SpawnInstance_  
    driverDetails.ID=driverId  
    driverDetails.SourcePath=driver.ContentSourcePath  

    ' Add the driver details.  
    ReDim Preserve referencedDrivers (Ubound (referencedDrivers)+1)  
    Set referencedDrivers(Ubound(referencedDrivers))=driverDetails  
    bootImagePackage.ReferencedDrivers=referencedDrivers  

    bootImagePackage.Put_   
    bootImagePackage.RefreshPkgSource   

End Sub  
```  

```c#  
public void AddDriverToBootImagePackage(  
    WqlConnectionManager connection,   
    int driverId,   
    string packageId)  
{  
    try  
    {  
        // Get the boot image package.  
        IResultObject bootImagePackage = connection.GetInstance(@"SMS_BootImagePackage.packageId='" + packageId + "'");  

        // Get the driver.  
        IResultObject driver = connection.GetInstance("SMS_Driver.CI_ID=" + driverId);  

        // Get the drivers that are referenced by the package.  
        List<IResultObject> referencedDrivers = bootImagePackage.GetArrayItems("ReferencedDrivers");  

        // Create and populate an embedded SMS_Driver_Details. This is added to the ReferencedDrivers array.  
        IResultObject driverDetails = connection.CreateEmbeddedObjectInstance("SMS_Driver_Details");  

        driverDetails["ID"].IntegerValue = driverId;  
        driverDetails["SourcePath"].StringValue = driver["ContentSourcePath"].StringValue;  

        // Add the driver details to the array.  
        referencedDrivers.Add(driverDetails);  

        // Add the array to the boot image package.  
        bootImagePackage.SetArrayItems("ReferencedDrivers", referencedDrivers);  

        // Commit the changes.  
        bootImagePackage.Put();  
        bootImagePackage.ExecuteMethod("RefreshPkgSource", null);  
    }  
    catch (SmsException e)  
    {  
        Console.WriteLine(e.Message);  
        throw;  
    }  
}  
```  

 The example method has the following parameters:  

|Parameter|Type|Description|  
|---------------|----------|-----------------|  
|`Connection`|-   Managed:[WqlConnectionManager](assetId:///WqlConnectionManager?qualifyHint=False&autoUpgrade=True)<br />-   VBScript: [SWbemServices](assetId:///SWbemServices?qualifyHint=False&autoUpgrade=True)|A valid connection to the SMS Provider.|  
|`driverID`|-   Managed: `String`<br />-   VBScript:  `String`|The Windows driver identifier available in [SMS_Driver.CI_ID](assetId:///SMS_Driver.CI_ID?qualifyHint=False&autoUpgrade=True).|  
|`PackageID`|-   Managed: `String`<br />-   VBScript: `String`|The boot image package identifier available in [SMS_BootImagePackage.PackageID](assetId:///SMS_BootImagePackage.PackageID?qualifyHint=False&autoUpgrade=True).|  

## Compiling the Code  
 This C# example requires:  

### Namespaces  
 System  

 System.Collections.Generic  

 System.Text  

 Microsoft.ConfigurationManagement.ManagementProvider  

 Microsoft.ConfigurationManagement.ManagementProvider.WqlQueryEngine  

### Assembly  
 microsoft.configurationmanagement.managementprovider  

 adminui.wqlqueryengine  

## Robust Programming  
 For more information about error handling, see [About Configuration Manager Errors](../../develop/core/understand/about-configuration-manager-errors.md).  

## .NET Framework Security  
 For more information about securing Configuration Manager applications, see [Securing Configuration Manager Applications](../../develop/core/understand/securing-configuration-manager-applications.md).  

## See Also  
 [About Operating System Deployment Driver Management](../../develop/osd/about-operating-system-deployment-driver-management.md)   
 [How to Remove a Windows Driver from a Boot Image Package](../../develop/osd/how-to-remove-a-windows-driver-from-a-boot-image-package.md)   
 [Operating System Deployment Driver Management](../../develop/osd/operating-system-deployment-driver-management.md)