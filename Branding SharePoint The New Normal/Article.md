# DRAFT
# Branding SharePoint: The New Normal

LINKS

[Modern SharePoint](https://bob1german.com/2018/02/09/what-is-modern-sharepoint-and-why-should-i-care/) is catching on, and sites are looking better than ever right out of the box. With mobile-ready pages and easier editing, customers and partners are starting to ask for it. And as SharePoint 2019 brings the modern experience on premises, the demand is likely to appear just as it has online.

Yet even as sites look better than ever "out of the box", there are limitations on how they can be customized. Partners and customers who want to completely change the look sometimes run into these boundaries and get frustrated.

This article will explain what you can and can't customize on modern SharePoint pages and why. It will also offer some alternatives for those who absolutely most color outside the lines.

## How did we get here?

The story really began with SharePoint 2007, when Microsoft Content Management Server (MCMS) was "merged" into SharePoint. MCMS was based on a wonderful web content management (WCM) system called NCompass Resolution, and it allowed designers to build pretty much anything they could imagine. SharePoint introduced "Publishing Sites" which include most of the MCMS capabilities, such as the ability to start with a design composite (image) and make an editable page that looks exactly like it.

![Out of the box publishing site, circa 2007](SP2007-PublishingSite.png)
Out of the box publishing cite, circa 2007

Microsoft started to promote SharePoint as a general purpose WCM system for building any site, public or private. They worked hard to get customers to adopt the technology. If you'd like to see some examples, check out [Top SharePoint Sites](http://www.topsharepoint.com/category/top-sites?r_sortby=highest_rated&r_orderby=desc); notice that they're mostly pretty old.

While SharePoint never did capture more than a sliver of the already fragmented market for public-facing websites, it became the market leader for internally facing Intranet sites. Its flexibility led to an assumption that you could hire designers and developers to build any imaginable site from scratch, and it would benefit from SharePoint's powerful document management, search, security, and other enterprise features.

The resulting solutions were pretty cool until it came time to upgrade SharePoint. Each upgrade required a lot of rework, and sometimes a complete rewrite of the solution. Many enterprises resigned themselves to maintaining multiple versions of SharePoint rather than incur the expenses of upgrading their most customized sites.

Here are some of the architectural issues that led to this situation:

1. SharePoint and customizations share key files, such as master pages and page layouts, with SharePoint and custom code freely intermixed. There was no easy way to  upgrade just the SharePoint parts of these files without breaking the customization. This situation remains in today's SharePoint Online "classic" sites, except now it's compounded by SharePoint's constant and ongoing upgrades, which are pretty standard for a SaaS (Software as a Service) cloud offering.

1. Many of the customization methods required installing code directly on SharePoint servers, which isn't workable in a multi-tenant environment like SharePoint Online. Server-side changes also created problems when migrating to a new version of SharePoint; if every customization isn't installed perfectly on the new servers, the result is broken features and often the dreaded "correlation ID".

1. Many of the customization methods such as JSLink and Display Templates are designed to be performed right on SharePoint sites, and don't lend themselves well to use of source control or any kind of testing outside of production. The same goes for tools like SharePoint Designer.

## You don't brand Word or Outlook, why should you brand SharePoint?

All of this led to another school of thought: instead of "completely customize SharePoint", many wary customers and partners learned to "never customize SharePoint; it only leads to problems later on."

There are good reasons to brand SharePoint, however. Intranets are supposed to drive employee engagement, and branding is an important part of that. And SharePoint is primarily a website, and web sites are nearly always branded.

In addition to branding, other customizations allow SharePoint to be tailored to the policies and workstreams of each customer. Building on SharePoint can save money (vs. starting from scratch), and empower a company's workforce. So let's not throw the out baby with the bath water!

Fortunately, Microsoft learned from the painful issues that arose from customizing classic SharePoint. Modern SharePoint is designed for the cloud, where multiple tenants and ongoing changes are a fact of life. It's also designed to allow developers to follow established best practices, like managing code in a source control system and testing that code before placing it in production.

This is good news, but it comes at a price: there are limits to the ways you can customize modern SharePoint. Some limitations are probably temporary, as Modern SharePoint is still a work in progress. But, like most SaaS services, there will always be some limitations on how you can customize SharePoint. That's the new normal, and requires thinking differently about branding and customization.

For example, you can still hire a designer to come up with a look for your new Intranet, but he or she needs to be aware of what can and can't be changed on modern SharePoint pages, or you need to choose a different path entirely. The next section explains exactly what you can and can't change in modern SharePoint, and the section which follows offers alternatives to consider if you decide you can't live within the boundaries.

## Customization or Configuration?

Every part of a SharePoint page can be configured - you can set up your own content, colors, logos, etc. And most of the page can be fully customized with custom coded web parts and SharePoint Framework extensions.

Figure ??? shows the home page of a modern Communication site, which is typically the one used to publish information on an intranet. Gold boxes and callouts show the portions of the page that SharePoint owns, so you can configure but not customize them. Green boxes and callouts show the portions of the page where you can put use custom code to create anything you wish.

![Customization and Configuration of Modern SharePoint](./CommunicationSitePageRegions.png)

The callouts point to specific areas of the page as follows:

??? ADD LINKS TO ALL OF THESE ???

1. Suite Bar: This bar is common to most Office 365 services, and allows navigation between products, access to the user profile, help, etc. It can be configured with your choice of colors and a company logo.

2. SharePoint Framework header: This is fully customizable (with code) to hold anything you wish.

3. Hub site navigation: This allows navigation within a set of sites that belong to a hub. You can configure the links and add your logo here.

4. SharePoint page header: This displays navigation within the site, the site classification, and some common features such as following, sharing, or editing the page. You can configure this with colors (the site "theme") and a logo.

5. Web parts: You can put any web parts you want on the page. This can include custom web parts written using the SharePoint Framework. So-called "full bleed" web parts can span the width of the page, inviting use for single page applications. So you can really put anything you wish here. This typically makes up the majority of the page, with more screen real estate than all the other elements combined.

6. SharePoint Framework footer: Like the header, this is fully customizable to hold anything you wish.

7. Pop-up buttons: These buttons overlay the bottom of the page, and currently can't be customized or configured. ???

## Branding options

This section offers a number of alternative options for branding SharePoint, some of which go beyond the limitations of modern SharePoint, but generally at a price.

### Option 1: Stay within the guardrails

There are a lot of compelling reasons to leave classic SharePoint and move to the new modern experience, and Microsoft expects that most customers won't mind working within the guardrails. The [official guidance is here](https://docs.microsoft.com/en-us/sharepoint/dev/scenario-guidance/branding).

While it may seem desirable to customize everything, it's a good idea to evaluate the business benefits to the changes. What's the business value in moving the search box a few pixels?

I once watched the marketing VP for a major investment bank have a complete melt-down when the Office 365 suite bar appeared in the wireframes his company had commissioned. The developers found a way to hide it, but at what benefit and cost? I'm still not clear on the benefit of removing it, but the cost was that users could no longer navigate to other Office 365 properties, or easily manage their user profiles.

If you've identified customizations that go beyond modern SharePoint's guidelines, and there is a clear business benefit, there are other options. Read on!

### Option 2: Take a chance on a few hacks

Don't get me wrong, I'm not recommending that you do this! But that doesn't mean some people won't. SharePoint pages run in web browsers, which use HTML, CSS, and JavaScript. There's nothing stopping the code in a SharePoint Framework web part or extension from  overriding CSS or manipulating the HTML Document Object Model (DOM). It's up to you, but you should at least understand the risks. When you do this, you're hoping that SharePoint won't change the page in some way that would break your code, and potentially the whole page.

If you code against a documented SharePoint API and it breaks, you've got a legitimate support case with Microsoft. And trust me, Microsoft does its best to keep its APIs stable and provide plenty of notice in case of a change.

On the other hand, if you reverse-engineer SharePoint and code against an undocumented CSS class or DOM element, all bets are off. Microsoft could change it without notice because they don't know about your change, and notifying everyone about every little change would be ridiculous. If you called support they'd probably be sympathetic, but they're unlikely to back out the changes for one customer. That's the very meaning of the word "unsupported."

Hacking SharePoint is a rocky path, and if you decide to go there, it's your choice to slow down and hack carefully or just barrel on through. If you make 100 changes and forget what they are, what happens when it breaks? Chances are you'll have a small crisis on your hands (or maybe a big one).

It is possible, however, to hack carefully. For example a large enterprise I've worked with has exactly one of these hacks across their entire intranet. It's on the home page, where they used CSS and DOM manipulation to change the out of the box heading. They maintain a copy on a Targeted Release tenant where they test it weekly, knowing that any breaking changes will appear there before they reach production. And they don't just test it once, they test it in the SharePoint mobile app and in a Microsoft Teams tab, as well as in each of the company's supported browsers. If there's a problem, they can remove one hidden web part from the production home page and it will revert to the out-of-box look, and then take the time to come up with a new hack.

If that sounds like a lot of work, well it is. There's a tradeoff between risk, resources, and reward; if you're going to hack, at least make the tradeoff consciously.

### Option 3: Stay on Classic

This is the path chosen by many who have already invested in branding a classic SharePoint site. Microsoft has pretty much stopped enhancing classic SharePoint, which will limit your ability to take advantage of new web parts and other features. An upside to this is that if Microsoft doesn't enhance classic SharePoint, all that intermingled code on the master page is less likely to break.

Bottom line is you'll need to live with the limitations of classic. For example, classic SharePoint pages don't work well on small screens like phones and tablets; you can add your own responsiveness in the master page or page layout, but you'll soon discover that none of the built-in classic web parts are responsive either and you end up writing all custom web parts.

### Option 4: Use SharePoint as a headless CMS

With SharePoint's rich set of client API's, you can surface SharePoint content in any web application. You can design any page you want, host it anywhere, and have it call SharePoint API's to bring in the content. You could even surface SharePoint content in another CMS such as SiteCore or WordPress.

While this works, it's likely to be costly as you're now maintaining more code and possibly a second product to make your Intranet work.

### Option 5: Use an Intranet in a Box solution

There's a huge market of Intranet in a Box solutions out there, most of which are either built on or somehow integrate with SharePoint. These solutions generally come with their own limitations, but perhaps you'll find them more acceptable than the ones in modern SharePoint.

However this isn't really a 5th option, since every Intranet in a Box solution will need to choose one of the other options to build their product! Looking at the big list of products ???, I recognize some which run on classic SharePoint, some on modern, and some outside of SharePoint entirely. And they may or may not resort to a few hacks along the way.

My advice here is to discuss this with the product vendor, and be aware that the same tradeoffs exist for them as they would for you. 

 - If it's built on classic SharePoint, are you ready to live without enhancements from Microsoft? What's the roadmap? Will they be asking you to migrate again someday to something more modern?
 - If it's built outside of SharePoint, are you ready to forgo the SharePoint ecosystem of web parts and add-ins?
 - If it's built on modern SharePoint, does it have the same boundaries, or did they hack it? What's the support plan?

## Getting the genie back in the bottle

There are a lot of benefits to modern SharePoint, not the least of which is improved stability, even in the face of the constant updates made to Office 365. You might not be able to move the search box, but you'll never spend another sleepless night trying to upgrade your Intranet to a new version of SharePoint.

The broad WCM capabilities introduced in 2007, however, set an expectation that, given sufficient time and money, any wish can be fulfilled. Now we have to reset that expectation.

Modern SharePoint will continue to evolve. In all probability there will be more opportunities for customization over time, as well as more features which can only be configured and not fully customized.

Given that reality, it would be smart to educate decision-makers and designers on both the benefits and the boundaries of modern SharePoint. So far, for most customers I've asked, the boundaries aren't really a problem and the benefits are significant. If you're one of the few who need to customize beyond what's possible in modern SharePoint, it's a good idea to understand the options and tradeoffs, and make a conscious decision.


