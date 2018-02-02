---
layout: post
title: About OpenId and OAuth 2.0 tokens in Azure Active Directory
description: About OpenId and OAuth 2.0 tokens in Azure Active Directory
date: 2018-01-31
categories: authentication authorization saas azure active directory 
img: 2018/azure_active_dierctory.jpg
author: Carlos Raffellini
---

Azure Active Directory has plenty of documentation and scenarios you can use for authentication and authorization. That's why I am motivated to write this post.

For this post, I am considering Single-Tenant Applications only. I am relaying in Azure AD HTTP endpoints instead of using libraries. Multi-Tenant Applications will be considered in future posts. Also, most of this post consider portal interaction, automation scripts will be considered in future posts.

For [**authentication** we have 3 protocols](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-protocols):

- [OpenId Connect](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-openid-connect-code)
- SAML 2.0
- WS-Federation


For **authorization,** we have the [Azure AD implementation of OAuth 2.0](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code).

In this post, I am going to talk about Authorizing **Users** and **Apps On-Behalf-Of a user** to an application.

## Let's sort out user authentication and authorization with OpenId Connect.

### Preconditions:

- Application with `appRoles` defined in its manifest.
- Application Id.
- Application redirect url.
- A user with id and password. The user must have at least one of the roles assigned by the app. This user must consent the permissions required by the app during the consent flow. Also, permissions can be granted by the administrator of the Directory using Azure Portal only.

In this case I have an app showing the following roles in its manifest:

```
{
    "appId": "3687a30c-1296-4d61-ad5c-e20dceec8f3e",
    ...
    "appRoles": [
        {
        "allowedMemberTypes": [
            "User"
        ],
        "displayName": "Admin",
        "id": "81e10148-16a8-432a-b86d-ef620c3e48ef",
        "isEnabled": true,
        "description": "Admins can manage roles and perform all task actions.",
        "value": "Admin"
        },
        {
        "allowedMemberTypes": [
            "User"
        ],
        "displayName": "Approver",
        "id": "fc803414-3c61-4ebc-a5e5-cd1675c14bbb",
        "isEnabled": true,
        "description": "Approvers have the ability to change the status of tasks.",
        "value": "Approver"
        },
        {
        "allowedMemberTypes": [
            "User"
        ],
        "displayName": "Observer",
        "id": "fcac0bdb-e45d-4cfc-9733-fbea156da358",
        "isEnabled": true,
        "description": "Observers only have the ability to view tasks and their statuses.",
        "value": "Observer"
        },
        {
        "allowedMemberTypes": [
            "User"
        ],
        "displayName": "Writer",
        "id": "d1c2ade8-98f8-45fd-aa4a-6d06b947c66f",
        "isEnabled": true,
        "description": "Writers Have the ability to create tasks.",
        "value": "Writer"
        }
    ]
    ...
}
```

I added a user as Observer

![role_assignation](/assets/images/2018/app_user_role_assignation.jpg)

### Requesting an `id_token`

This part of the OpenId flow has the following steps:

- A login call to Azure Active Directory.
- Eventually, a redirect to consent the permissions.
- A redirect to the redirect url we specified in the first request. In Azure AD we can have more than one redirect url registered in our application.

```
GET /{azure_active_directory_tenant_id}/oauth2/authorize
    ?client_id={app_id}
    &response_type=id_token
    &redirect_uri=http%3A%2F%2Flocalhost%3A4628%2F
    &response_mode=query //We want to receive the id_token in the query string
    &state=12345
    &nonce=7362CAEA-9CA5-4B43-9BA3-34D7C303EBA7
    &scope=openid 
    HTTP/1.1
Host: login.microsoftonline.com
```

The arguments are quite self-explanatory.

### Postconditions (of the happy path)
  - The previous flow ends in a redirect request to the redirect url specified.
  - The redirect request contains the `id_token`.

Example of redirect request:

```
GET /?id_token={jwt}
&state=12345
&session_state=7576f5e6-676b-4dc2-88fb-dfb8d8b29e3d 
HTTP/1.1

Host: localhost:4628
```

The `token_id` is a JWT that contains roles the user has assigned to the application. Having a look at the payload of the token:

```
{
  ...
  "roles": [
    "Observer"
  ],
  ...
}
```

We were able to authenticate the user and now the application can decide whether to get access to resources or return an authentication failure response. Do not forget to validate the token signature with the public keys provided by Azure AD: `https://login.microsoftonline.com/common/discovery/keys`.

