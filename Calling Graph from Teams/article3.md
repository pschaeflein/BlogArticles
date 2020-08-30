# Calling the Microsoft Graph from your Teams application - Part 3: Calling Graph from a Teams tab

Microsoft Teams applications almost always need to call the Graph API, yet it's not as easy as just calling a REST service. Most of the complexity has to do with getting an Azure AD access token, which is required on every Graph call to establish what, if anything, the caller is authorized to do.

Getting the access token requires an understanding of Teams, Azure AD, Graph, and sometimes other components like the SharePoint Framework or Bot Framework, yet each of these is documented separately and each one assumes the reader knows all the others. I literally get questions every day from frustrated developers trying to figure this out! (Yesterday I got 3!) In 2 1/2 years of Teams app development, this by far the most common source of difficulties.

I wrote these articles hoping they'll assist developers in calling the Graph from Microsoft Teams. They're also companions for my talk, "Calling Microsoft Graph from your Teams Application", at the [PnP Virtual Conference 2020](https://pnp.github.io/pnpconference/).

1. [Introduction](article1.md)
2. [Deep dive concepts](article2.md) - Deep dive concepts (optional)
3. [Calling Graph from a Teams tab](article3.md) (this article)
4. [Calling Graph from a Teams bot or messaging extension](article4.md)

This article will explain the options for building tabs for Microsoft Teams which directly call the Microsoft Graph.
It presents four options:

|  | Option | Pros and Cons |
|---|---|---|
| 1 | [Azure AD SSO](#) - Single sign-on |  Excellent user experience; host anywhere; requires custom web service |
| 2 | [SharePoint Framework](#) - SharePoint Framework | Excellent user experience; requires SharePoint Online hosting; permission shared w/all 3rd party web parts in tenant |
| 3 | [Pop-up with MSAL 2.0](#) - Teams pop-up using MSAL 2.0 with Auth Code PKCE flow | Works on static web sites w/no server side assistance; users see some pop-ups  |
| 4 | [Microsoft Graph Toolkit](#) - Microsoft Graph toolkit / Teams pop-up using Implicit flow | Easy development; host anywhere; works on static sites; users will see pop-ups; Implicit flow is less secure than SSO or Auth Code PKCE  |

## 1. Azure AD SSO

This approach, which became Generally Available at Build 2020, provides an extremely seamless user experience. It is documented [here](https://docs.microsoft.com/en-us/microsoftteams/platform/tabs/how-to/authentication/auth-aad-sso). 

At first this looks really easy! There's a [function](https://docs.microsoft.com/en-us/javascript/api/@microsoft/teams-js/microsoftteams.authentication?view=msteams-client-js-latest#getauthtoken-authtokenrequest-) in the [Teams JavaScript client SDK](https://docs.microsoft.com/en-us/javascript/api/overview/msteams-client?view=msteams-client-js-latest) that just gives you an access token:

~~~javascript
microsoftTeams.authentication.getAuthToken({
    successCallback: (result) => {
        display(result)
        resolve(result);
    },
    failureCallback: function (error) {
        reject("Error getting token: " + error);
    }
});
~~~

That is really easy! But unfortunately it's [really limited](https://docs.microsoft.com/en-us/javascript/api/@microsoft/teams-js/microsoftteams.authentication?view=msteams-client-js-latest#getauthtoken-authtokenrequest-); if you wanted to do the [simple Graph API call to read the user's information](https://docs.microsoft.com/en-us/graph/api/user-get) (https://graph.microsoft.com/v1.0/me/), you need user.read permission, which is not included here. To do that, you need to implement a web service which uses the On Behalf Of flow to trade this access token for one that includes the permissions you're looking for. It's not a ton of code, but you do need to stand up a web service to do it.

Then there's the issue of consent. There are really two choices here: either get an administrator to consent on behalf of all users during setup, or support a pop-up that allows the user to consent at runtime. 
This pop-up uses the same mechanism as the older non-SSO approach (approach #3).

You can find a full sample with detailed instructions [here](https://github.com/OfficeDev/msteams-tabs-sso-sample-nodejs).

> NOTE: The On Behalf Of flow used in the web service requires use of a Client Secret to prove the application's identity. Secure storage of the client secret is extremely important; this is easily handled in Azure by storing the secret in [Azure Keyvault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview) and using an app setting that contains a [Keyvault Reference](https://docs.microsoft.com/en-us/azure/app-service/app-service-key-vault-references). 

## 2. SharePoint Framework

If you host your tab in SharePoint, you don't have to worry about getting an access token; SharePoint handles this for you! However at the time of this writing, there is a caveat: any permission you give your tab is also given to every 3rd party web part in your tenant's SharePoint installation. SharePoint provides an isolated mode for web parts which addresses this, but it currently doesn't work with Teams tabs.

Begin by [creating a Teams tab with SharePoint Framework](https://docs.microsoft.com/en-us/sharepoint/dev/spfx/web-parts/get-started/using-web-part-as-ms-teams-tab). Then specify the resource and permission you need in `config/package-solution.json`. For example, to read the user's email you would include:

~~~json
"webApiPermissionRequests": [
    {
    "resource": "Microsoft Graph",
    "scope": "Mail.Read"
    }
],
~~~

When the web part is installed, an administrator will need to approve the permission in the [API access](https://docs.microsoft.com/en-us/sharepoint/api-access) screen in SharePoint online administration.

Calling the Microsoft Graph is really simple since SharePoint Framework takes care of it for you.

~~~typescript
this.context.msGraphClientFactory.getClient()
.then((client) => {
client.api("me/mailFolders/inbox/messages")
    .get((error, response) => {
    // check the error, use the response
    });
});
~~~

A full sample is available [here](https://github.com/pnp/teams-dev-samples/tree/master/samples/tab-aad-spfx).

## 3. Teams pop-up with MSAL 2.0

Azure AD and many other authorization services don't work in an IFrame, and Teams tabs are IFrames. To solve this, Teams provides a browser pop-up and the ability to send information between the pop-up and your tab.

One advantage of this is that it works with any browser-based authorization service. However in this case, we want to call the Microsoft Graph, so we'll be calling Azure AD. From Azure AD's perspective, the pop-up window is a fine place to log the user in, and the access token it provides can include any user delegated access without the need for an On Behalf Of token exchange. That means no server-side code is required, so your solution could be hosted in any static site, such as [Azure Static Websites](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website-host) or [Github Pages](https://pages.github.com/).

The sample [here](https://github.com/pnp/teams-dev-samples/tree/master/samples/tab-aad-msal2) uses the new MSAL 2.0 library, which is more secure than its predecessors due to use of the Auth Code PKCE grant flow. In order to get an access token, the tab calls the [Teams JavaScript client SDK](https://docs.microsoft.com/en-us/javascript/api/overview/msteams-client?view=msteams-client-js-latest) to launch the pop-up and get data back.

~~~javascript
microsoftTeams.authentication.authenticate({
    url: window.location.origin + "/TeamClock/#teamsauthpopup",
    width: 600,
    height: 535,
    successCallback: (response) => {
        const { username, accessToken, expiresOn } =
            JSON.parse(response);
        this.authState = { username, accessToken, expiresOn };
        resolve(accessToken);
    },
    failureCallback: (reason) => {
        reject(reason);
    }
});
~~~

Ironically the `authenticate` call in the `authentication` namespace doesn't do authentication! It only launches a pop-up; your app needs to handle the authentication and return the resulting access code and any other relevant information. In this sample, it returns the username and expiration time, so the app can request a new access token when needed.

The code inside the pop-up really consists of three parts:

### a. Checking for redirects

With the older Implicit flow, Azure AD returns the access token on the URL. A safer approach, used in Auth Code PKCE and MSAL 2.0, returns an authorization code on the URL, which is then exchanged for the access token. Either way, your code needs to handle the redirect back from Azure AD to get this data.

~~~javascript
async init() {

    let response = await this.msalClient.handleRedirectPromise();
    if (response != null && response.account.username) {
        return true;
    } else {
        const accounts = this.msalClient.getAllAccounts();
        if (accounts === null || accounts.length === 0) {
            return false;
        } else if (accounts.length > 1) {
            throw new Error("ERROR: Multiple accounts are logged in");
        } else if (accounts.length === 1) {
            return true;
        }
    }
}
~~~

The handleRedirectPromise() call checks the browser's [window object](https://www.w3schools.com/jsref/obj_window.asp) for a URL containing an auth code; if found, it handles the auth code and removes it from the URL; if not, it does nothing. When this is done, the MSAL 2.0 client object contains the information needed to obtain an access token.

### b. Logging In

The user is logged in using this call:

~~~javascript
this.request = {
    scopes: ["user.read"]
}
this.msalClient.loginRedirect(this.request);
~~~

This is one of two options from MSAL; the other is `loginPopup()`. In this case, Teams needs to handle the popup so the `loginRedirect()` call is used; this redirects the whole web page (in the Teams popup) to Azure AD to log the user in. The response contains the auth code and is handled as shown above.

### c. Acquiring an access token

With any luck, the scopes required were specified in the login request, and the access token is already there inside the MSAL 2.0 client object; it can be retrieved using the `acquireTokenSilent()` function. However, if additional permission is needed, the code needs to go back to Azure AD; this is handled by `acquireTokenRedirect()` (or `acquireTokenPopup()` for non-Teams applications). 

Here is a snippet from the sample which tries to get the access token silently but, if it's not there (either because additional permission was needed or the token has expired), it does another redirect back to Azure AD.

~~~javascript
try {
    let resp = await this.msalClient.acquireTokenSilent(this.request);
    if (resp && resp.accessToken) {
        return {
            username: this.getUsername(),
            accessToken: resp.accessToken,
            expiresOn: (new Date(resp.expiresOn)).getTime() 
        }
    } else {
        return null;
    }
}
catch (error) {
    if (error instanceof msal.InteractionRequiredAuthError) {
        console.warn("Silent token acquisition failed; acquiring token using redirect");
        this.msalClient.acquireTokenRedirect(this.request);
    } else {
        throw (error);
    }
}
~~~

This is all well and good for the Teams desktop and web clients, but the mobile clients currently don't support the Teams pop-up. In the mobile clients, the tab is just a regular web page; hence the MSAL calls can be done directly. The sample handles this by providing [a second version of the tab](https://github.com/pnp/teams-dev-samples/blob/master/samples/tab-aad-msal2/src/components/Web.js) that calls MSAL directly.

You can see how this works in the [app.js](https://github.com/pnp/teams-dev-samples/blob/master/samples/tab-aad-msal2/src/components/App.js) component; it first checks for a redirect and then to see if it's in an IFrame or not. If it is in an IFrame, it renders [Tab.js](https://github.com/pnp/teams-dev-samples/blob/master/samples/tab-aad-msal2/src/components/Tab.js), which uses the Teams popup; if not, it renders [Web.js](https://github.com/pnp/teams-dev-samples/blob/master/samples/tab-aad-msal2/src/components/Web.js), which does not.

### d. Graph Toolkit

The [Microsoft Graph Toolkit](https://docs.microsoft.com/en-us/graph/toolkit/get-started) is an easy-to-use library of web components for calling the Microsoft Graph; it includes an auth component that includes Teams support! The result is an application that uses the pop-up mechanism with an older auth library, so it uses Implicit Flow. This may change in future versions of the Toolkit.

My colleague [Ayça Baş](https://quickbites.dev/) has written a [detailed article](https://dev.to/aycabs/handling-authentication-in-custom-teams-tabs-using-microsoft-graph-toolkit-53i8) which walks you through creating this solution.


The next article in the series will consider calling Graph from Bots and Messaging Extensions.