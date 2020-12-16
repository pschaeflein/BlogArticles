# Introduction to JSON

It seems like JSON is everywhere these days. Adaptive cards, Microsoft Teams app manifests, and SharePoint list formats are all written in JSON. And JSON is the standard for [REST APIs](#) like Microsoft Graph; you can't make a call without it. Power Apps, Power Automate, and Power BI can all handle JSON too; even Excel can import and export JSON. It really is everywhere except, it seems, in older products which were written when XML was king.

JSON and XML do pretty much the same thing: they give you a way to convert data in a computer into text strings and back again. By using these standard formats, products  can reuse code and developers can reuse their knowledge; it's a win for everyone.

This article is organized in order from simple to complex; if you don't need the more complex parts, just stop reading; you can always come back and read them later!

JSON data is organized as objects containing name/value pairs. The values can be simple things like text strings or numbers, collections of things (arrays), or more objects containing their own name/value pairs.

> Geek note: Translating JSON text to a data is generally called "parsing", and translating an object into JSON text is "serializing".

A really simple example of JSON is:

~~~JSON
{ "name": "Parker" }
~~~

It's a single name/value pair enclosed in curly braces. A JSON string has to begin with `{` and end with `}`; the very shortest valid JSON string, `{ }`, represents an empty object. Names are case sensitive and need to be enclosed in double quotes, and are followed by a `:` and then a value. In this case, since "Parker" is a string value, it's enclosed in double quotes as well. Spaces, tabs, and newlines are ignored in JSON, but are helpful for readability.

If you want to put more than one name/value pair in your JSON, simply separate them with commas like this:

~~~JSON
{
  "name": "Parker",
  "species": "porcupine"
}
~~~

The value part doesn't need to be a string like "Parker"; it could also be a number, a boolean, an array, or another JSON object. This allows you to have objects within objects. For example, here's a more complete description of the quilled crusader:

~~~JSON
{
  "name": "Parker",
  "species": "porcupine",
  "occupation": "mascot",
  "motto": "Sharing is caring",
  "centimeters": 75,
  "kilograms": 28.5,
  "quills": 3.0e+4,
  "friendly": true,
  "bossy": false,
  "nicknames": [
    "Quill",
    "Spike"
  ],
  "classification": {
    "kingdom": "animalia",
    "phylum": "chordata",
    "class": "mammalia",
    "order": "rodentia",
    "suborder": "hystricomorpha",
    "infraorder": "hystricognathi"
  },
  "dnaSequence": null
}
~~~

These are all name/value pairs, but there are several kinds of values. Here are the details on each kind of value available in JSON:

## Strings

Strings need to be enclosed in double quotes like `"Parker"`. But what if your string has a double quote in it? `"Parker says "Hello""` is not valid JSON because the parser thinks the `"` before `Hello` is the end of the string, and then it gets really confused. (Computers are dumb, aren't they?) So to put a `"` within a string, you need to "escape" it by preceeding it with a `\`. For example:

~~~JSON
{
  "name": "Parker",
  "action": "Parker says \"Hello\""
}
~~~

As you might expect, this escaping thing is a bit of a slippery slope, and you also have to escape the `\` character, as in:

~~~JSON
{
  "name": "Parker",
  "home": "C:\\Users\\Parker"
}
~~~

While `"` and `\` are the only characters you need to escape, you can also insert UTF-16 code into your JSON strings in the format \uXXXX... but then, you could just include the Unicode character in your JSON.

~~~JSON
{
  "name": "Parker",
  "mood": "😀"
}
~~~

## Numbers

Numeric values don't get quotes around them. For example, Parker's length and weight are expressed as numbers.

~~~JSON
{
  "name": "Parker",
  "centimeters": 75,
  "kilograms": 28
}
~~~

Note that 75 is not the same as "75", which is a text string containing two digits. Numbers are in decimal, and can contain a sign, decimal point, and exponent such as:

~~~JSON
{
  "quills": 3.0e+4,
  "damage": -10
}
~~~

## Boolean

For boolean values, just use `true` and `false` with no quotes.

~~~JSON
{
  "name": "Parker",
  "friendly": true,
  "bossy": false
}
~~~

## Objects

All the JSON examples in this article so far have consisted of one JSON object with name/value pairs enclosed in a set of curly braces. But you don't need to limit yourself to one object! You can have as many objects as you want as values inside other objects. This kind of nesting allows you to create a hierarchy.

~~~JSON
{
  "name": "Parker",
  "classification": {
    "kingdom": "animalia",
    "phylum": "chordata",
    "class": "mammalia",
    "order": "rodentia",
    "suborder": "hystricomorpha",
    "infraorder": "hystricognathi"
  }
}
~~~

So there we have two name/value pairs, `"name"` and `"classification"`, and the value of `"classification"` is itself an object with several name/value pairs of its own. This is very convenient for organizing the data and, when combined with arrays, allows creating lists, tables, and all sorts of other data structures.

## Array

An array is an ordered set of values enclosed in square braces `[` and `]`, such as:

~~~JSON
{
  "name": "Parker",
  "nicknames": [
    "Quill",
    "Spike"
  ]
}
~~~ 

or, more succinctly,

~~~JSON
{
  "name": "Parker",
  "nicknames": [ "Quill", "Spike" ]
}
~~~ 

Mascots could have any number of nicknames or none at all, and an array allows you to list them. Parker's sister Penny doesn't have any nicknames.

~~~JSON
{
  "name": "Penny",
  "nicknames": []
}
~~~ 

Arrays of objects are especially useful. For example, suppose you wanted to compile a list of Microsoft developer mascots:

~~~JSON
{
  "mascots": [
    {
      "name": "Bit",
      "species": "raccoon",
      "team": "Microsoft Developer Advocates"
    },
    {
      "name": "G-raffe",
      "species": "giraffe",
      "team": "Microsoft Graph"
    },
    {
      "name": "Parker",
      "species": "porcupine",
      "team": "Microsoft 365 PnP"
    }
  ]
}
~~~

Remember that spaces, tabs, and newlines are ignored, so this the same data could be written more compactly like this. Suddenly it starts to look a little bit like a table!

~~~JSON
{
  "mascots": [
    { "name": "Bit", "species": "raccoon", "team": "Microsoft Developer Advocates" },
    { "name": "G-raffe", "species": "giraffe", "team": "Microsoft Graph" },
    { "name": "Parker", "species": "porcupine", "team": "Microsoft 365 PnP" }
  ]
}
~~~

Notice that if you change the order of the array values you need to remember to have a comma between each one with no comma at the end. This is kind of a pain when you change the order and need to remember to move the comma. To make this easier, people sometimes leave an extra comma at the end of the last array value; though some applications allow this, it's not part of the JSON spec, so use it at your own risk.

## Dates and other things

Unfortunately, there is no standard way to express a date in JSON. In practice, dates are passed in string values, but different applications use different date formats, which can be a bit maddening at times. Your best bet is to look up the date format in the documentation for the product or API you're using.

Images and other binary objects are rarely included in JSON, but if you wanted to do that you'd need to turn them into strings somehow, perhaps by Base64 encoding them.

## Null

To indicate an empty value, use null. For example, Parker hasn't had his DNA sequenced, so we have no value for that in his profile.

~~~JSON
{
  "name": "Parker",
  "dnaSequence": null
}
~~~

Note that `[]`, an empty array, and '{}`, an empty object, are different than null. They're empty containers whereas null is really nothing at all.

## Comments

If I could add one thing to JSON, it would be comments! Officially there are no comments in JSON, but some products (like the SharePoint Framework) seem to encourage using JavaScript style comments in JSON. It seems harmless but it's not proper JSON, and most applications will choke on them.

A trick I sometimes use (if I know it's OK) is to just add a few extra name/value pairs in lieu of comments:

~~~JSON
{
  "name": "Parker",
  "classification": {
    "comment": "This is the biological taxonomy",
    "kingdom": "animalia",
    "phylum": "chordata",
    "class": "mammalia",
    "order": "rodentia",
    "suborder": "hystricomorpha",
    "infraorder": "hystricognathi"
  }
}
~~~

While it's not recommended, it is [legal](https://tools.ietf.org/html/rfc8259#section-4) to have duplicate names in a JavaScript object, so you could have more than one `"comment"` if you're daring. This is valid JSON:

~~~JSON
{
  "name": "Parker",
  "comment": "Great mascot but gets a bit prickly at times",
  "comment": "Models for \"Parker's Place\" online apparel shop"
}
~~~

## Tools

There are a lot of web sites out there that will format and validate your JSON; recently I've been using [this one](https://jsonformatter.org/), which does both. (NOTE: Be careful never to paste personal or confidential data into these web sites!)

## Schema support

It's often helpful to enforce some structure to your JSON, requiring that certain name/value pairs are included with certain value types, etc. That's the role of [JSON Schema](). This allows validating the JSON and offering features such as intellisense.

A JSON Schema describes a specific JSON structure. For example, all animal mascots need to have a `name` and zero or more nicknames with an optional value for `quills`, such as:

~~~JSON
{
  "name": "Parker",
  "nicknames": [
    "Quill",
    "Spike"
  ],
  "quills": 30000
}
~~~

This would be expressed in JSON Schema as:

~~~JSON
{
	"definitions": {},
	"$schema": "http://json-schema.org/draft-07/schema#", 
	"$id": "https://example.com/object1607485037.json", 
	"title": "Root", 
	"type": "object",
	"required": [
		"name",
		"nicknames"
	],
	"properties": {
		"name": {
			"$id": "#root/name", 
			"title": "Name", 
			"type": "string",
			"default": "",
			"examples": [
				"Parker"
			],
			"pattern": "^.*$"
		},
		"nicknames": {
			"$id": "#root/nicknames", 
			"title": "Nicknames", 
			"type": "array",
			"default": [],
			"items":{
				"$id": "#root/nicknames/items", 
				"title": "Items", 
				"type": "string",
				"default": "",
				"examples": [
					"Quill"
				],
				"pattern": "^.*$"
			}
		},
		"quills": {
			"$id": "#root/quills", 
			"title": "Quills", 
			"type": "integer",
			"examples": [
				30000
			],
			"default": 0
		}
	}
}
~~~

If this looks complicated, don't worry; many tools like PowerApps can generate a schema for you based on some sample JSON. You can easily generate a schema online using the [JSON Schame Validator and Generator](https://extendsclass.com/json-schema-validator.html); I used it to generate the schema above.

Why bother? Well once you have a scehma, you can get syntax checking and intellisense in tools like Visual Studio Code. Power Apps and Power Automate use schemas to determine what name/value pairs are available in your application.

You can add a property to your JSON to indicate the URL of the JSON schema; for example, to indicate that a file is a Microsoft Teams manifest, include this schema URL:

~~~JSON
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.2/MicrosoftTeams.schema.json"
}
~~~

When a `$schema` property is present, Visual Studio and Visual Studio Code will validate your JSON automatically and provide Intellisense. There are a ton of schemas available at [https://www.schemastore.org/json/](https://www.schemastore.org/json/) for you to reference. You can even reference a JSON schema in your own project by just specifying a relative path:

~~~JSON
{
  "$schema": "./myschema.json"
}
~~~

## JSON and JavaScript

Although JSON stands for "JavaScript Object Notation", and was inspired by the format JavaScript uses for object literals, they are not the same. Some major differences are:

 * In a JavaScript object literal, the names only need to be enclosed in quotes if they are reserved words like `for` or `if` in JavaScript. Furthermore, you can use either single or double quotes. In JSON, names always need to be enclosed in double quotes.
 * JavaScript objects can contain values such as dates, regular expressions, and HTML elements, whereas JSON values are limited to strings, numbers, boolean, arrays, and null.
 * JavaScript strings can be contained in single or double quotes; JSON strings must be contained in double quotes
 * JavaScript numbers can be in octal (using a leading 0) or hexadecimal (using a leading 0x) as well as decimal; JSON numbers must be in decimal.

Bottom line: all JSON objects are valid JavaScript object literals but not all JavaScript object literals are valid JSON.

If you're writing JavaScript, you'll often need to convert a JSON string to an object. You can easily do this using the JSON object. To convert JSON to a JavaScript object,

~~~javascript
var json = '{"name": "Parker"}';
var o = JSON.parse(json);

console.log(o.name); // Parker
~~~

To convert a JavaScript object to JSON:

~~~javascript
var o = new Object();
o.name = "Parker";
var json = JSON.stringify(o);

console.log(json);  // {"name":"Parker"}
~~~

When you make a REST call, you end up using JSON as well. Here's a call to the Microsoft Graph:

~~~javascript
const response = await fetch("https://graph.microsoft.com/v1.0/me/",
    {
        method: 'GET',
        headers: {
            "accept": "application/json",
            "authorization": "bearer " + token,
        }
    });

if (response.ok) {
    const profile = await response.json();
    const name = profile.displayName;   // Parker
}
~~~

Notice that to ask the service for a JSON response, the HTTP header is set to accept "application/json", which is the MIME type for JSON. And the `response` object returned by `fetch()` has a `json()` function built right in to turn the returned JSON into a JavaScript object.



# LINKS

Teams app manifest
SharePoint site themes and list formatting
API's like the Microsoft Graph
Excel can import and export it
Parsers in PowerApps, Power Automate, and Power BI
 - Power Apps: https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/functions/function-json
 - Power Automate: https://docs.microsoft.com/en-us/power-automate/data-operations
 - ower BI: https://docs.microsoft.com/en-us/power-query/connectors/json


Name/value pairs

* keys must be quoted with double quotes
* values can be: string (double-quoted only), number (decimal only), object, array, true, false, or null

Have a little subsection for each type of value

* strings: quoting, escaping
* number: signed, unsigned, exponent, decimal only
* object: unordered name/value pairs
* array: ordered values

Using JSON

* REST calls - specify MIME type application/json
* In Power Platform

Schema support

Comments in JSON

JSON in JavaScript
* JSON is a subset of JavaScript literal notation - all JSON strings are valid JavaScript literals but not viceversa
* Show code for converting JavaScript objects to/from JSON