In ASP.NET applications you can use the package `Microsoft.Owin.Security.OpenIdConnect` to manage this authentication flow for you.

For the authorization based on user roles in `System.Web.Mvc.Controller` we can use the `System.Web.Mvc.AuthorizeAttribute`.

```csharp
    [HttpGet]
    [Authorize(Roles = "Admin, Observer, Writer, Approver")]
    public ActionResult Index()
    {
        ...
    }
```

Also, we can ask the `System.Security.Principal.IPrincipal` whether the user has the role, `User.IsInRole("Writer")`. Also `ClaimsPrincipal.Current.Claims` has a `roles` claim.


## Let's get the token to access resources on-behalf-of a User with OAuth 2.0.

### Preconditions

- Application with `oauth2Permissions` defined in its manifest.
- Another app with required access to the above permissions.
- Application redirect url.
- A user with id and password. The user must have at least one of the roles assigned by the app. This user must consent the permissions required by the app during the consent flow. Also, permissions can be granted by the administrator of the Directory using Azure Portal only.

One of my apps exposes the following permissions:

```
{
    "appId": "3687a30c-1296-4d61-ad5c-e20dceec8f3e",
    ...
    "oauth2Permissions": [
        {
            ...
            "id": "5457d4dc-38ed-48e9-815d-7d255f495816",
            "type": "User",
            "value": "AppAdmin"
            ...
        },
        {
            ...
            "id": "852bc905-2362-4702-8bea-0a8c19e58799",
            "type": "User",
            "value": "AppObserver"
            ..
        },
        {
            ...
            "id": "e7962288-cb46-49f5-93d6-1e41e52cd293",
            "type": "User",
            "value": "user_impersonation"
            ...
        }
    ],
    ...
}
```

The second app has the required permissions in the `requiredResourceAccess` field of its manifest:

```
{
    ...
        "appId": "4cd406ee-8b7b-4ae9-8ab8-dd9fea0b0477"
    ...
    "requiredResourceAccess": [
        {
        "resourceAppId": "3687a30c-1296-4d61-ad5c-e20dceec8f3e",
        "resourceAccess": [
            {
            "id": "852bc905-2362-4702-8bea-0a8c19e58799",
            "type": "Scope"
            },
            {
            "id": "5457d4dc-38ed-48e9-815d-7d255f495816",
            "type": "Scope"
            }
        ]
        },
    ...
    ],
    ...
}
```

Since `852bc905-2362-4702-8bea-0a8c19e58799` is "AppObserver" and `5457d4dc-38ed-48e9-815d-7d255f495816` is "AppAdmin".

### Requesting an `access_token`

The OAuth flow starts with a login request initiated by the app that wants to access the resources of an API:

```
GET /{tenant_id}/oauth2/authorize
    ?client_id=4cd406ee-8b7b-4ae9-8ab8-dd9fea0b0477
    &response_type=code
    &redirect_uri=https%3A%2F%2Flocalhost%3A4444%2F
    &response_mode=query
    &state=12345
    &nonce=7362CAEA-9CA5-4B43-9BA3-34D7C303EBA7
    &resource=3687a30c-1296-4d61-ad5c-e20dceec8f3e
    &prompt=consent HTTP/1.1
Host: login.microsoftonline.com
```

You can see `client_id=4cd406ee-8b7b-4ae9-8ab8-dd9fea0b0477` is the app that initiate the flow and `resource=3687a30c-1296-4d61-ad5c-e20dceec8f3e` represents the resource the user is asked to consent.

The Authorization server comes back with a request to:

```
GET /?code={oauth_authorization_code}
    &state=12345
    &session_state={session_state}
    HTTP/1.1
Host: localhost:4444
```

We are interested in the `code` value. In the next step, our app asks the Authorization Server to change the code for an access token.

```
Request:

POST /f8cfc101-7f27-4067-a166-55057162058a/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

code={oauth_authorization_code}
&redirect_uri=https%3A%2F%2Flocalhost%3A4444%2F
&client_id=4cd406ee-8b7b-4ae9-8ab8-dd9fea0b0477
&client_secret={some_secret}
&grant_type=authorization_code
&resource=3687a30c-1296-4d61-ad5c-e20dceec8f3e

Response:

{
    "token_type": "Bearer",
    "scope": "AppAdmin AppObserver",
    "expires_in": "3599",
    "ext_expires_in": "0",
    "expires_on": "1517497587",
    "not_before": "1517493687",
    "resource": "3687a30c-1296-4d61-ad5c-e20dceec8f3e",
    "access_token": {access_token},
    "refresh_token": {refresh_token},
    "id_token": {id_token_without_signature}
}
```

