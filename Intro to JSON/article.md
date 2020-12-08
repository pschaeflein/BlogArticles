# Introduction to JSON

It seems like JSON is everywhere these days. Microsoft Teams app manifests, SharePoint list formats, and Adaptive Cards are all written in JSON. And JSON is the standard for [REST APIs](#) like Microsoft Graph; you pretty much can't make a call without it. Power Apps, Power Automate, and Power BI can all handle JSON too. It really is everywhere, except, it seems in older products which were written when XML was king.

JSON and XML do pretty much the same thing: they give you a way to translate objects in a computer into text strings and back again. In the case of JSON, the "objects" can be any kind of structured data that can be expressed as name/value pairs, including complex values that themselves are made of name/value pairs.

> Geek note: Translating JSON text to an object is generally called "parsing", and translating an object into JSON text is "serializing".

A really simple example of JSON is:

~~~JSON
{ "name": "Parker" }
~~~

It's a single name/value pair enclosed in curly braces. A JSON string has to begin with `{` and end with `}`; the JSON string `{ }` represents an empty object. Names are case sensitive and need to be enclosed in double quotes, and in this case, since "Parker" is a string value, it's enclosed in double quotes as well. Spaces, tabs, and newlines are ignored in JSON, but are helpful for readability.

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
  "centimeters": 75,
  "kilograms": 28,
  "friendly": true,
  "siblings": null,
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
  }
}
~~~

These are all name/value pairs, but there are several kinds of values. Here are the details on each kind of value available in JSON:

## Strings

Strings need to be enclosed in double quotes like `"Parker"`. But what if your string has a double quote in it? `"Parker says "Hello""` is not valid JSON because the parser thinks the `"` before `Hello` is the end of the string, and then it gets really confused. Computers are dumb, aren't they? So to put a `"` within a string, you need to "escape" it by preceeding it with a `\`. For example:

~~~JSON
{
      "action": "Parker says \"Hello\""
}
~~~

You also have to escape the `\` character, as in:

~~~JSON
{
  "home": "C:\\Users\\Parker"
}
~~~

While `"` and `\` are the only characters you need to escape, you can also insert UTF-16 code into your JSON strings in the format \uXXXX... but then, you could just include the Unicode character in your JSON.

~~~JSON
{
    "mood": "😀"
}
~~~

## Numbers

Numeric values don't get quotes around them. For example, Parker's dimensions and weight are expressed as numbers.

~~~JSON
{
  "centimeters": 75,
  "kilograms": 28
}
~~~

Note that 75 is not the same as "75", which is a string containing two digits. Numbers are in decimal, and can contain a sign, decimal point, and exponent such as

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
  "friendly": true
}
~~~

## Dates

Unfortunately, there is no standard way to express a date in JSON. In practice, dates are passed in string values, but different applications use different date formats, which can be a bit maddening at times. Your best bet is to look up the date format in the documentation for the product or API you're using.

## Array



## Object

## Null

To indicate an empty value, use null.

~~~JSON
{
    
}
~~~





If you want to 




There are a lot of web sites out there that will format and validate your JSON; recently I've been using [this one](https://jsonformatter.org/). (Be careful never to paste personal data into these web sites!) Why not open it up and play?



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





