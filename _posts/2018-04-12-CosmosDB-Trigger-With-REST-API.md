---
layout: post
title: Create and invoke trigger in CosmosDb through REST API
description: About Cosmos DB Triggers, Authentication and REST API Operations
date: 2018-04-12
categories: saas azure storage rest microsoft api cosmosdb trigger
img: 2018/trigger-in-CosmosDb-through-REST-API.jpg
author: Carlos Raffellini
---

Table of Contents:

- [Create and invoke trigger in CosmosDb through REST API](#create-and-invoke-trigger-in-cosmosdb-through-rest-api)
    - [Pre-conditions](#pre-conditions)
    - [Create trigger](#create-trigger)
        - [Create Trigger REST Request](#create-trigger-rest-request)
    - [Post-Conditions](#post-conditions)
        - [Create Document Invoking Trigger REST Response](#create-document-invoking-trigger-rest-response)
    - [Conclusion](#conclusion)
    - [Extra: Calculate the authentication token](#extra-calculate-the-authentication-token)
    - [References](#references)

## Pre-conditions

- We should have a way to calculate the hash signature used for authentication to CosmosDb REST API. Help yourself with an example function in the [extra section](#extra-calculate-the-authentication-token).

- We should have a function that represents the execution of the trigger. This will go in the body of the POST request.

- CosmosDb request authorization token requires:
    - Resource type: `trigger`
    - Resource link: `dbs/{database}/colls/{collection}`
    - Key type used for the hash calculation: `master`
    - HTTP verb: `POST`

- Also we need the URL we will be using to create the trigger:
    - `"https://{cosmosdb-instance}.documents.azure.com/dbs/{database}/colls/{collection}/triggers"`

## Create trigger

Now we have all the things we need we can concentrate on triggering the request. I will show a PowerShell to do so. However, this is just a way to construct the HTTP request we will see after the script.

This trigger is going to pick the document before storing it in the collection. Then it will add the string `" - checked"` at the end of the `name` property and the normal flow of creating the document will proceed.

```powershell
$body = @"
{  
    "body": "function(){ var item = getContext().getRequest().getBody(); item.name = item.name + \" - checked\"; getContext().getRequest().setBody(item);}",
    "id": "pre-name-trigger",  
    "triggerOperation": "All",  
    "triggerType": "Pre"  
} 
"@

$verb = "POST"
$date = Get-Date
$dateString = $date.ToUniversalTime().ToString("r", [System.Globalization.CultureInfo]::InvariantCulture)
$resourceType = "triggers"
$resourceLink = "dbs/ToDoList/colls/Items"
$keyType = "master"
$key = "example key"
$secureKey = ConvertTo-SecureString -String $key -AsPlainText -Force

$token = Get-CosmosDbAuthorization `
    -Key $secureKey `
    -KeyType $keyType `
    -Verb $verb `
    -ResourceType $resourceType `
    -ResourceLink $resourceLink `
    -Date $dateString

$Headers = @{
    'authorization' = $token
    'x-ms-date'     = $dateString
    'x-ms-version'  = "2017-02-22"
}

$invokeRestMethodParameters = @{
    Uri         = "https://charliedb.documents.azure.com/dbs/ToDoList/colls/Items/triggers"
    Headers     = $Headers
    Method      = $verb
    ContentType = 'application/json'
    Body = $body
}

$restResult = Invoke-RestMethod @invokeRestMethodParameters
```

### Create Trigger REST Request

This is the request produced for the former script.

```HTTP
POST https://charliedb.documents.azure.com/dbs/ToDoList/colls/Items/triggers HTTP/1.1
authorization: type%3dmaster%26ver%3d1.0%26sig%3dluNK%2fJjtXtLTjsU1YZVBa3BjNvmC1XDX%2bg0a7EQLZDc%3d
x-ms-version: 2017-02-22
x-ms-date: Mon, 09 Apr 2018 20:21:01 GMT
Content-Type: application/json
Host: charliedb.documents.azure.com
Content-Length: 260

{  
    "body": "function(){ var item = getContext().getRequest().getBody(); item.name = item.name + \" - checked\"; getContext().getRequest().setBody(item);}",
    "id": "pre-name-trigger",  
    "triggerOperation": "All",  
    "triggerType": "Pre"  
} 
```

## Post-Conditions

Having a trigger stored in the database is not so interesting if we don't use it. Then, here is a way to create a document invoking the trigger before saving the document.

```powershell
$random = Get-Random
$body = @"
{  
    "id": "$(random)",
    "name": "new document $(random)"
} 
"@

$verb = "POST"
$date = Get-Date
$dateString = $date.ToUniversalTime().ToString("r", [System.Globalization.CultureInfo]::InvariantCulture)
$resourceType = "docs"
$resourceLink = "dbs/ToDoList/colls/Items"
$keyType = "master"
$key = "example key"
$secureKey = ConvertTo-SecureString -String $key -AsPlainText -Force

$token = Get-CosmosDbAuthorization `
    -Key $secureKey `
    -KeyType $keyType `
    -Verb $verb `
    -ResourceType $resourceType `
    -ResourceLink $resourceLink `
    -Date $dateString

$Headers = @{
    'authorization' = $token
    'x-ms-date'     = $dateString
    'x-ms-version'  = "2017-02-22"
    'x-ms-documentdb-pre-trigger-include' = "pre-name-trigger"
    #x-ms-documentdb-post-trigger-include: post-name-trigger
}

$invokeRestMethodParameters = @{
    Uri         = "https://charliedb.documents.azure.com/dbs/ToDoList/colls/Items/docs"
    Headers     = $Headers
    Method      = $verb
    ContentType = 'application/json'
    Body = $body
}

$restResult = Invoke-RestMethod @invokeRestMethodParameters
```

### Create Document Invoking Trigger REST Response

I am showing straight the REST response to the former request. The actual REST request is not so enlightening after showing the Create Trigger Request.

We see the document was created adding the string `" - checked"` at the end of the `name` property.

```HTTP
x-ms-schemaversion: 1.5
x-ms-alt-content-path: dbs/ToDoList/colls/Items
x-ms-content-path: K2BkAMizHwA=
x-ms-quorum-acked-lsn: 13
x-ms-current-write-quorum: 3
x-ms-current-replica-set-size: 4
x-ms-xp-role: 1
x-ms-global-Committed-lsn: 13
x-ms-number-of-read-regions: 1
x-ms-transport-request-id: 23
x-ms-share-throughput: true
x-ms-request-charge: 7.94
x-ms-serviceversion: version=1.21.0.0
x-ms-activity-id: 9c1197ef-dfd1-4b87-8960-380101b28b97
x-ms-session-token: 0:14
x-ms-gatewayversion: version=1.21.0.0
Date: Mon, 09 Apr 2018 20:50:45 GMT

110
{"id":"50657818","name":"new document 2087597658 - checked","_rid":"K2BkAMizHwACAAAAAAAAAA==","_self":"dbs\/K3BkAA==\/colls\/K2BkAFizHwA=\/docs\/K2BkAMizKwACAAAAAAAAAA==\/","_etag":"\"0000d740-0000-0000-0000-5acbd2261000\"","_attachments":"attachments\/","_ts":1523307046}
0
```

## Conclusion

This post went further than just the creation of a trigger. A normal user would have used the Cosmos DB SDK. Certainly for supporting a development process the REST API it isn't the best tool. You need some abstraction. However, for having a good understanding of Authentication and Operations working to the REST API level is a need.

Also, I showed how to use a trigger. It was a surprise seeing that the caller should specify which triggers to invoke. So, you should evaluate this to decide if it is suitable for your needs.

## Extra: Calculate the authentication token


```powershell
param
(
    [System.String]
    $KeyType = 'master',

    [System.String]
    $Key ="master key",

    [System.String]
    $BaseUrl = "https://localhost:8081/"
)
function Get-CosmosDbAuthorization
{
    [CmdletBinding()]
    [OutputType([System.String])]
    param
    (
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [System.Security.SecureString]

        $Key,

        [Parameter()]
        [ValidateSet('master', 'resource')]
        [System.String]
        $KeyType = 'master',

        [Parameter()]
        [ValidateSet('', 'Delete', 'Get', 'Head', 'Merge', 'Options', 'Patch', 'Post', 'Put', 'Trace')]
        [System.String]
        $Verb = '',

        [Parameter()]
        [System.String]
        $ResourceType = '',

        [Parameter()]
        [System.String]
        $ResourceLink = '',

        [Parameter(Mandatory = $true)]
        [System.String]
        $DateString,

        [Parameter()]
        [ValidateSet('1.0')]
        [System.String]
        $TokenVersion = '1.0'
    )

    $BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($Key)
    $decryptedKey = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
    $base64Key = [System.Convert]::FromBase64String($decryptedKey)
    $hmacSha256 = New-Object -TypeName System.Security.Cryptography.HMACSHA256 -ArgumentList (, $base64Key)
    $payLoad = @(
        $Verb.ToLowerInvariant() + "`n" + `
            $ResourceType.ToLowerInvariant() + "`n" + `
            $ResourceLink + "`n" + `
            $DateString.ToLowerInvariant() + "`n" + `
            "" + "`n"
    )

    $body = [System.Text.Encoding]::UTF8.GetBytes($payLoad)
    $hashPayLoad = $hmacSha256.ComputeHash($body)
    $signature = [Convert]::ToBase64String($hashPayLoad)

    Add-Type -AssemblyName 'System.Web'
    $token = [System.Web.HttpUtility]::UrlEncode(('type={0}&ver={1}&sig={2}' -f $KeyType, $TokenVersion, $signature))
    return $token
}
```

---

## References

- [Access control in the Azure Cosmos DB SQL API](https://docs.microsoft.com/en-gb/rest/api/cosmos-db/access-control-on-cosmosdb-resources)

- [Create Trigger REST Operation](https://docs.microsoft.com/en-us/rest/api/cosmos-db/create-a-trigger)

- [Create Document REST Operation](https://docs.microsoft.com/en-us/rest/api/cosmos-db/create-a-document)

- [Azure Cosmos DB server-side programming: Stored procedures, database triggers, and UDFs - x-ms-documentdb-{pre/post}-trigger-include](https://docs.microsoft.com/en-us/azure/cosmos-db/programming)
