---
layout: post
title: AAD Part II - About OpenId and OAuth 2.0 tokens in Azure Active Directory
description: OAuth2 service-to-service authorization in Azure AD
date: 2018-02-14
categories: authentication authorization saas azure active directory 
img: 2018/azure_active_dierctory_part_2.jpg
author: Carlos Raffellini
---

This is a sequel of my [previous post](/openid-oauth2-add/) about Azure Active Directory. In my experience using Azure AD, I found plenty of documentation for many different flows and use cases. Also, since the vocabulary used clashes with other context making the understanding harder. For instance, the words, application, service, etc; they have a very technical meaning in this context and it takes a while to get used to.

This time I am going to talk about using application credentials to authorize to another application.


# Let's authorize an application to call another with specific roles

## Preconditions

Given an app registration called `Producer` with an application role and another application called `Consumer`.

```json
{
    ...
    "appId": "387cfb6f-d0f7-45ab-8d7f-25206509997a",
    "appRoles": [
    {
        "allowedMemberTypes": [
        "Application"
        ],
        "displayName": "Manage",
        "id": "81e10148-16a8-432a-b86d-ef620c3e48ef",
        "isEnabled": true,
        "description": "Manager role for the Producer Application.",
        "value": "Manage"
    }
    ],
    "displayName": "Producer",
    ...
}
```

```json
{
    ...
    "appId": "ce4accbd-24cf-452d-a622-a15df8310b1b",
    "appRoles": [],
    "displayName": "Consumer",
    ...
}
```

We will need a **secret key** for Consumer.

## Requesting an `access_token`

Before requesting an access_token let's add the access to Consumer to call Producer.

There are two ways to do it. Through the portal or using the manifest directly.

After adding the access in Consumer we finish with the following manifest:

```json
{
    ...
    "appId": "ce4accbd-24cf-452d-a622-a15df8310b1b",
    "displayName": "Consumer",
    ...
    "requiredResourceAccess": [
    {
        "resourceAppId": "387cfb6f-d0f7-45ab-8d7f-25206509997a",
        "resourceAccess": [
            {
                "id": "81e10148-16a8-432a-b86d-ef620c3e48ef",
                "type": "Role"
            }
        ]
    },
    ...
    ],
    ...
}
```

#### IMPORTANT: Click on **Grant Permissions** button for the Consumer app. This can only be done by an Azure AD administrator and cannot be automated.

Now let's get the the token:

```
POST /{my_tenant_id}/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=ce4accbd-24cf-452d-a622-a15df8310b1b&client_secret={client_secret}&resource=387cfb6f-d0f7-45ab-8d7f-25206509997a

Response:

{
    "token_type": "Bearer",
    "expires_in": "3600",
    "ext_expires_in": "0",
    "expires_on": "1518640821",
    "not_before": "1518636921",
    "resource": "387cfb6f-d0f7-45ab-8d7f-25206509997a",
    "access_token": "{a_long_jwt}"
}
```

Then we can validate that our token has the `Manage` role. Looking at the payload:

```json
{
  "aud": "387cfb6f-d0f7-45ab-8d7f-25206509997a",
  ...
  "appid": "ce4accbd-24cf-452d-a622-a15df8310b1b",
  "appidacr": "1",
  "idp": "https://sts.windows.net/{tenant_id}/",
  ...
  "roles": [
    "Manage"
  ],
  "tid": "{tenant_id}",
  ...
  "ver": "1.0"
}
```

# Postconditions

We can verify that the token signature is valid. We can use the method I describe in my [previous post](/openid-oauth2-add/).


---

# References

[Service to service athorization](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-service-to-service)