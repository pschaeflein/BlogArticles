# Calling the Microsoft Graph from your Teams application - Part 3: Calling Graph from a Teams tab

Microsoft Teams applications almost always need to call the Graph API, yet it's not as easy as just calling a REST service. Most of the complexity has to do with getting an Azure AD access token, which is required on every Graph call to establish what, if anything, the caller is authorized to do.

Getting the access token requires an understanding of Teams, Azure AD, Graph, and sometimes other components like the SharePoint Framework or Bot Framework, yet each of these is documented separately and each one assumes the reader knows all the others. I literally get questions every day from frustrated developers trying to figure this out! (Yesterday I got 3!) In 2 1/2 years of Teams app development, this by far the most common source of difficulties.

I wrote these articles hoping they'll assist developers in calling the Graph from Microsoft Teams. They're also companions for my talk, "Calling Microsoft Graph from your Teams Application", at the [PnP Virtual Conference 2020](https://pnp.github.io/pnpconference/).

1. [Introduction](article1.md)
2. [Deep dive concepts](article2.md) - Deep dive concepts (optional)
3. [Calling Graph from a Teams tab](article3.md) (this article)
4. [Calling Graph from a Teams bot or messaging extension](article4.md)

The first article is intended to explain the basics which anyone should understand before embarking on a Teams project that will call Microsoft Graph. This article is an optional deep dive that will go into more detail, either for the curious or to help in troubleshooting. The articles which follow target specific scenarios, such as calling the Graph from a tab or bot in Teams.

