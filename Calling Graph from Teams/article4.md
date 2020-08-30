# Calling the Microsoft Graph from your Teams application - Part 4: Calling Graph from a Teams bot or messaging extension

Microsoft Teams applications almost always need to call the Graph API, yet it's not as easy as just calling a REST service. Most of the complexity has to do with getting an Azure AD access token, which is required on every Graph call to establish what, if anything, the caller is authorized to do.

Getting the access token requires an understanding of Teams, Azure AD, Graph, and sometimes other components like the SharePoint Framework or Bot Framework, yet each of these is documented separately and each one assumes the reader knows all the others. I literally get questions every day from frustrated developers trying to figure this out! (Yesterday I got 3!) In 2 1/2 years of Teams app development, this by far the most common source of difficulties.

I wrote these articles hoping they'll assist developers in calling the Graph from Microsoft Teams. They're also companions for my talk, "Calling Microsoft Graph from your Teams Application", at the [PnP Virtual Conference 2020](https://pnp.github.io/pnpconference/).

1. [Introduction](article1.md)
2. [Deep dive concepts](article2.md) - Deep dive concepts (optional)
3. [Calling Graph from a Teams tab](article3.md) 
4. [Calling Graph from a Teams bot or messaging extension](article4.md) (this article)

This article will explain the options for building tabs for Microsoft Teams which directly call the Microsoft Graph. Three options are considered, however it will be easier to decide because there's really only one choice per scenario.

|  | Option | Permission granted |
|---|---|---|
| 1 | [Bot with Auth Dialog](#)| User delegated |
| 2 | [Messaging Extension](#) | User delegated |
| 3 | [Bot or Messaging Extension with Client Credentials Flow](#) | Application  |

Recall that Bots and Messaging extensions are implemented as web services; Teams calls your web service as specified in your app manifest file.

### 1. Bot with Auth Dialog

As you may be aware by now, in most cases Azure AD needs to interact with the user via a browser in order to log them in and get any needed consents. The mechanism for doing this is an [Auth Dialog](https://docs.microsoft.com/en-us/microsoftteams/platform/bots/how-to/authentication/add-authentication). Note that this is part of the [Microsoft Bot Framework](https://dev.botframework.com/).

A sample solution which calls the Graph to get a list of email [can be found here](https://github.com/pnp/teams-dev-samples/tree/master/samples/bot-call-graph-as-user) along with setup instructions.

NOTE: This write-up is for Bot Framework 4.0 or greater. Earlier versions of the Bot Framework may differ.

Bots in the Microsoft Bot Framework support the concept of _dialogs_, which are a way to manage multiple interactions with a Bot, such as logging a user in and then calling the Graph API. To log the user in and acquire an access token, a special prebuilt Auth Dialog is provided.

The auth dialog is defined like this in the [main dialog](https://github.com/pnp/teams-dev-samples/blob/master/samples/bot-call-graph-as-user/dialogs/mainDialog.js):

~~~javascript
this.addDialog(new OAuthPrompt(OAUTH_PROMPT, {
    connectionName: process.env.connectionName,
    text: 'Please Sign In',
    title: 'Sign In',
    timeout: 300000
}));
~~~

The main dialog includes two steps relating to the auth dialog. First, a prompt step displays the auth dialog to the user.

~~~javascript
async promptStep(stepContext) {
    return await stepContext.beginDialog(OAUTH_PROMPT);
}
~~~

Control then passes to the auth dialog. If the user is already logged in and we have a valid access token,  it does nothing. If not, the Auth dialog displays a pop-up window showing Azure AD's login page. Azure AD may have a cookie that indicates the user has already logged in; in this case the prompt will go by in a flash. If necessary, Azure AD will display the login page and request any needed consents.

The result is handled in the login step that follows.

~~~javascript
async loginStep(stepContext) {
    const tokenResponse = stepContext.result;
    if (tokenResponse) {
        await stepContext.context.sendActivity('You are now logged in.');
        // The token response contains the access token
        // Return the next dialog in the chain
    }
    // Error - end the dialog
    await stepContext.context.sendActivity('Login was not successful please try again.');
    return await stepContext.endDialog();
}
~~~

You'd think that would be the end of the story - use the access token! And indeed that is possible but the sample is trying to be realistic and capturing some additional information from the user first; in this case it presents a choice prompt so the user can view the access token or call the Graph. This points out an important practice: we really need to check again before calling the Graph. For example, what if the user sees that prompt and then goes to lunch, and stays away long enough for the token to expire? For this reason, the code calls the Auth Dialog again just before calling the Graph each time; this normally will do nothing and the user won't notice, but if the access token is expired, the user will be prompted again.

### 2. Messaging Extension

[Markus Moeller](https://twitter.com/Moeller2_0) has written an excellent set of articles about this, including detailed samples. Rather than duplcate his work, I'll just refer to it here.

[A Microsoft Teams Messaging Extension with Authentication and access to Microsoft Graph](https://mmsharepoint.wordpress.com/2020/07/03/a-microsoft-teams-messaging-extension-with-authentication-and-access-to-microsoft-graph-i/)

### 3. Bot or Messaging Extension with Client Credentials Flow

Since bots and messaging extensions run server side, they can use Application permissions as well as User Delegated. This allows access across the Microsoft 365 tenant without being subject to the user's permissions (except for the case of [Resource-specific Consent](https://docs.microsoft.com/en-us/microsoftteams/resource-specific-consent), which is out of the scope of this article series). 

Application permissions require the use of [Client Credentials Flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow). I'm hoping to create a sample for this soon.

One thing that makes this tricky in Node.js solutions is that, at the time of this writing, the MSAL library does not support server-side Node.js calls, leaving developers to build their own or to use the ADAL library, [which will only be supported until June 30, 2022.](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/update-your-applications-to-use-microsoft-authentication-library/ba-p/1257363). That said, here's some code that I wrote on another project using ADAL:

~~~javascript
var adal = require('adal-node');
var settings = require('./settings');
var getSecret = require('./getSecret');

module.exports = function getToken(context) {

  return new Promise((resolve, reject) => {

    // Get the ADAL client
    const authContext = new adal.AuthenticationContext(
      settings().AUTH_URL + settings().TENANT);
 
    // Get the client secret
    getSecret(context, settings().CLIENT_SECRET, 
                       settings().CLIENT_SECRET_NAME)
    .then((secret) => {
      // If here we have the client secret, go ahead and get
      // the token
      authContext.acquireTokenWithClientCredentials(
        settings().GRAPH_URL, settings().CLIENT_ID, secret, 
        (err, tokenRes) => {
          if (err) {
            reject(err); 
          } else {
            resolve(tokenRes.accessToken);
          }
      });
    })
  });
}
~~~

Secure storage of the client secret is extremely important; this is easily handled in Azure by storing the secret in [Azure Keyvault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview) and using an app setting that contains a [Keyvault Reference](https://docs.microsoft.com/en-us/azure/app-service/app-service-key-vault-references). 