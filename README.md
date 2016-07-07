# Terrifically Simple JSON

## Introduction

The goal of
Terrifically Simple JSON is to define the simplest and most regular way possible of using JSON to represent data in Web APIs<a href="#footnote1" id="ref1"><sup>1</sup></a>. 
The syntax of Terrifically Simple JSON is exactly the same as the syntax of regular JSON—Terrifically Simple JSON concerns 
itself only with how you use JSON.

[Regular JSON](http://www.json.org/) describes an object as "an unordered set of name/value pairs"<a href="#footnote2" id="ref2"><sup>2</sup></a>. 
It does not otherwise say what an object is, what a name is or what a value is (beyond defining basic datatypes).
This gives designers a lot of flexibility, which often leads to complex JSON. 

Terrifically Simple JSON reduces complexity by adding 3 constraints:

1. Every Terrifically Simple JSON object must correspond to an entity in the data model of the API.
2. The `name` of a name/value pair must refer to a property or relationship in the state of the corresponding entity.
3. The `value` of a name/value pair must be the value of the entity property referenced by the name.

Terrifically Simple JSON defines a special property, `_self`, that allows you to declare explicitly which data model entity a particular JSON object corresponds to.
Its value is always a URI, encoded as a JSON string.

The only requirements of Terrifically Simple JSON are that you follow the 3 constraints above, and use the `_self` property to express constraint #1 explicitly.

The 3 constraints seem to imply that the media type is "just JSON", but the use of `_self` 
implies a new media type<a href="#footnote3" id="ref3"><sup>3</sup></a>, however minimal. Registration is pending for `application/vnd.terrifically-simple+json`.

There is a generalization of the `_self` concept that allows arbitrary datatypes to be expressed in JSON in a consistent fashion. 
This generalization is expressed with the optional `_value` and `_datatype` properties. Terrifically Simple JSON
does not require you to use them, but they are there if you want an explcit way to handle arbitrary datatypes in Terrifically Simple JSON.

## Tutorial

The following is a Terrifically Simple JSON document:
```JSON
{
 "name": "Martin",
 "bornOn": "1957-01-05",
 "bornIn": "http://www.scotland.org#"
}
```
Because of Terrifically Simple JSON's 3 constraints, we know that this single JSON object with 3 name/value pairs represents a single entity 
in the API data model with 3 property values.

The following JSON document encodes the same information in a different style. It is not Terrifically Simple JSON—it violates all 3 of the constraints above. That does not mean it's a bad
design, just that it follows a different set of rules than Terrifically Simple JSON. It is its own media type.
```
 1. {
 2.  "properties": 
 3.     {
 4.      "name": "Martin",
 5.      "bornOn": "1957-01-05"
 6.     },
 7.  "links": [
 8.     {
 9.      "rel": "bornIn",
10.      "href": "http://www.scotland.org#"
11.     } 
12.  ]
13. }
```

Lines 3-6 and 8-11 violate constraint 1—these JSON objects have no corresponding entity in the data model.  
Lines 2, 7, 9, and 10 violate constraint 2—these JSON names are not properties of a corresponding entity.  
Line 9 violates constraint 3—`bornIn` is a property (or relationship) name in the data model, not a value.

### _self

Constraint #1 of Terrifically Simple JSON is that every JSON object
corresponds to an entity in the API data model. The `_self` property provides a direct way of specifying which entity. 
Its value is always a URI.
Here is an example of its use:

```JSON
{
 "_self": "http://martin-nally.name#",
 "firstName": "Martin"
}
```
This example says simply that the entity whose id is http://martin-nally.name# has the first name Martin.
Constraints 2 and 3 tell us that the first name is Martin and the `_self` property tells us which API entity we are talking about.
This is the primary idea in Terrifically Simple JSON—if you have understood this idea, you have understood most of what
is valuable in Terrifically Simple JSON.

The `_self` property can be used in nested objects too, like this:
```JSON
{
 "_self": "http://martin-nally.name#",
 "bornIn": 
    {
    "_self": "http://www.scotland.org#",
    "gaelicName": "Alba"
    }
}
```
This example encodes two separate pieces of information:
* http://martin-nally.name# was born in http://www.scotland.org#
* The Gaelic name for http://www.scotland.org# is Alba

### When `_self` is missing

When the `_self` property of a Terrifically Simple JSON object is missing, the object still must correspond to an 
entity in the API data model. A JSON object with no `_self` should be read as a noun clause that references an entity. This example
```JSON
{
 "isA": "Person",
 "name": "Martin",
 "eyeColor": 
    {
     "isA": "RGBColor",
     "red": 0,
     "green": 0,
     "blue": 155
    }
}
```

should be read as meaning,
"**the Person whose name is Martin** has eyes colored **the RGBColor whose red value is zero, green value is 0 and blue value is 155**".

### <a name="explicit-urls"></a>An explicit way of encoding URLs

The following two examples are both Terrifically Simple JSON (you can verify that they follow the constraints).
```JSON
{
 "_self": "http://martin-nally.name#",
 "bornIn": "http://www.scotland.org#"
}
```
```JSON
{
 "_self": "http://martin-nally.name#",
 "bornIn": {"_self": "http://www.scotland.org#"}
}
```
Interpreted strictly,
the second says I was born in a country while the first says I was born in a string. Common sense tells us that
the intent of the first is the same as that of the second. Computers are not very good at common sense, but humans are if they have some context.
Whether you use the first or second form in your API might depend on who the audience is and whether you want to require 
them to have context. This choice involves a trade-off between ease-of-use and precision—
the first form is simpler to read and code for a human who has the required context and the second form is more precise and correct 
and therefore can be reliably interpreted without context <a href="#footnote4" id="ref4"><sup>4</sup></a>.

### That's it

That is it for the required part of Terrifically Simple JSON. The next section describes an optional extension.

### <a name="datatypes"></a>Datatypes

One of the challenges of JSON is that it only supports 3 useful datatypes: number, string, and boolean (the `null` value is the unique member of a 4th datatype). 
The two most common
datatypes in Web API programming that are not covered by JSON are URL and date/time. Unless you are willing to invent extensions or
conventions on top of JSON—in other words, a new media type—the best you can do is to encode them as strings. The [example above](#explicit-urls) shows how the `_self` property
can be used in Terrifically Simple JSON to encode URLs more precisely, at some cost to simplicity and ease-of-programming. 
This idea can be extended to other datatypes by defining the following two Terrifically Simple JSON examples to be equivalent.
```JSON
{
 "_self": "http://martin-nally.name#",
 "bornIn": {"_self": "http://www.scotland.org#"}
}
```
```JSON
{
 "_self": "http://martin-nally.name#",
 "bornIn": {
    "_value": "http://www.scotland.org#",
    "_datatype": "resource"
    }
}
```
The `_datatype` value tells you the datatype of the entity referenced in the `_value` field, and by inference the notation of the reference itself. 
`_datatype` and `_value` must both appear in an object, or neither must appear.
A JSON object may have `_self` or `_value`, but not both.
The `_self` property is a shorthand way to express a `_value` for entities whose `_datatype` is `resource` and hence whose reference notaton is [URI](https://tools.ietf.org/html/rfc3986). 

Other values for `_datatype` can be used for other datatypes, e.g. dates, like this:
```JSON
{
 "_self": "http://martin-nally.name#",
 "bornOn": {
    "_value": "1957-01-05",
    "_datatype": "http://www.w3.org/2001/XMLSchema#dateTime"
    }
}
```

In principle, the JSON built-in types can be
handled the same way. In other words, the following are equivalent:
```JSON
{
 "_self": "http://martin-nally.name#",
 "heightInCM": 178
}
```
```JSON
{
 "_self": "http://martin-nally.name#",
 "heightInCM": {
    "_value": "178",
    "_datatype": "http://www.json.org/#number"
    }
}
```
[<a href="#footnote5" id="ref5"><sup>5</sup></a>] If this example
seems unintuitive, consider that
when you write `178` in JSON, you are writing a reference to an entity. The number 178 has been around a lot longer 
than JSON—the notation you are using to write this reference was
[developed over 3 millenia](https://en.wikipedia.org/wiki/History_of_the_Hindu-Arabic_numeral_system). Other notations for writing references to the same entity
exist—it can be written as `CLXXVIII` in [Roman numerals](https://en.wikipedia.org/wiki/Roman_numerals).
When you write `true` or `false` in JSON, you are writing a reference using a different notation—one that is specific to booleans. 
Strings can be viewed the same way, although it may take a little more thought to see why this is true (think of different notions for designating the same string). 
Any datatype can be thought of as consisting of a pre-defined set of entities with a notation for writing
references to them [and perhaps some operators on them, but operators are outside of the scope of JSON].
JSON has built-in notations for referencing numbers, booleans and strings—for other datatypes, 
we must state explicitly which datatype and notation we're using, or rely on contextual knowledge.
This view of datatypes says that in Terrifically Simple JSON, values as well as objects must correspond to entities in the API model.

I'm not suggesting that anyone would actually encode a number in JSON in this way<a href="#footnote6" id="ref6"><sup>6</sup></a>—the point is to show
that the concept works for all datatypes. If you need a way
to encode dates and other dataypes that distinguishes them from strings, this is the way you should do it in Terrifically Simple JSON.
If you don't need to distinguish dates from their stringified equivalents—that is, you rely on the client having enough context—
you can represent dates as simple strings.

## No Collections?

Many media types include special support for collections. Terrifically Simple JSON considers collections to be simply entities
that are exposed using the normal rules of Terrifically Simple JSON. Here is an example:

```JSON
{"_self": "http://scotland.org/native-sons",
 "contents": [
    {"_self": "http://martin-nally.name#"},
    {"_self": "many more like this"}
    ]
}
```

It would be even simpler to write:
```JSON
[   {"_self": "http://martin-nally.name#"},
    {"_self": "many more like this"}
]
```
This second option may not be wrong, but it does not express the fact that "http://scotland.org/native-sons" is itself an 
entity with a URL and potentially properties.

In summary, we think that standardization of collection representations is better handled as a data modelling (ontology) problem
than a media-type problem.

## Prior Art and Acknowledgements

If you know RDF, you will recognize that Terrifically Simple JSON is a representation format for a data model that is loosely related to RDF.
Adding a `_self` property to a JSON object converts JSON's name/value pairs into triples, with the value of the `_self` property providing the subject.
Terrifically Simple JSON objects without a `_self` property are like RDF blank nodes.
Terrifically Simple JSON is not trying to define a new RDF format or a new data model—its goal is
to build minimally on JSON to make Web API design and implementation easier.

Compared to the RDF data model, Terrifically Simple JSON lacks the requirements that predicates and classes be entities 
identified with URLs and lacks the ability to express multi-valued properties—JSON's array feature expresses list-valued properties.
Having done a couple of projects using a strict interpretation of the RDF model, 
we've seen that those RDF features cause significant friction in practical API programming. 
Those features also contribute significantly to the complexity of standard JSON representations
of RDF, especially [JSON-LD](http://json-ld.org/), whose complexity is likely to be fatal in our opinion, 
but also [RDF/JSON](https://www.w3.org/TR/rdf-json/).

Terrifically Simple JSON has very few concepts, and those it has are mostly stolen from elsewhere. The value is in what was left out, not what was put in.
The `_self` property of Terrifically Simple JSON corresponds fairly exactly to the `@id` property of JSON-LD. This is the only concept found in JSON-LD that also appears in Terrifically Simple JSON. 
We chose `_self` instead of `@id` because `@id` is awkward for Javascript programming, and `self` is used by others (standardized by [ATOM](https://tools.ietf.org/html/rfc4287), and often copied).
`_value` and `_datatype` perform the same functions as `value`, `type` and `datatype` from RDF/JSON. We use `_value` and `_datatype` (only 2 are needed)
to reduce the likelihood of collisions.

## _
<a name="footnote1"><sup>1</sup></a> Terrifically Simple JSON could also be used in other contexts where JSON is used. <a href="#ref1">↩</a>

<a name="footnote2"><sup>2</sup></a> Although this is what the JSON site says, it is not what is generally implemented for JSON.
What is implemented is an unordered set of names, each with an associated value. The definition as written
implies that there can be more than one name/value pair with the same name but different values. The [full IETF spec for JSON](http://www.ietf.org/rfc/rfc4627.txt?number=4627)
says that "The names within an object SHOULD be unique". This restriction allows JSON to map simply to common programming language constructs,
which is the primary value of JSON and reason for its success.<a href="#ref2">↩</a>

<a name="footnote3"><sup>3</sup></a> Since it is part of the media type, the `_self` JSON
property is exempt from constraint #2, although you can think of an entity's identity as being one of its properties if you prefer. 
`_value` and `_datatype` are also part of the media type and thus exempt from constraint #2.<a href="#ref3">↩</a>

<a name="footnote4"><sup>4</sup></a> To understand what it is like to lack the required context, imagine both examples with all names and values in Chinese characters
(unless you can actually read Chinese, in which case use Cryllic or Arabic). <a href="#ref4">↩</a>

<a name="footnote5"><sup>5</sup>I invented this URL for the JSON number notation—I'm not aware of an official one.</a><a href="#ref5">↩</a>

<a name="footnote6"><sup>6</sup></a> [RDF/JSON](https://www.w3.org/TR/rdf-json/) encodes even types for which JSON has built-in support this way.
Perhaps they didn't want to depend on JSON's notations for basic types, preferring those defined by RDF, XML Schema and other standards. <a href="#ref6">↩</a>
