---
layout: post
title: About Graph API and Azure Active Directory - Part III
description: About Graph API and Azure Active Directory
date: 2018-01-31
categories: authentication authorization saas azure active directory graphapi
img: 2018/azure_active_dierctory_part_3.jpg
author: Carlos Raffellini
---

# Azure Active Directory Part III - Calling Graph API using access token

This time I am going to show how to get an access token to access to Graph API resource. I show in my previous post how to get an access_token to access another Web API on-behalf-of another user. In that case, the resource we were accessing was another API. This time the resource is https://graph.microsoft.com.

## Preconditions

We will be working with an application which in its manifest has the following access:

  ```json
  "requiredResourceAccess": [
    {
      "resourceAppId": "00000002-0000-0000-c000-000000000000",
      "resourceAccess": [
        {
          "id": "311a71cc-e848-46a1-bdf8-97ff7156d8e6",
          "type": "Scope"
        }
      ]
    }
  ],
  ```

This access is a permission to UserProfile.Read on the Active Directory Application. This means an `access_token` that has UserProfile.Read in its scope can read the profile on-behalf-of the user.

## Let’s get a token and query Graph API

As usual, we will request a code which later we will change for an access_token

```
GET /{my_teant_id}/oauth2/authorize?response_type=code
&redirect_uri={url_encoded_return_url}
&response_mode=query
&state=12345
&nonce=7362CAEA-9CA5-4B43-9BA3-34D7C303EBA7
&resource=https://graph.microsoft.com
&client_id={client_id_of_my_app} HTTP/1.1

Host: login.microsoftonline.com
```

This request returns a redirect to the reply URL with the code in the query string.

Then we perform another request to get the `access_token`:

```
POST /{my_tenant_id}/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

code={the_code_we_just_got}&redirect_uri={the_same_redirect_url}&client_id={app_id}&client_secret={secret_key}&grant_type=authorization_code&resource={url encoded https://graph.microsoft.com }
```

We must have received an access_token and `refresh_token`. Let’s focus only on the `access_token`.

The token is a base 64 encoded JWT. When we decode the payload we see the following:

```json
{
  "aud": "https://graph.microsoft.com",
  "iss": "https://sts.windows.net/{my_tenant_id}/", 
  …
  "family_name": "Raffellini",
  "given_name": "Carlos",
  "name": "Raffellini, Carlos",
  …
  "scp": "User.Read",
  "ver": "1.0"
}
```

## Postconditions

The most important thing here is to recognize that the audience (`aud`) that this token is issued for is the Graph API and the scope (`scp`) this access token have is "User.Read". This means we can then use this token to get the profile information of the user. Let’s do that.

Have in mind we are using the new Graph API (graph.microsoft.com) that covers more than the previous Azure AD Graph API (https://graph.windows.net). Also, Azure AD Graph API is becoming deprecated.

```
GET /v1.0/me HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer {the_access_token_we_just_got}
```

We got the profile of the user we are acting on-behalf-of. The response we get looks like:

```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "id": {id}
    "businessPhones": [
        "some kind of phone number"
    ],
    "displayName": "Raffellini, Carlos",
    "givenName": "Carlos",
    "jobTitle": "Software Engineer",
    "mail": "carlosraffellini@gmail.com",
    "mobilePhone": "personal phone",
    "officeLocation": "office location",
    "preferredLanguage": null,
    "surname": "Raffellini",
    "userPrincipalName": "Carlos.Raffellini@{my_tenant_id}"
}
```

Thank you for reading. My next post will be about calling the Graph API using Aplication access intead of User permissions.

---

# References


[Scopes, permissions, and consent in the Azure Active Directory v2.0 endpoint](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-scopes)

[Assign a user or group to an enterprise app in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

[Quickstart for the Azure AD Graph API](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-graph-api-quickstart)

[Authorize access to web applications using OAuth 2.0 and Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code)

[Featured scenarios for Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/concepts/featured_scenarios)

[Get access tokens to call Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_overview)

[Azure AD Permissions – summary table](http://www.cloudidentity.com/blog/2015/09/01/azure-ad-permissions-summary-table/)
