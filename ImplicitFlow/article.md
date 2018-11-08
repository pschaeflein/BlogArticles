# Day ??? - Calling the Microsoft Graph from a single-page application

In day 15??, you learned how to call the Graph API from a .NET command line application. Today you'll learn to do much the same thing from a web browser. There are a couple of big differences:

* The code will be written in JavaScript or TypeScript, because that's what runs in the browser
* We'll have to use a different OAuth 2.0 flow, "Implicit Flow" to get the access token for use with Microsoft Graph, because that's the only option in the browser

Sometimes fewer choices makes life easier, right? That said, you'll still get to choose your flavor:

<table>
<tr><th>File</th><th>Description</th></tr>
<tr><td>index.1a.html</td><td>v1 endpoint using ADAL with jQuery</td>
<tr><td>index.1b.html</td><td>v1 endpoint using ADAL with TypeScript, Webpack, and React</td>
<tr><td>index.2a.html</td><td>v2 endpoint using MSAL with jQuery</td>
<tr><td>index.2b.html</td><td>v2 endpoint using MSAL with TypeScript, Webpack, and React</td>
</table>

The JavaScript examples are really simple, with good old jQuery and all the code on one page. They're actually drawn from elsewhere; the Azure AD V1  sample is from [Julie Turner's](https://twitter.com/jfj1997) awesome article series, [Extending SharePoint with ADAL and the Microsoft Graph API](https://julieturner.net/2017/01/extending-sharepoint-with-adal-and-the-microsoft-graph-api-part-1-the-setup/); the V2 example is from [this Microsoft tutorial.](https://docs.microsoft.com/en-us/azure/active-directory/develop/tutorial-v2-javascript-spa).

The TypeScript examples are new, and are intended for developers building full scale single page applications. They're also instructive because TypeScript made it easy to separate the authentication code from the Graph calls and user interface, so it's a lot easier to see what's going on.

## Updating your App Registrations

This demo builds on articles 9?? and 10??, where you registered applications in Azure AD v2 and v1 respectively. These applications were used in several other articles. You'll need to update those registrations for this exercise; the sections which follow will walk you through the process.

### Step 1. Add Delegated Permissions

To review, Azure AD has two permission models: application and delegated. Application permissions are like service accounts; they have their own direct access and aren't associated with any particular user. With delegated permissions, the application acts on behalf of a user, and is limited by both the application permissions and the end-user's permissions.

![Effective permission](2018-11-EffectivePermissions.png)

For security reasons, delegated permissions are the only ones allowed when calling directly from a web browser, so we need to add a delegated permission.

#### V1 instructions

Find the application you registered in the Day 10?? article, (or [register a new one](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-v1-add-azure-ad-app) of type Web app / API with the redirect URL http://localhost:8080/index.1a.html). Click Settings, then Required Permissions, and click into the Microsoft Graph API. The application permissions are at the top, and the delegated at the bottom, so scroll way down and make sure you're selecting the delegated one - it can get confusing!

![Permissions screen](./2018-11-AADV1-Permissions.png)

You need to check the "Read all groups" delegated permission (1); you can see the scope, group.read.all, if you hover over it. We'll need the scope in our code. 

Now notice that some of these permissions require an administrator to consent (those marked with a green "Yes") - so what that's saying is that a tenant administrator needs to approve your application's ability to have the permission. All the app permissions and some of the delegated ones require administrator consent. So, after clicking Save (2), click "Grant Permissions" (3) and agree to allow the permissions.

#### V2 Instructions

Return to the application you created on Day 9?? (or [create a new one](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-v2-register-an-app) with your choice of supported account types. Don't worry about Redirect URL yet.)

Click "API Permissions" to open the permissions panel.

![V2 permissions](./2018-11-AADV2-Permissions.png)

Then pick + Add a permission (1), and add the delegated Group.Read.All permission (2). Then, because this scope requires administrative consent, click the button (3) and agree.

### Step 2 - Enable Implicit Flow

The OAuth 2.0 standard includes several "flows" for getting a access token. If you're in a web browser using Azure AD, you need to use [implicit flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-implicit-grant-flow) because it doesn't require the browser to handle any secrets like an app password or key. 

There's no way to keep a secret in the web browser where the user can always pop into the developer console, so it's better not to try. Instead of using a secret, implicit flow requires the user to log in from the web browser and to consent (once) to letting the application act on his or her behalf. 
Since the only security is the user's login, you can only use delegated permissions with implicit flow.

In addition, because implicit flow is less secure than the other OAuth flows, you need to mark the application as allowing implicit flow in Azure AD.

#### V1 Instructions

In old-school Azure AD, you need to edit the manifest JSON to enable implicit flow. Click the "Edit Manifest" button, and set "oauthAllowImplicitFlow" to true (it's false by default).

![Enable Implicit Flow v1](./2018-11-AADV1-ImplicitFlow.png)

Click Save and you're done.

#### V2 Instructions

The shiny new Azure AD takes a giant leap forward with actual checkboxes for implicit flow!

![Enable Implicit Flow v2](./2018-11-AADV2-ImplicitFlow.png)

Check them, click Save, and you're done.

### Step 3 - Add the redirect URLs

Now you might have been wondering about these redirect URLs; with some OAuth flows they're not even used. We'll need them for sure this time! 

Getting an access token isn't as simple as making a web service call because Azure AD might need to interact with the user to log in and consent to the permissions. So the implicit "flow" is to redirect the web browser (either main page or a popup window) right to Azure AD, and let it take over the UI for a while  - or for just for an instant if the user has already logged in and consented. When it's done, Azure AD redirects back to your site with the token in the URL hash (or an error if something went wrong). That URL is called the Redirect or Reply URL.

Redirect URLs need to be registered to prevent a hacker from requesting an access token be sent to some unknown site. So to make the demos work, you'll need to register the redirect URL's.

#### V1 Instructions

In your application, click Settings, then Reply URLs.

![V1 redirect URLs](./2011-11-AADv1-ReplyURLs.png)

Add one for each of the V1 demos: http://localhost:8080/index.1a.html and http://localhost:8080/index.1b.html.

#### V2 Instructions

In your application's overview page, you'll see a link to the Redirect URLs in the right column of information.

![V2 redirect URLs](./2011-11-AADv2-ReplyURLs.png)

Add one for each of the V2 demos: 
http://localhost:8080/index.2a.html and http://localhost:8080/index.2b.html.

    At the time of this writing, the Azure AD App Registrations (Preview) section in the Azure Portal does not allow "." characters in Redirect URL's. You can add them using the old stand-alone [Appliction Registration Portal](https://apps.dev.microsoft.com/).

## Configure the Sample

First, clone or download the sample code from [https://github.com/BobGerman/AADsamples](https://github.com/BobGerman/AADsamples); this sample is in the [ImplicitFlow](https://github.com/BobGerman/AADsamples/tree/master/implicitFlow) folder. You'll need to have [Node] installed to build the TypeScript demos, and to run all of them.

From a command line in the implicitFlow directory, type:

    npm install
    npm install http-server -g

Now edit the code to include your App ID's and other details.

* In index.1a.html, plug in your V1 client ID (the Application ID) and tenant id (like "mytenant.onmicrosoft.com)
* In index.2a.html, plug in your V2 client ID (the Application ID).
* In the src folder, rename or copy constants.sample.ts to constants.ts, and plug in your V1 and V2 application ID's and your tenant id

Demos 2a and 2b require a build step to compile the TypeScript and make a webpack bundle. Type:

    npm run build

## Run the Sample

To start the web browser, type

    http-server

Point your web browser to

    http://localhost:8080

and you can navigate to the various demos.

The V1 JavaScript demo logs the user on and displays a bit of profile information; the V2 JavaScript demo is similar but has an explicit Login/Logout button. The TypeScript demos display a list of all groups in your tenant.

![Running demo 2b](./2018-11-AADV2-Run.png)

## Learning from the Code

Now you have 4 samples to check out, which all do the same thing. They first "log the user in" (validate the user and get an ID token), and then they get an access token that lets them call the Graph API.

If you're most comfortable in JavaScript and jQuery, the JavaScript samples are for you.

index.1a.html shows ADAL and the v1 endpoint. Walking through the code can be a bit tricky because the same page is used to initiate the login and handle redirects after the user is logged in and the token is obtained. Here's the $(document).ready function which runs when the page is loaded:

```javascript
        $(document).ready(function () {
            // Check For & Handle Redirect From AAD After Login or Acquiring Token
            var isCallback = sampleApp.authContext.isCallback(window.location.hash);

            if (isCallback && !sampleApp.authContext.getLoginError()) {
                sampleApp.authContext.handleWindowCallback(window.location.hash);
            } else {
                var user = sampleApp.authContext.getCachedUser();
                if (!user) {
                    //Log in user
                    sampleApp.authContext.login();
                } else {
                    sampleApp.getGraphData();
                }
            }
```

Notice that it checks for a callback - that is, it asks ADAL if the page is running due to a redirect from Azure AD. authContext.handleWindowCallback() takes the token from Azure AD and puts it into local cache. If it needs to log the user in, it calls authContext.login() which _does not return_ - it redirects the browser window to Azure AD! The same thing can happen in the authContext.acquireToken() (not shown in the listing but it's in the code). 

index.2a.html is similar, except it has login and logout buttons so the user can initiate the process. If the user isn't logged in, the page shows the login button (very little JavaScript runs). When the user clicks the button, they are logged in, but this time the Popup version of the login method is used, so the call returns a promise which is resolved when the user logs in via a popup window.

```javascript
function signIn() {
    myMSALObj.loginPopup(applicationConfig.graphScopes).then(function (idToken) {
        //Login Success
        showWelcomeMessage();
        acquireTokenPopupAndCallMSGraph();
    }, function (error) {
        console.log(error);
    });
}
```

With all the UI code, Graph calls, and authentication stuff mixed together, it can be a little hard to follow. So even if you're not a TypeScript developer, you might find it easier to understand.

The TypeScript SPA includes two services, MSGraphService (which calls the Graph) and two implementations of an AuthService (which gets the access token) - one each for V1 and V2. ServiceFactory.ts is a poor person's dependency injection, and configures the MSGraphService to use the V1 or V2 version of AuthService based on the scenario selected.

MSGraphService doesn't have to fuss about authentication, it just asks for a token and moves on:

```typescript
this.authService.getToken()
    .then((token) => {

        fetch(
            `https://graph.microsoft.com/v1.0/groups/?$orderby=displayName`,
            {
                method: "GET",
                mode: "cors",
                cache: "no-cache",
                headers: {
                    "Content-Type": "application/json",
                    "Authorization": "Bearer " + token
                }
            }
        )
        .then((response) => {
            // handle the response
        }
        // et cetera
```

The v1 Auth service creates a new AuthenticationContext, which is the main object in ADAL, and then calls an internal function called ensureLogin() to make sure the user is logged in.

```typescript
private ensureLogin(authContext: AuthenticationContext): boolean {

    var isCallback = authContext.isCallback(window.location.hash);

    if (isCallback && !authContext.getLoginError()) {
        authContext.handleWindowCallback(window.location.hash);
    } else {
        var user = authContext.getCachedUser();
        if (!user) {
            authContext.login();
        } else {
            return true;
        }
    }
    return false;
}
```

It handles the login callback if necessary, which grabs the ID token off the URL hash, caches it, and sets the user state to logged in. If we don't have a callback, it looks for the user in cache. No cache means no user was logged in, so it calls authContext.login() - and again, that call never returns, it redirects to Azure AD.

Assuming ensureLogin() returns (maybe after a redirect or two), the code does the exact same thing to acquire the access token. If no access token is found in cache, it calls authContext.acquireToken() - another call that never returns, it redirects and the code will find the token in cache when Azure AD redirects back to the page.

```typescript
if (this.ensureLogin(authContext)) {

    let cachedToken = authContext.getCachedToken(constants.resourceId);
    if (cachedToken) {
        resolve(cachedToken);
    } else {
        authContext.acquireToken(
            constants.resourceId,
            (error, acquiredToken) => {
                if (error || !acquiredToken) {
                    reject(error);
                } else {
                    resolve(acquiredToken);
                }
            }
        );
    }
} else {
    reject(`Login error: ${authContext.getLoginError()}`);
}
});
```

The V2 version of the Auth service is equally simple. Instead of ADAL's AuthenticationContext, we've got MSAL's UserAgentApplication ... kind of the same thing. This time the login and access token code are even more consise!

```typescript
// Ensure user is logged in
if (!userAgentApp.getUser() ||
    userAgentApp.isCallback(window.location.hash)) {
    userAgentApp.loginRedirect(constants.scopes);
}

// Try to get a token silently
userAgentApp.acquireTokenSilent(constants.scopes)
    .then((accessToken) => {
        resolve(accessToken);
    })
    .catch((error) => {
        console.log(error);
        // If the error is due to a need for user
        // interaction, then redirect to allow it
        if (error.indexOf("consent_required") !== -1 ||
            error.indexOf("interaction_required") !== -1 ||
            error.indexOf("login_required") !== -1) {
            userAgentApp.acquireTokenRedirect(constants.scopes);
        } else {
            reject('Error acquiring token: ' + error);
        }
    });
```

The calls are different, but the logic is pretty much the same. In MSAL, the calls that redirect are clearly marked: loginRedirect() and acquireTokenRedirect(). There are popup versions of both those methods, which you can see in the JavaScript version.

## Conclusion

The simple instructions, "acquire an acess token," might as well be "fly to the moon" for a new Graph developer. Here's hoping your experience was easier than rocket science, and you're now equipped to use it in your single page applications and other browser based solutions!