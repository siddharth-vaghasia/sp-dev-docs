---
title: Replace an expiring client secret in a SharePoint Add-in
description: Add a new client secret for a SharePoint Add-in that is registered with AppRegNew.aspx.
ms.date: 06/22/2021
ms.prod: sharepoint
ms.localizationpriority: high
---

# Replace an expiring client secret in a SharePoint Add-in

Client secrets for SharePoint Add-ins that are registered by using the AppRegNew.aspx page expire after one year. This article explains how to add a new secret for the add-in, and how to create a new client secret that is valid for three years.

> [!NOTE]
> This article is about SharePoint Add-ins that are distributed through an organization catalog and registered with the AppRegNew.aspx page. If the add-in is registered on the Seller Dashboard, see [Create or update client IDs and secrets in the Seller Dashboard](/office/dev/store/create-or-update-client-ids-and-secrets).

## Prerequisites

Ensure the following before you begin:

- [Microsoft Online Services Sign-In Assistant](https://www.microsoft.com/download/details.aspx?id=39267) is installed on the development computer.
- You can connect to Office 365 with PowerShell: [Connect to Office 365 PowerShell](/office365/enterprise/powershell/connect-to-office-365-powershell)
- You're a tenant administrator for the Office 365 tenant (or a farm administrator on the farm) where the add-in was registered with the AppRegNew.aspx page.

## Find out the expiration dates of the SharePoint Add-ins installed to the Office 365 tenancy

1. Open Windows PowerShell and run the following cmdlet:

    ```powershell
    Connect-MsolService
    ```

1. At the sign-in prompt, enter tenant-administrator (or farm administrator) credentials for the Office 365 tenancy or farm where the add-in was registered with AppRegNew.aspx.
1. Generate a report that lists each add-in and the date that its secret expires with the following lines. Note the following about this code:

    - It first filters out Microsoft's own applications, add-ins still under development (and a now-deprecated type of add-in that was called autohosted).
    - From the remainder, it filters out non-SharePoint add-ins and add-ins that use asymmetric keys, such as workflows.

    ```powershell
    $applist = Get-MsolServicePrincipal -all  |Where-Object -FilterScript { ($_.DisplayName -notlike "*Microsoft*") -and ($_.DisplayName -notlike "autohost*") -and  ($_.ServicePrincipalNames -notlike "*localhost*") }

    foreach ($appentry in $applist) {
        $principalId = $appentry.AppPrincipalId
        $principalName = $appentry.DisplayName

        Get-MsolServicePrincipalCredential -AppPrincipalId $principalId -ReturnKeyValues $false | ? { $_.Type -eq "Password" } | % { "$principalName;$principalId;" + $_.KeyId.ToString() +";" + $_.StartDate.ToString() + ";" + $_.EndDate.ToString() } | out-file -FilePath c:\temp\appsec.txt -append
    }
    ```

1. Open the file C:\temp\appsec.txt to see the report. Leave the Windows PowerShell window open for the next procedure, if any of the secrets are near expiration.

## Generate a new secret

1. Create a client ID variable with the following line, using the client ID of the SharePoint Add-in as the parameter.

    ```powershell
    $clientId = 'client id of the add-in'
    ```

1. Generate a new client secret with the following lines:

    ```powershell
    $bytes = New-Object Byte[] 32
    $rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
    $rand.GetBytes($bytes)
    $rand.Dispose()
    $newClientSecret = [System.Convert]::ToBase64String($bytes)
    $dtStart = [System.DateTime]::Now
    $dtEnd = $dtStart.AddYears(1)
    New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Symmetric -Usage Sign -Value $newClientSecret -StartDate $dtStart -EndDate $dtEnd
    New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Symmetric -Usage Verify -Value $newClientSecret -StartDate $dtStart -EndDate $dtEnd
    New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Password -Usage Verify -Value $newClientSecret -StartDate $dtStart -EndDate $dtEnd
    $newClientSecret
    ```

1. The new client secret appears on the Windows PowerShell console. Copy it to a text file. You use it in the next procedure.

    > [!TIP]
    > By default, the add-in secret lasts one year. You can set this to a shorter or longer by using the **-EndDate** parameter on the three calls of the **New-MsolServicePrincipalCredential** cmdlet.

## Update the remote web application in Visual Studio to use the new secret

> [!IMPORTANT]
> If your add-in was originally created with a pre-release version of the Microsoft Office Developer Tools for Visual Studio, it may contain an out-of-date version of the TokenHelper.cs (or .vb) file. If the file does not contain the string "secondaryClientSecret", it is out of date and must be replaced before you can update the web application with a new secret. To obtain a copy of a release version of the file, you need Visual Studio 2012 or later. Create a new SharePoint Add-in project in Visual Studio. Copy the TokenHelper file from it to the web application project of your SharePoint Add-in.

1. Open the SharePoint Add-in project in Visual Studio, and open the web.config file for the web application project. In the **appSettings** section, there are keys for the client ID and client secret. The following is an example:

    ```XML
    <appSettings>
      <add key="ClientId" value="your client id here" />
      <add key="ClientSecret" value="your old secret here" />
        ... other settings may be here ...
    </appSettings>
    ```

1. Change the name of the **ClientSecret** key to `SecondaryClientSecret` as shown in the following example:

    ```XML
    <add key="SecondaryClientSecret" value="your old secret here" />
    ```

    > [!NOTE]
    > If you are performing this procedure for the first time, there is no **SecondaryClientSecret** property entry at this point in the configuration file. However, if you are performing the procedure for a subsequent client secret expiration (second or third), the property **SecondaryClientSecret** is already present and contains the initial or already expired old secret. In this case, delete the **SecondaryClientSecret** property first before renaming **ClientSecret**.

1. Add a new **ClientSecret** key and give it your new client secret. Your markup should now look like the following:

    ```XML
    <appSettings>
      <add key="ClientId" value="your client id here" />
      <add key="ClientSecret" value="your new secret here" />
      <add key="SecondaryClientSecret" value="your old secret here" />
        ... other settings may be here ...
    </appSettings>
    ```

    > [!IMPORTANT]
    > You will not be able to use the newly generated client secret until the current client secret expires. Therefore, changing the ClientId key to the new client secret without the SecondaryClientSecret key present will not work. You must follow the  procedure in this article and wait for the previous client secret to expire. You can then remove the SecondaryClientSecret if you want to.

1. If you changed to a new TokenHelper file, rebuild the project.
1. Republish the web application.

## Create a client secret that is valid for three years

For expired client secrets, first you must delete all of the expired secrets for a given **clientId**. You then create a new one with MSO PowerShell, wait at least 24 hours, and test the app with the new **clientId** and **ClientSecret** key.

1. Connect to MSOnline using the tenant admin user with the following markup using SharePoint Windows PowerShell.

    ```powershell
    import-module MSOnline
    $msolcred = get-credential
    connect-msolservice -credential $msolcred
    ```

1. Get **ServicePrincipals** and keys. Printing **$keys** returns three records. You also see the **EndDate** of each key. Confirm whether your expired key appears there.

    > [!NOTE]
    > The **clientId** needs to match your expired **clientId**. It's recommended to delete all keys, both expired and unexpired, for this **clientId**.

    ```powershell
    $clientId = "27c5b286-62a6-45c7-beda-abbaea6eecf2"
    $keys = Get-MsolServicePrincipalCredential -AppPrincipalId $clientId
    $keys
    ```

1. Remove all keys once you have confirmed that they are indeed expired.

    ```powershell
    Remove-MsolServicePrincipalCredential -KeyIds $keys.KeyId -AppPrincipalId $clientId
    ```

1. Generate a new **ClientSecret** for this **clientID**. It uses the same **clientId** as set in the preceding step. The new **ClientSecret** is valid for three years.

    ```powershell
    $bytes = New-Object Byte[] 32
    $rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
    $rand.GetBytes($bytes)
    $rand.Dispose()
    $newClientSecret = [System.Convert]::ToBase64String($bytes)
    $dtStart = [System.DateTime]::Now
    $dtEnd = $dtStart.AddYears(3)
    New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Symmetric -Usage Sign -Value $newClientSecret -StartDate $dtStart  -EndDate $dtEnd
    New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Symmetric -Usage Verify -Value $newClientSecret   -StartDate $dtStart  -EndDate $dtEnd
    New-MsolServicePrincipalCredential -AppPrincipalId $clientId -Type Password -Usage Verify -Value $newClientSecret   -StartDate $dtStart  -EndDate $dtEnd
    $newClientSecret
    ```

1. Copy the output of **$newClientSecret**.
1. Replace the **Web.config** with this **ClientId** and **ClientSecret**. You don't need **SecondaryClientSecret** app settings.
1. Wait at least 24 hours to propagate **ClientSecret** to SharePoint Office (SPO).

## See also

- [Provider Hosted App fails on SPO](/archive/blogs/sharepointdevelopersupport/provider-hosted-app-fails-on-spo)
- [Creating SharePoint Add-ins that use low-trust authorization](creating-sharepoint-add-ins-that-use-low-trust-authorization.md)
- [Authorization and authentication of SharePoint Add-ins](authorization-and-authentication-of-sharepoint-add-ins.md)
