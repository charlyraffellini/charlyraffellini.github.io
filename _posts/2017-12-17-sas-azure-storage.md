---
layout: post
title: How Shared Access Signature works in Azure Storage
description: Create, use and invalidate SAS
date: 2017-12-17
categories: automation infrastructure azure storage 
img: 2017/sas_azure_storage.png
author: Carlos Raffellini
---

Shared Access Signatures are credentials we can use to access to blobs or containers in Azure Storage (Standard Storage and Blob Storage). The main reasons I find to use SAS are:

  - You can give access to limited resources in your Azure Storage without compromising others.
  - Ability to control access permissions (read, write) to the owners of a SAS.
  - Not need to share the kingdom key (`Access keys`).
  - Super easy invalidation of all created SAS. Just modifying any of the `Access keys` invalidates all the SAS previously created.
  - Granular access control using IP ranges, expiration time, protocol, type of resource (blob, container, file, queue or table).

Also with SAS, you can delegate access to an entire service like blob, file, queue, and table services. You can delegate access to a group of services with the same SAS.

Let's see an example of how to create a SAS that delegates read access to a blob for 2 hours:

```powershell
[CmdletBinding()]
Param (
    [Parameter(Mandatory=$false)]
    $ResourceGroupName = "my_resource_group",

    [Parameter(Mandatory=$false)]
    $Location = "centralus"
)

$ErrorActionPreference = "Stop"

Get-AzureRmResourceGroup -Name $ResourceGroupName -ev rgNotPresent -ea SilentlyContinue

if ($rgNotPresent){
    New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Location
}

$StorageAccount = New-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName `
-Name "mystorageaccount" `
-Location $Location `
-SkuName Standard_LRS `
-Kind BlobStorage `
-AccessTier Hot `
-EnableEncryptionService Blob

$Context = $storageAccount.Context

$ContainerName = "mycontainer"
New-AzureStorageContainer -Name $ContainerName -Context $Context -Permission blob

$localFileDirectory = "C:\Users\carlo\myfolder"
$blobName = "myfile.txt"
$localFile = $localFileDirectory + $blobName
Set-AzureStorageBlobContent -File $localFile `
-Container $ContainerName `
-Blob $blobName `
-Context $Context
  
Get-AzureStorageBlob -Container $ContainerName -Context $Context | select Name

$templateuri = New-AzureStorageBlobSASToken -Container $ContainerName `
-Blob $blobName `
-Permission r `
-ExpiryTime (Get-Date).AddHours(2.0) `
-FullUri `
-Context $Context

Write-Host $templateuri
```

This scripts return the following URL `https://mystorageaccount.blob.core.windows.net/mycontainer/myfile.txt?sv=2017-04-17&sr=b&sig=fSJixgtejg1RT%2FCnyn0bPLitTdvOvfahv%2BMNum%2BW3Qg%3D&se=2017-12-17T15%3A56%3A39Z&sp=r`.

Let's break this a bit:

  - `https://mystorageaccount.blob.core.windows.net/mycontainer/myfile.txt` this is the resource we want to access. It is the blob called `myfile.txt`, in the container called `mycontainer` and contained by the storage account `mystorageaccount`.
  - `sr=b` this means the resource we are trying to access is a blob.
  - `se=2017-12-17T15%3A56%3A39Z` the expiration time after URL encoding.
  - `sp=r` the permisions are read only.\
  - `sig=fSJixgtejg1RT%2FCnyn0bPLitTdvOvfahv%2BMNum%2BW3Qg` this is the signature of the whole URL. This sinature is created using the `Access keys` of the storage account and validated against them for every reques. In case any of the `Access keys` change this signature is not longer valid and the request will return `AuthenticationFailed`. The same will happen if we modify any character of this URL.
  - `sv=2017-04-17` this is the REST service version we used to create the signature.

If we modify any part of this URL (for instance adding write permission `sp=rw`) and we attempt to request again we will get this error message:


`GET https://mystorageaccount.blob.core.windows.net/mycontainer/myfile.txt?sv=2017-04-17&sr=b&sig=fSJixgtejg1RT%2FCnyn0bPLitTdvOvfahv%2BMNum%2BW3Qg%3D&se=2017-12-17T15%3A56%3A39Z&sp=rw`

```xml
<Error>
    <Code>AuthenticationFailed</Code>
    <Message>
        Server failed to authenticate the request. Make sure the value of Authorization header is formed correctly including the signature. RequestId:7e6c45cf-001e-00ad-4841-77f7b8000000 Time:2017-12-17T14:16:49.8534267Z
    </Message>
    <AuthenticationErrorDetail>
        Signature did not match. String to sign used was r 2017-12-17T15:56:39Z /blob/mystorageaccount/mycontainer/myfile.txt 2017-04-17
    </AuthenticationErrorDetail>
</Error>
```

This is the same error we receive when the `Access keys` for the storage account were modified.

### It was a brief introduction about SAS. For more information visit the following URLs:

  - https://docs.microsoft.com/en-us/azure/storage/common/storage-dotnet-shared-access-signature-part-1
  - https://docs.microsoft.com/en-us/azure/storage/blobs/storage-dotnet-shared-access-signature-part-2

