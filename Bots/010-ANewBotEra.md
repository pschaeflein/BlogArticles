# A New Era for Bots in Microsoft Teams

A new version of the Bot Framework was quietly revealed at Microsoft Ignite last month (November 2019). It wasn't a secret, but it didn't make any keynotes either. Mary-Jo Foley gave it nary a mention. Yet it represents a major step forward in bot development at Microsoft, and a model for partnership in engineering.

Two great Microsoft engineering organizations - the Azure bot team and the Microsoft Teams team - broke out of their silos and started collaborating.  And so, in version 4.6 of the Azure Bot framework, Teams became a first-class citizen with native support from the Azure Bot framework itself.

There are a number of benefits to this arrangement:

 * Versioning: When one library depends on another, developers often are unable to upgrade either library until new, compatible versions of both are available. Teams Bot developers were stuck in just such a debacle when Bot Framework v4.0 came along, and broke compatibility with the v3.0 Teams bot extensions.  By combining the libraries, this can never happen again.
 * Consistentcy: Developer tooling, samples, and documentation are simpler to navigate. Developers don't need to ask if something is a "bot" thing or a "teams" thing - everything is there in one place.
 * Easier: Less is more when it comes to libraries and frameworks. Developers only have to install and learn one framework.

This article how these technologies work together, and kicks off a multi-part series on building Teams bots.

* [Part 1 - A new era for Bots in Microsoft Teams](010-ANewBotEra.md) <-- This article
* [Part 2 - Building low-code bots for Teams with QnA Maker](020-BuildingLowCodeQnABots.md) 
* [Part 3 - Tooling up for Teams Bot development](030-ToolingUpForTeamsBots.md)
* [Part 4 - Building Teams chatbots with LUIS and Dialogs](040-BuildingTeamsDialogs.md)
* [Part 5 - Using Adaptive Cards in Teams bot conversations](050-AdaptiveCardConversations.md)
* [Part 6 - Building Bot experiences in Teams with messaging extensions and task modules](060-MessagingExtensionsTaskModules.md)
* [Part 7 - Calling back-end services from Bots in Teams](070-CallingBackEndServices.md)

## What can a Teams bot do?

A Teams bot is a conversational user interface that blends into the Teams collaboration experience. Teams bots have a number of ways to interact with users, including:

* Exchanging text messages in Teams channels, or in 1:1 or group chat
* Enhancing text interactions with interactive cards that add a visual element to the conversation, and provide simple forms functionality
* Extending context menus and commands in the Teams UI to allow users to insert text, and cards into the conversation
* Displaying modal dialog boxes with embedded web pages or cards

Many people may think of chatbots as applications that can pass the [Turing Test](https://en.wikipedia.org/wiki/Turing_test) - that is, that can pass themselves off as human. While conversation and natural language understanding are key aspects of a good bot, the best bots aren't necessarily the ones with the most impressive AI. Like any UI, a great bot solves the user's needs in the easiest and quickest way; often this means constraining the bot's capabilities rather than trying to make a do-it-all personal agent.

Bots are distinct from command lines in that humans don't have to learn how to talk to them; they can speak in plain language, and the Bot will converse with them until it has a clear understanding of what the user wants. This gives bots a strong usability advantage, especially for tasks that users do infrequently where they may forget how to do them.

Other advantages of bots include:

* They work well in small spaces, like on phones or embedded into other applications
* They work well in groups, where everybody can see the conversation and have a chance to interact
* They leave a discoverable audit trail, which is useful to teammates and for legal discovery

Teams bots go beyond ordinary chatbots, with features that work well among groups of users.

### Adaptive Cards

Adaptive cards are snippets of interactive content that drop right into a Teams conversation, or can be displayed elsewhere in the Teams UI. 


### Messaging Extensions


### Task Modules



## How does a Teams bot work?


