# Calling the Microsoft Graph from your Teams application - Part 2: Deep dive

Microsoft Teams applications almost always need to call the Graph API, yet it's not as easy as just calling a REST service. Most of the complexity has to do with getting an access token, which is required on every Graph call to establish what, if anything, the caller is authorized to do. 

Getting the access token requires an understanding of Teams, Azure AD, Graph, and sometimes other components like the SharePoint Framework or Bot Framework, yet each of these is documented separately and each one assumes the reader knows all the others. I literally get questions every day from frustrated developers trying to figure this out! (Yesterday I got 3!) In 2 1/2 years of Teams app development, this by far the most common source of difficulties.

I wrote these articles hoping they'll assist developers in calling the Graph from Microsoft Teams. They're also companions for my talk, "Calling Microsoft Graph from your Teams Application", at the [PnP Virtual Conference 2020](https://pnp.github.io/pnpconference/).

1. Introduction (this article)
2. [Deep dive concepts](article2.md) - Deep dive concepts (optional)
3. [Calling Graph from a Teams tab](article3.md)
4. [Calling Graph from a Teams bot or messaging extension](article4.md)

The first article is intended to explain the basics which anyone should understand before embarking on a Teams project that will call Microsoft Graph. This article is an optional deep dive that will go into more detail, either for the curious or to help in troubleshooting. The articles which follow target specific scenarios, such as calling the Graph from a tab or bot in Teams.

Sections in this article:

 * What is Azure Active Directory?
 * What is not Azure Active Directory? (Similarly named products that may lead to confusion)
 * Azure AD Tenants
 * App Registration and Permission
 * Consents and Service Principals
 * Azure AD v1.0 vs. v2.0
 * Terminology Review
 * Troubleshooting steps

## What is Azure Active Directory?

Azure Active Directory (Azure AD) is the identity service used by Microsoft 365 and a number of other Microsoft cloud services. When you log into Microsoft Teams, Microsoft Outlook, SharePoint Online, or other Microsoft 365 services, you are logging into Azure AD. 

When your application wants to call a service that's secured by Azure AD, it needs to obtain an access token from Azure AD. The service should validate Azure AD's digital signature within the token and read "claims" that establish your permission to use the service. Microsoft Graph is such a service, and it won't work unless you include an access token in the HTTP header on every REST call.

You may find yourself wanting to look inside these tokens, usually during troubleshooting. They conform to the [JSON Web Token standard](https://tools.ietf.org/html/rfc7519); you can read all about it and decode tokens at [https://jwt.io](https://jwt.io). JWT tokens are digitally signed but not encrypted, so they should only be sent over secured (encrypted) channels such as https. For that reason, all Azure AD applications should use https and never http.

## What is not Azure AD?

Be sure not to confuse Azure AD with other, similarly named services! **Azure AD is NOT the same as:**

 * [Active Directory](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview): On its own, "Active Directory" refers to the older, on-premises identity service, usually Active Directory Domain Services. There are other related services such as [Active Directory Federation Services](https://docs.microsoft.com/en-us/windows-server/identity/active-directory-federation-services) and [Active Directory Rights Management Service](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831364(v=ws.11)). These services an work together with Azure AD but for the purposes of a Teams developer, you can pretty much leave them to the IT pros and infrastructure experts who will sync and federate them with Azure AD. By the time you get to Teams, your user has signed into Azure AD. Move on folks, nothing to see here.
 * [Azure Access Control Service](https://docs.microsoft.com/en-us/previous-versions/azure/azure-services/hh147631(v=azure.100)) (Azure ACS): This service predates Azure AD and is still used by [SharePoint Add-ins](https://developer.microsoft.com/en-us/office/blogs/impact-of-azure-access-control-deprecation-for-sharepoint-add-ins/), but has otherwise been [retired](https://azure.microsoft.com/en-us/blog/one-month-retirement-notice-access-control-service/). 
 * [Azure AD B2C](https://azure.microsoft.com/en-us/services/active-directory/external-identities/b2c/): Despite the similar name, this is a completely different identity service used to federate with consumer identities such as Google and Facebook. It doesn't work with Microsoft 365 identities and can't be used to obtain an access token for calling the Graph.
 * Older versions of Azure AD came in two flavors, v1 and v2. In May, 2019 these were consolidated, however there are still two endpoints (more on this later). Documentation newer than May 2019 will avoid the older, more separate, versions.

When you're looking for documentation or samples, make sure you're looking at an up-to-date Azure AD scenario and not one of the other, similarly named services.


## Azure AD Tenants

Each instance of Azure AD is called a _tenant_ and has a unique tenant ID (GUID) and DNS name (like contoso.onmicrosoft.com). Each Microsoft 365 subscription has exactly one Azure AD tenant.

![Azure AD Tenants](CallGraphFromTeams-005.png)

Tenants are sometimes called "directories" in the UI and documentation; an Azure AD Directory and Tenant are the same thing. Also, a Directory ID and Tenant ID are the same globally unique identifier (GUID).

While only one Microsoft 365 subscription can be associated with an Azure AD tenant, several Azure subscriptions can be, and it's possible to change an Azure subscription to associate with a different Azure AD tenant. (Check the "Subscriptions" panel in the Azure portal.) This is important because Azure AD will only be able to generate access tokens if:

 - The target application is in the same tenant, OR, 
 - The target application is marked as multi-tenant

In the case of the Microsoft Graph, you don't need to worry about this because Graph is already registered in Microsoft's tenant and is multi-tenant. But you may want to call your own services as well; in this case they need to be registered in your tenant or marked as multi-tenant.

## App Registration and Permission

In every Graph scenario you'll have at least two applications: your application and the Graph, which is itself an Azure AD application. If you want to call the Graph, you need to register an application and request permission for your application to do things with the Graph, such as reading the user's calendar, or maybe every user's calendar. 

> The application you're calling - Graph in this case - is sometimes called a _resource_ or _audience_. In the Azure documentation is usually called a resource; in the JWT it's in a claim called "aud," which is short for audience. The permission - Calendar.Read in this case - is sometimes called a _scope_. These are defined in the [OAuth 2.0 specification](https://tools.ietf.org/html/rfc6749), which is the authorization framework used by Azure AD.

You can register your Azure AD application in the Azure AD section of Azure portal or the Office 365 Admin Center. Navigate to the "App registration" section. This is where you request the initial [permissions](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-permissions-and-consent) for your application. (Applications using the v2.0 endpoint can also request permission at runtime; this is called "dynamic consent". See the section which follows on v1.0 vs. v2.0 endpoints.)

Azure AD supports two kinds of permissions:

  * **User delegated** permissions allow an application to act _on behalf of a user_. Therefore, the effective permission is the intersection of the user's permission and the application's permission. For example, suppose Alice has access to files A, B, and C, and the application has Files.Read permission. When Alice runs the application, it can read files A, B, and C.
  * **Application** permissions allow an application to act _on its own behalf_. Therefore the effective permission is generally the application's permission across the whole tenant. This is often used for daemon services or to elevate permissions beyond what the user can do. If an application has Files.Read permission, it can read every file in the entire Microsoft 365 tenant.

In the Azure AD portal, you can see both delegated and application permissions for the Microsoft Graph.
Notice that there's a column for "Administrative Consent Required". If this field is blank, then users can consent for themselves; if it's marked "yes", then a tenant administrator has to grant consent.

![Requesting Graph permissions](CallGraphFromTeams-006.png)

User delegated permissions always require the user to prove their identity (log in). In Azure AD this is usually done via some kind of web browser - either a pop-up or hosted browser. This prevents applications from ever touching the user's password or other secrets, and allows Azure AD to enforce policies like multi-factor authentication and conditional access. It also allows Azure AD to prompt the user for any missing consents.

In most cases, applications have to prove their identity as well. This is done using an _application secret_ (a secret string generated by Azure AD) or a digital certificate. There is no way to keep a secret in a web browser, so this is the exception: apps running in the browser (such as Teams tabs) can't prove their identity, and this limits the permissions they can have, and they can never have application permissions!  This is discussed further for each scenario. 

In Azure, the best place to store application secrets and certificates containing private keys is [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/).

### Consents and Service Principals

When someone - a user or admin - consents to an application's request for permission, Azure AD creates a _service principal_. (These are called "Enterprise Applications" in the Azure AD portal.) Your service principals may be for apps registered in your tenant, or in the case of a multi-tenant application, for some other tenant. If you logged into the Graph Explorer, for example, you'll see it under Enterprise Applications because you had to consent, even though the app registration is in Microsoft's tenant. If you want to revoke consent, the easiest way is to simply delete the app's service principal under enterprise applications; this will remove any consents that have been granted in that tenant.

In a single tenant application, such as one written for internal use by an enterprise, both the app registration and service principal are in the same tenant. In a multi-tenant application, such as an ISV or SaaS application, the app is registered in the ISV or SaaS provider's tenant and there is a service principal in every tenant that has consented to give the app some permission. This allows ISVs to securely request the list of permissions they need, and each of their customers the ability to consent or decline those permissions.

> Want to see what applications users and admins have consented to? It's under "Enterprise Applications" in the Azure AD portal. Each one has a Permissions page that has two tabs, one for Admin consents and one for User consents; you can see every detail.

> For example, the Graph Explorer is an application registered in Microsoft's tenant so you won't see it on your list of App Registrations. But if you consented to give the Graph Explorer any permission in your tenant, there will be a Service Principal for it in your Azure AD portal under Enterprise Applications.

![Azure AD permissions granted to Graph Explorer](CallGraphFromTeams-007.png)

### Azure AD v1.0 and v2.0 endpoints

Azure AD supports two "endpoints" - literally two sets of URLs for accessing its services; they are called v1.0 and v2.0. The tokens from these services are not interchangable; fortunately the Graph API supports both.

The v2 [endpoints have additional features](https://docs.microsoft.com/en-us/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison), such as the ability to dynamically consent to permissions and to log in Microsoft personal accounts. However [sometimes you need to use the v1 endpoint](https://docs.microsoft.com/en-us/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#limitations), such as:

 * when calling an application that can only understand v1 tokens, such as an Azure Function that's secured using the built-in [App Services authentication (sometimes called "easy auth")](https://docs.microsoft.com/en-us/azure/app-service/overview-authentication-authorization#:~:text=Azure%20App%20Service%20provides%20built-in%20authentication%20and%20authorization,helps%20simplify%20authentication%20and%20authorization%20for%20your%20app.)
 * when using an older client-side library such as the [Active Directory Authentication Library (ADAL)](https://docs.microsoft.com/en-us/azure/active-directory/azuread-dev/active-directory-authentication-libraries).

In some scenarios, you'll need to know the endpoint URL. You can find it in the Azure AD portal under App Registrations - not under a specific app registration, but in the App Registrations panel; it's the same for all the apps in your tenant.

![Azure AD endpoints](CallGraphFromTeams-008.png)

Notice that both v1 and v2 endpoints are shown. The OAuth 2.0 token endpoint is the one you need to get an access token; in some scenarios you'll also need the authorization endpoint.

## OAuth 2.0 Flows

[OAuth 2.0](https://oauth.net/2/) is the protocol used by Azure AD. OAuth distinguishes between [public and confidential clients](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-client-applications), where public clients can't be trusted with a secret and confidential clients can. For example, a web browser or mobile app are considered public clients; a web server is considered a confidential client.

OAuth defines several protocols which are chosen depending on the type of client in use. These protocols are often called "grant flows" or simply "flows", and each leads to a specific ["grant type"](https://oauth.net/2/grant-types/).

Here are the Azure AD grant flows you're likely to encounter as a Teams developer; a full list can be found [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-app-types)

| Name | When to use | Permission type | Registration must include |
|----|----|----|---|
| [Implicit](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-implicit-grant-flow) | Code in a browser calls a web service (legacy) | User delegated | Redirect URI, implicit flow checkboxes |
| [Authorization Code PKCE Flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow) | Code in a public client calls a web service (preferred, but only available for web browsers beginning in May 2020) | User delegated | Redirect URI |
| [On Behalf Of](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) | A web service exchanges a token received from a client for a new token so it can call on behalf of a user | User delegated | App secret or certificate |
| [Client Credentials](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) | Client passes app ID and app secret or certificate to Azure AD | Application | App secret or certificate |

Some of the fields in the app registration are optional because they're not used by all the flows; this is shown in the 4th column.

 * One or more Redirect URIs may be registered; this is used in all browser-based flows, which return authorization codes or tokens via a browser redirect. Registering the valid URLs prevents a hacker from having a token or authorization code sent to an unintended web page.
 * One or more App Secrets (sometimes called Client secrets) or Certificates may be registered; this is used in any [confidential client]() such as in a secured web server and helps to prove the application's identity to Azure AD. Web browsers and mobile apps are considered public clients and can't be trusted with a secret; hence those devices can't use flows that require an app secret.

### Client Libraries

It is certainly possible to acquire an access token from Azure AD using only REST calls, but there are also client-side libraries available.

 * The [Microsoft Authentication Library (MSAL)](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-overview) uses the new v2.0 endpoint and is the preferred solution. It supports a number of languages and environments including .NET, Android, iOS and macOS, Java, Python, and browser-based JavaScript. A new version, [MSAL 2.0](https://developer.microsoft.com/en-us/microsoft-365/blogs/msal-js-2-0-supports-authorization-code-flow-is-now-generally-available/), allows use of Auth Code flow from a web browser, which is more secure than the previously used Implicit Flow.

 * The older [Azure Active Directory Authentication Library (ADAL)](https://docs.microsoft.com/en-us/azure/active-directory/azuread-dev/active-directory-authentication-libraries) (deprecated and soon to be unsupported) uses the v1.0 endpoint.

For a detailed comparison of the ADAL and MSAL libraries, [see this article](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-compare-msal-js-and-adal-js).

NOTE: MSAL does not currently support server-side NodeJS applications, which leaves developers to use the old ADAL (knowing it will [go out of support in a couple years](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-migration#frequently-asked-questions-faq)), or call the Azure AD REST endpoints manually.

Just to clarify the version numbers of the endpoints and libraries, here's a quick reference:
 
  | Library | Azure AD Endpoint | OAuth version |
  |----|----|---|
  | ADAL (deprecated) | v1.0 | 2.0 |
  | MSAL 1.0 | v2.0 | 2.0 |
  | MSAL 2.0 | v2.0, supports one new protocol not found in MSAL 1.0 | 2.0 |

  NOTE: I think that MSAL can call the v1.0 endpoint as well in order to acquire an access token for calling an old, v1 endpoint such as an Azure App Service or Azure Function secured by App Service Authentication.

  * [This page](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-overview#differences-between-adal-and-msal) says, "Active Directory Authentication Library (ADAL) integrates with the Azure AD for developers (v1.0) endpoint, where MSAL integrates with the Microsoft identity platform (v2.0) endpoint" which implies that MSAL does not work with the v1.0 endpoint.
  * [This page](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-net-migration#scopes-for-a-web-api-accepting-v10-tokens) says, "It's also possible in MSAL.NET to access v1.0 resources. See details in Scopes for a v1.0 application," and provides instructions for MSAL.NET

  I am seeking clarification on this.




## Terminology review

The terminology isn't super consistent, so here's a summary just to be clear.

| term | synonyms | meaning |
|---|---|---|
| Tenant | Directory | An instance of an Azure AD directory |
| Tenant ID | Directory ID | A globally unique identifier (GUID) that identifies a tenant |
| Application ID | App ID, Client ID | A GUID that identifies an application registered in Azure AD |
| Application Secret | App secret, Client secret | A string that is used to authenticate an application's identity |
| Application Registration | App registration | An entry in Azure AD representing an application |
| Service Principal | Enterprise application | An entry in Azure AD representing administrative and user consents for an application. The service principal is in the tenant which has consented; the application registration may be in a different tenant if it's marked as multi-tenant |

## Troubleshooting Steps

All this complexity means maybe you won't get it all right the first time. (If you do, take a victory lap, you might be the first!) Yet troubleshooting can be tricky, especially since error information is often limited so as to not give clues to hackers. A "401 Forbidden" error is sometimes all you have to go on!

For that reason, I've compiled this handy list of troubleshooting suggestions.

1. Always start in [Postman](https://www.postman.com/). The idea is to divide and conquer; if it doesn't work in Postman, there's something wrong with your app registration, the permissions you've requested, or the consent that has been granted. If it works in Postman but not your app, then there's a problem with your application code.

2. Check the details

   a. is the Tenant ID correct? (Tenant ID and Directory ID are the same.) In a single-tenant app, the tenant ID is part of the Azure AD endpoint URLs; is it correct there too?

   a. is the App ID correct? (App ID, Application ID, and Client ID are the same)

   b. if there is an App Secret, is that correct?

   c. if there is a Redirect URL, is that exactly correct? For example if you registered your app with http://contoso.com and your app is at https://contoso.com, it won't match!

3. Are permissions correct and have they been consented to? Try a simpler (less privileged) Graph call and see if that works. Double check the documentation and be sure the scope you requested is the same as the call you're using. Check the service principal for your app to be sure the permission was granted.

4. In your Auth URL, you need to specify the resource you're accessing, such as ?resource=https://graph.microsoft.com (or for your own app, ?resource=your app ID). Is that correct?

5. If you're using Implicit flow, did you enable that in your Azure AD app registration?

6. If you're using Client Credentials flow, do you have App Permissions? If you're using anything other than Client Credentials flow, do you have User Delegated Permissions?

