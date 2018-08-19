# WORKING DRAFT
# Branding SharePoint: The New Normal

Article about how people should think about branding SharePoint in 2018. 
I'm not a branding expert but I know a lot about customizing SharePoint, and that's the source of the confusion around branding.

## How did we get here?

* SharePoint merges with MCMS (it's a destructive merge) and is billed as a WCM system for public and internal sites. Complete, deep branding is possible.  This is the current "classic" publishing model.

* The architecture proves to be brittle and unsuitable for a SaaS Service (and even on-prem SharePoint should be managed as a private SaaS service and not a custom application).
The problems came from (a) lack of clear swim lanes between SharePoint and the customization (e.g. who updates a custom master page?) and (b) dependence on changing the server in ways that aren't workable in the cloud (e.g. delegate controls, navigation node cache), (c) customization methods that can't be effectively managed as they're deployed right in production with no provision for source control or testing (e.g. JSLink, display templates)

* Modern is more locked down; there are clear "swim lanes" between the product and the customization. BUT people have the expectation they can do deep branding AND they also expect to need it. (SHOW screen shot of OOB publishing site circa 2007)

## Limits to modern

Picture of what can and can't be customized on a modern SP page

## The New Normal

Approaches to branding

[Official guidance is here](https://docs.microsoft.com/en-us/sharepoint/dev/scenario-guidance/branding)

 - Stay in the guardrails for Modern. Though this will evolve there will probably always be limits. But really, what is the business benefit to the changes you want to make? Themes allow you to control the color scheme and fonts; SPFx extensions give you custom headers and footers; and the vast majority of the screen is content. Look over the options which follow and ask yourself if it's worth it to get control of the last few percent of the page.
 - Take a chance on a few hacks. It's up to you if you want to hack carefully or just throw caution to the wind. For example a large enterprise I've worked with has exactly one of these hacks on the intranet home page to modify the out of the box heading. They maintain a copy on a Targeted Release tenant where they test changes from Microsoft before they hit the production tenant. They test it regularly in the SharePoint mobile app and in a Microsoft Teams tab, as well as in the company's supported browsers. If there's a problem, they can remove one web part from the production home page and it will revert to the out-of-box look. If that sounds like a lot of work, well it is. There's a tradeoff between risk, resources, and reward; if you're going to hack, at least make the tradeoff consciously.
 - Stay on Classic
 - Use SharePoint as a headless CMS. With SharePoint's rich set of client API's, you can surface SharePoint content in any web application. You could even surface SharePoint content in another CMS which gives complete creative control such as SiteCore. Unily is an example of this.
 - Full custom with SharePoint as a headless CMS

## Getting the genie back in the bottle

Educate decision-makers and designers
Ask your Intranet in a Box vendor how they do it

