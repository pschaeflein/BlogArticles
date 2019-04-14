# Idempotent Site Scripts for SharePoint

As you probably are aware, SharePoint site _scripts_ are used to set up content and settings in SharePoint sites. They're applied using site _designs_, which allow the same script to be reused with different names and permissions. Site designs can be applied when a site is created, when it's added to a hub site, or on demand via PowerShell or the SharePoint UI.

One of my favorite aspects of site scripts is that they are _idempontent_. Idempotent is a fancy word that means an operation can be applied applied multiple times without changing the result. For example, the number 1 is idempotent with respect to multiplication, since any number times 1 is the same number. By making site scripts idempotent, they can be run over and over again without changing the result. This is really handy in case you want to re-apply the site design and ensure your site is up-to-date with the latest and greatest.

For example, if you create a SharePoint list with the New-PnPList command it will hopefully work right the first time, but if you run it again you'll get an error because the list already exists. So the script can only be run once, unless it explicitly handles the possibility that the list was already created. This is inconvenient and makes it harder to write scripts that can be re-run to keep sites up to date with the latest version of the script.

On the other hand if you create a list in a site script, you can apply the script over and over again and it will only create the list if it's missing.

~~~JSON
 {
    "$schema": "schema.json",
    "actions": [
        {
            "verb": "createSPList",
            "listName": "TestList",
            "templateType": 100,
        }
    ],
    "version": 1
 }
~~~

I wanted to show this to a roomful of conference attendees, so I applied my site design twice. Much to my surprise, although the list was only created once and no error occurred, the fields I created in my subactions were duplicated, and more were added each time I ran it! I guess I was tempting fate by trying this in front of an audience after singing the praises of idempotence!

Here's the script in question; it adds a list with two custom columns, Column 1 and Column 2. And if you run it twice, you get two Column 1's and two Column 2's.

~~~JSON
{
    "$schema": "schema.json",
    "actions": [{
        "verb": "createSPList",
        "listName": "TestList",
        "templateType": 100,
        "subactions": [
            {
                "verb": "SetDescription",
                "description": "Test list for Hub 1"
            },
            {
                "verb": "addSPField",
                "fieldType": "Text",
                "displayName": "Column 1",
                "isRequired": false,
                "addToDefaultView": true
            },
            {
                "verb": "addSPField",
                "fieldType": "Number",
                "displayName": "Column 2",
                "addToDefaultView": true,
                "isRequired": true
            }
        ]
    }],
    "bindata": { },
    "version": 1
}
~~~

So I reached out to my friend and colleague Sean Squires, who led the development of Site Designs. And it turns out there was a subtle aspect to this, and an easy fix.

It turns out that when you create a field and only specify the display name (as shown above), SharePoint helpfully generates a new internal name for the field each time, so each time I applied the site design I got new fields with new internal names! The fix is simple: specify an explicit internal name so SharePoint won't generate a new one, and the site script engine will avoid creating a duplicate.

~~~JSON
{
    "$schema": "schema.json",
    "actions": [{
        "verb": "createSPList",
        "listName": "TestList",
        "templateType": 100,
        "subactions": [
            {
                "verb": "SetDescription",
                "description": "Test list for Hub 1"
            },
            {
                "verb": "addSPField",
                "fieldType": "Text",
                "displayName": "Column 1",
                "internalName": "Column1",
                "isRequired": false,
                "addToDefaultView": true
            },
            {
                "verb": "addSPField",
                "fieldType": "Number",
                "displayName": "Column 2",
                "internalName": "Column2",
                "addToDefaultView": true,
                "isRequired": true
            }
        ]
    }],
    "bindata": { },
    "version": 1
}
~~~

The rub is that the internalName property is optional, and is often not used in examples on the web. Further, the documentation doesn't point this out or specify other objects where SharePoint may generate a new item rather than allowing the site script to be re-run as expected.

Moral of the story: Always test your site scripts for idempotence by running them repeatedly on the same site before releasing them. And never test this in production or - worse - in front of an audience!