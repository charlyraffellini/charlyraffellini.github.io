---
layout: post
title: Authorization for Azure Management REST API using Azure Resource Manager
description: Obtaining authorization token for Azure Management REST API
date: 2018-03-28
categories: authentication authorization saas azure management rest microsoft api
img: 2018/authorization_azure_management_rest_api.jpg
author: Carlos Raffellini
---

# This blog post is about obtaining an `access_token` to call the Azure Management REST API.

For this post, I will consider the Azure Resource Manager deployment model which has `https://management.azure.com/` as URL. Service or classic deployment model is not considered. Classic deployment model use `https://management.core.windows.net` as URL.

## Pre-conditions

- Create an app registration.
- Assign the `Windows Azure Service Management API` delegated permission to your application. The cool thing is to put that permission in the application you don't need the big boss (AAD admin) authorization.
- A secret key for your application.

## Actions: Get the `access_token`

Because this is a delegation of permissions on behalf of the user you need to call Azure Active Directory twice.

The first time is to get the code that we will use in the second request to get the access token.

If you want more details about this authorization flow, visit my [previous post](/openid-oauth2-add).

```http
GET /{aad_tenant_id}/oauth2/authorize?client_id={your_app_id}&response_type=code&redirect_uri={any_reply_url}&response_mode=query&resource=https%3A%2F%2Fmanagement.azure.com%2F&state=12345&nonce=7362CAEA-9CZ5-4B43-9BA3-34D7C303EBA7&scope=openid  HTTP/1.1
Host: https://login.microsoftonline.com
```

This request returns the OAuth code we use to request the access token:

```http
POST /{aad_tenant_id}/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

code={code}&grant_type=authorization_code&client_id={your_app_id}&redirect_uri={any_reply_url}&resource=https%3A%2F%2Fmanagement.azure.com%2F&client_secret={app_secret_key}
```

This request returns an `access_token` we can use to call the Management REST API.

## Post-conditions

Using the `access_token` we will request a Virtual Machine information.

```http
GET /subscriptions/{subscription_id}/resourceGroups/{resource_group}/providers/Microsoft.Compute/virtualMachines/{vm_name}?api-version=2017-12-01 HTTP/1.1
Host: management.azure.com
Content-Type: application/json
Authorization: Bearer {access_token}
```

The response contains the representation of the Virtual Machine in Azure.

---

## References

[Azure REST API Reference](https://docs.microsoft.com/en-gb/rest/api/)

[Virtual Machines - Get Operation](https://docs.microsoft.com/en-gb/rest/api/compute/virtualmachines/get)

[JWT decoder](https://jwt.ms/)
