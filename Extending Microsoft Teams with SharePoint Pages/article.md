# Extending Microsoft Teams with SharePoint Pages

## Any SharePoint page can be a Teams Application

At the recent Teams Airlift in Bellevue, WA, [Karuana Gatimu](https://github.com/karuanag) demonstrated the new [Microsoft 365 Learning Pathways](https://docs.microsoft.com/en-us/office365/customlearning/) solution integrated into the Teams user interface. If you're not familiar, Learning Pathways is a customizable end-user training portal you can host directly in SharePoint Online. Microsoft provides a large base of training content, which is automatically updated as the M365 service changes, and the in-tenant SharePoint site stores enterprise-specific  help, support, and community content. Given the price (free) it's hard to imagine why any organization wouldn't deploy this!

But this isn't an article about Learning Pathways. The power in Karuana's demo went beyond that, as she showed how to share the Learning Pathways site from Microsoft Teams, available from the top of the Teams sidebar. (The Microsoft logo could be replaced with any icon you wish.) This encourages everyone to discover and access the training any time they're in Teams.

![Learning Pathways in Teams](LearningPathwaysInTeams.png)
_Learning Pathways training portal is right at the top of the Teams sidebar_

It turns out this technique can be used for any SharePoint page. If you can edit a site page or format a list column, you can make a Teams application that users can access from the sidebar, as a personal application, or even as a tab within Teams channels. Anything you can do in (modern) SharePoint is up for grabs - Power Apps too - sky's the limit. It's pretty easy, and this article will show you how.

## A bit about Teams Applications

To get started, you'll need to understand a little about Microsoft Teams applications. Teams applications can include custom tabs, bots, UI extensions ("messaging extensions"), and more. The Learning Pathways sample is a small Teams app containing some static tabs that display the Learning Pathways SharePoint pages; this is installed in the tenant's enterprise app catalog and pinned to the sidebar for all users.

Teams apps don't actually run in Teams, they just look like they do. Teams cleverly stitches apps into its UI, either by exchanging web service calls or, in the case of tabs, displaying web pages. A tab is just a web page displayed in the native Teams application or in a browser IFrame. [The "website" tab in Teams](http://davidgiard.com/2019/01/27/AddingAWebsiteTabToAMicrosoftTeamsChannel.aspx) is an obvious example.

A Teams app package is a just zip file containing 3 files: two icons (large and small) and a file called manifest.json. The manifest.json file tells Teams where the various parts of the app are located; in this case, it contains the URL's of the Learning Pathways site pages. [(View it here)](https://github.com/msft-teams/tools/blob/master/getstartedapp/manifest.json). You can create the app package manually, or use another Teams app called [App Studio](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/app-studio-overview) to do it. This article includes some features not yet in App Studio, so we'll be creating the manifest file manually; if you can edit a little JSON and zip it up, you have the necessary skills.

Once you have an app package, you can [upload it](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/apps-upload) for yourself or a specific Team, or [publish it to the enterprise app catalog](https://docs.microsoft.com/en-us/microsoftteams/tenant-apps-catalog-teams) for everyone to use. This is accessible from the Apps icon in the sidebar (1). To view the enterprise app store, click the link marked "Built for (tenant name)" (2).  You can see I only have the Getting Started (Learning Pathways) app in my catalog (3). You can upload an app using the Upload link (4); you'll need to be a tenant admin to upload to your tenant's enterprise app catalog. If App uploads are enabled in your tenant, you can also upload apps for yourself and your Teams using this link.

![Apps in Teams](AppsInTeamsCallouts.png)

You can control who can use the app, and pin it to the sidebar, using [Teams app policies](https://docs.microsoft.com/en-us/microsoftteams/teams-custom-app-policies-and-settings) in the Teams admin portal.

![App policies](AppSetupPolicies2.png).

As you can see I've pinned the Getting Started app to the top of the sidebar using this screen.



References

* [Create an app package for your Microsoft Teams app](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/apps-package)
* [Use built-in and custom tabs in Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/built-in-custom-tabs)