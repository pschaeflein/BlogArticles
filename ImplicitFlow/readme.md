# Calling an Azure AD protected web service from your Single Page Application

To call the Microsoft Graph API, you need an access token which proves to the Graph that you're a legitimate user and what permission you should have. The instructions sound simple: "acquire an access token", right? But in reality you might as well tell someone to "fly to the moon." Obtaining the right access token is often the most challenging part of calling the Graph API!

This sample aims to demystify the the process of obtaining an Azure AD access token when your code is running in a web browser. It includes four samples: two all-in-one JavaScript samples using the familiar jQuery library (everything is in one file), and two more modern samples written with TypeScript, React, and Webpack. The latter may be helpful if you're writing a full-on single-page application with a bundling system such as Webpack.

<table>
<tr><th>File</th><th>Description</th></tr>
<tr><td>index.1a.html</td><td>v1 endpoint using ADAL with jQuery</td>
<tr><td>index.1b.html</td><td>v1 endpoint using ADAL with TypeScript, Webpack, and React</td>
<tr><td>index.2a.html</td><td>v2 endpoint using MSAL with jQuery</td>
<tr><td>index.2b.html</td><td>v2 endpoint using MSAL with TypeScript, Webpack, and React</td>
</table>

## No secrets in the browser 

The OAuth standard includes several "grant flows" for granting an access token to an application; if you're in a web browser, a typical choice is [implicit flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-implicit-grant-flow) because it doesn't require the browser to handle any secrets like a client secret or application key. There's no way to keep a secret in the web browser where the user can always pop into the developer console and inspect any variable or line of code. So instead of using a secret, implicit flow requires the user to log in from the web browser and to consent (once) to letting the application act on his or her behalf. The resulting access token is limited by the permissions of the user plus the permissions they've consented to.

![Effective permission](2018-11-EffectivePermissions.png)

When you register your application, you'll notice there are "application" and "delegated" permissions available; only the "delegated" permissions will work with the implicit flow. In addition, because implicit flow is less secure than the other OAuth flows, you need to mark the application as allowing implicit flow (the default is to not allow it).

## A tale of two endpoints

Azure AD has two ways of registering and authorizing applications: the old way (via the "v1 endpoint") and the new way (via the "v2 endpoint"). Each one has its own registration and client-side library as well as its own endpoint URL for accessing the service. The choice is yours; [this article](https://docs.microsoft.com/en-us/azure/active-directory/develop/azure-ad-endpoint-comparison) can help you decide. This sample will show both, using the [ADAL.js](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-libraries) library for v1 access and [MSAL (preview)](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-v2-libraries) for v2 access.

## Registering your application

This article won't 