The `access_token` is a JWT with header and signature. It contains the scopes of the token, the audience, etc. Let's have a look at a simplified payload:

```
{
  "aud": "3687a30c-1296-4d61-ad5c-e20dceec8f3e",
  "appid": "4cd406ee-8b7b-4ae9-8ab8-dd9fea0b0477",
  "roles": [
    "Observer"
  ],
  "scp": "AppAdmin AppObserver",
  "tid": "f8cfc101-7f27-4067-a166-55057162058a",
  "ver": "1.0"
}
```

Having a look at the [documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-token-and-claims) for token and claims:

- `aud`	**Audience**	The intended recipient of the token. The application that receives the token must verify that the audience value is correct and reject any tokens intended for a different audience.

- `appid`	**Application ID**	Identifies the application that is using the token to access a resource. The application can act as itself or on behalf of a user. The application ID typically represents an application object, but it can also represent a service principal object in Azure AD. 

- `scp`	**Scope**	Indicates the impersonation permissions granted to the client application. The default permission is user_impersonation. The owner of the secured resource can register additional values in Azure AD.

- `roles`	**Roles**	Represents all application roles that the subject has been granted both directly and indirectly through group membership and can be used to enforce role-based access control. Application roles are defined on a per-application basis, through the `appRoles` property of the application manifest. The `value` property of each application role is the value that appears in the roles claim. 

- `ver`	**Version**	Stores the version number of the token. 



### Postconditions


The `access_token` is a jwt that contains what permissions the app has on-behalf-of the user.

#### An example of using the scope the validate whether the app has permissions to request a resource.

```csharp
    Claim scopeClaim = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/scope");
    if (scopeClaim != null)
    {
        if (scopeClaim.Value != "user_impersonation")
        {
            throw new HttpResponseException(new HttpResponseMessage { StatusCode = HttpStatusCode.Unauthorized, ReasonPhrase = "The Scope claim does not contain 'user_impersonation' or scope claim not found" });
        }
    }
```

An example of configuring the authorization using Bearer tokens.

```csharp
    public partial class Startup
    {
        public void ConfigureAuth(IAppBuilder app)
        {
            app.UseWindowsAzureActiveDirectoryBearerAuthentication(
                new WindowsAzureActiveDirectoryBearerAuthenticationOptions
                {
                    Audience = ConfigurationManager.AppSettings["ida:Audience"],
                    Tenant = ConfigurationManager.AppSettings["ida:Tenant"]
                });
        }
    }
```


---

# Extras

#### OpenId flow of Microsoft.Owin.Security.OpenIdConnect v3.0.1 library

```csharp
public class AccountController : Controller
{
    public void SignIn(string redirectUri)
    {
        if (redirectUri == null)
            redirectUri = "/";

        HttpContext.GetOwinContext()
            .Authentication.Challenge(new AuthenticationProperties {RedirectUri = redirectUri},
                OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}
```

Produces a redirect to:

```
GET https://login.microsoftonline.com/{tenant_id}/oauth2/authorize
    ?client_id={client_id}
    &response_mode=form_post
    &response_type=code+id_token
    &scope=openid+profile
    &state={state_will_be_returned}
```

Which after a successful login and permission consent produces a post request. This kind of POST request is what we specified in the original query `response_mode=form_post`. This is the way the flow have to communicate with the app after the authorization. For instance:

```
Request
POST http://localhost:4628/ HTTP/1.1
Host: localhost:4628
Content-Type: application/x-www-form-urlencoded
Set-Cookie: ...

code={oauth_code}&id_token={id_token}&state={state_previously_sent}&session_state={session_state}

Response:
HTTP/1.1 302 Found
Location: /
Server: Microsoft-IIS/10.0
Set-Cookie: ...
```

---

# References

- Azure AD token reference: https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-token-and-claims
- OpenId Connect Flow: https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-openid-connect-code
- OAuth 2.0 Azure Implementation Flow: https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code
- Applications with Azure Active Directory: https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications
- Application roles: https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/app-roles
- Code Samples: https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-code-samples, https://azure.microsoft.com/en-gb/resources/samples/active-directory-dotnet-webapp-roleclaims/