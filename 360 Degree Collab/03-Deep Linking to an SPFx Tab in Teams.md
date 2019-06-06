# Deep Linking to a SharePoint Framework Tab in Microsoft Tams

This is part 3 of a 3-part article series about building a "360 degree view" mashup for Microsoft Teams using the SharePoint Framework and React. The articles are:

 1. A Teams tab using SharePoint Framework and React displaying "mashup" of related information [in Part 1 of the series](#)
 1. Accessing the hosting Team using the teams context and the Graph API (explained [in Part 2 of the series](#)
 1. Deep linking to a SharePoint Framework tab (this article)

 The series is based on a [sample Teams tab written in SharePoint Framework](https://github.com/SharePoint/sp-dev-fx-webparts/tree/master/samples/react-teams-tab-field-visit-mashup) which displays a mashup of information about customer visits.

![Field Visit Demo](./FieldVisitDemo.png)

## What is a deep link?

In the world of web applications, a deep link is a link that not only selects a web page but also passes data to an application running on that page. For example, a link could open an Excel spreadsheet in a particular Team, rather than opening Teams and Excel in separate pages.

Recall that [the sample application](https://github.com/SharePoint/sp-dev-fx-webparts/tree/master/samples/react-teams-tab-field-visit-mashup) is a SharePoint Framework tab in Teams that displays a mashup of team members' customer visits. A deep link, therefore, needs to work at three levels:

* The browser interprets the URL and fetches a Teams web page
* Teams interprets the URL and opens the correct Team, Channel, and Tab. The tab is running in SharePoint.
* The SharePoint Framework web part interprets the URL and shows the customer visit specified in the link

## Building a deep link for Teams

The deep link format in Teams is:

~~~
https://teams.microsoft.com/l/entity/<appId>/<entityId>?label=Vi32&context=<context>
~~~

where

* `<appId>` is your Teams application ID (from the Teams manifest.json file)
* `<entityId>` specifies the tab. If you were writing a tab from scratch, you'd get to specify this, but SharePoint owns the tab so consider it read only. The value is in the `entityId` property in the [Teams Context](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/tabs/tabs-context)
* `<context>` is a JSON string specifying two properties - the channel ID and a string called subEntityId. This is where you can pass some information to your application code.

For example, there's a text box in the customer visit demo; when the user clicks a button, the text is posted in the channel along with a deep link back to the same customer visit. To show a particular customer visit, I needed to know the user (person doing the visiting) and the customer ID, so that's what I passed in the subEntityId. The code is in the [PostToChannel.tsx](https://github.com/SharePoint/sp-dev-fx-webparts/blob/master/samples/react-teams-tab-field-visit-mashup/src/webparts/fieldVisitTab/components/PostToChannel.tsx) component.

~~~TypeScript
// Build a deep link to the current user tab and customer
const url = encodeURI(
    'https://teams.microsoft.com/l/entity/' +
    this.props.teamsApplicationId + '/' +
    this.props.entityId +
    '?label=Vi32&' +
    'context={"subEntityId": "' +
    this.props.selectedUser + ':' +
    this.props.customerId +
    '", "channelId": "' + this.props.channelId + '"}');
~~~

## Reading the deep link

When someone clicks the deep link, Teams takes care of loading the right channel and tab, with your application inside. All you have to do is read the `subEntityId` property of the Teams context and you're off to the races.

Here's the code in the main web part file, [FieldVisitTabWebPart.ts](https://github.com/SharePoint/sp-dev-fx-webparts/blob/master/samples/react-teams-tab-field-visit-mashup/src/webparts/fieldVisitTab/FieldVisitTabWebPart.ts), which passes all the information into the mashup, a React component called [FieldVisits.tsx](https://github.com/SharePoint/sp-dev-fx-webparts/blob/master/samples/react-teams-tab-field-visit-mashup/src/webparts/fieldVisitTab/components/FieldVisits.tsx). Notice that it reads the entity ID and subEntity ID from the Teams context and passes them into the component.

~~~TypeScript
const element: React.ReactElement<IFieldVisitsProps> = React.createElement(
  FieldVisits,
  {
    // Services
    visitService: visitService,
    weatherService: weatherService,
    mapService: mapService,
    documentService: documentService,
    activityService: activityService,
    conversationService: conversationService,
    photoService: photoService,
    // Team and channel information
    groupName: this.groupName,
    groupId: this.groupId,
    channelId: this.channelId,
    // Deep linking context
    entityId: this.teamsContext ? this.teamsContext.entityId : "",
    subEntityId: this.teamsContext ? this.teamsContext.subEntityId : "",
    // App and user info
    teamsApplicationId: '(your app ID here)',
    currentUserEmail: this.context.pageContext.user.email
  }
);

ReactDom.render(element, this.domElement);
~~~

## Conclusion

Posting deep links into a channel is a great way to connect your tab into the conversation in Teams. Users reach out to their collaborators with the postings, and the deep links bring them right into the same context.

I hope this little blog series has been helpful, and that you enjoy the demo!