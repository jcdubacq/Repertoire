# Goal of this project
This project is about an addressbook (referencing *people*) based on tags and dynamic inheritance of properties. If this looks like mumbo-jumbo to you, just imagine. You have a couple of friends living in the same flat. They will share a common mail address, a common phone number (ok, less and less with all these smartphones everywhere, but bear with me). They have two children, that you registered in your addressbook, to keep their birth date handy. Suddenly, they move to a new city. You have to change their address in three different places. And that's three times too many. With inheritance, they would have inherited from a common "home", which you would have changed once to see four records updated instantaneously.

Now, think of each "common set" as a *tag*. There is a tag for the home addresses of your friends (not necessary if they leave alone), but also a tag for their workplace (many people may have the same work address).
There is a tag shared by all your Japanese contacts marking that they prefer to have their full name written as FamilyName GivenName, where as there is another one (given to all people with very low priority) known as a *default tag* indicating that they prefer the western GivenName FamilyName scheme.

What I want to achieve is to have *people* be single elements in a [DAG](http://en.wikipedia.org/wiki/Directed_acyclic_graph), inheriting from *frames* (which I define as being aspects of people that are shared between people. In fact, frames also inherit from other frames. I will use three names to designate things in the address book:

 - **objects** are collections of properties, connected to other objects, called *ancestors* or *tags*. The object has its own properties, plus the ones of its ancestors.
 - **frames** are objects that are knowingly not people, and thus not meant to be displayed as people. All ancestors are frames.
 - Conversely, **people** are final objects, not inheritable by other objects, and considered as people.

An addresbook should be able to register any number of properties, such as emails, profiles (Facebook login, for example), phone number, but also names (people can have a surprisingly high number of names: usual name, official name, nickname, and all sorts of interesting variants).

Some address books implement also connections between contacts. This is highly desirable.

On top of that, I would like to fully support export to [VCard format](http://en.wikipedia.org/wiki/VCard) in the first release.

# What to do

## Properties

Most of the things an addressbook will do is request one or all properties for a given object.

Properties are to be organised by Type and Subtype. The Type is constrained to a limited list of types. The subtype is composed of letters, dashes and numbers.

The special subtype Custom will be equal to the list of values defined on the object with the same type and subtype beginning by Custom.

Some special cases do exist:

 * The special property Name/AbbreviatedGivenName, if not defined, will be equal to Name/GivenName truncated to its first letter. This allows for some weird cases in countries where the abbreviation of some given names is not.
 * Same for Name/AbbreviatedMiddleName: if not defined, will be equal to Name/MiddleName truncated to its first letter.
 * Any subtype beginning with Transcription- will not define a property *per se*, but will instead override the search string.
 * Other special cases may arise.

Export to VCard format will use a mapping from standard properties to VCard properties.

## Flattening

An important operation is the flattening of an object (especially *people*). This is the collapsing of all ancestors into a single group of properties, very simple to interpret, all conflicts having been resolved.

The flattening can be done by allowing to retrieve a list of properties of an object (this is a set computed by exploring the DAG leading to this object).

The property of the graph of ancestry being acyclic must be preserved at all times.

## Multiple inheritance

If an object may inherit from several other objects, there may be conflicts relating to some properties. Thus, each object must have an ordered list of ancestors, not just a set of ancestors. Ancestors coming first set the value of the property.

The UI should allow easy manipulation of this order so that conflicts may be resolved.

## Single properties vs Sets of properties
The paragraph about multiple inheritance may seem simple enough, until you realise that sometimes, you may want to store multiple values for a single property.

For example, professional email may be unique for some people, but often enough there are several forms (short and full) of emails. And somebody working at two different companies will have two different professional emails.

My view is that any property will in fact be a list of values rather than a single value. However, the first one will be the only one useful in most cases (e.g. the only one displayed).

So, one could inherit from two companies setting an email address, but when selecting the person in the UI, only the first one would be used for emails.

## Macros

I really want properties to be computable from other properties.

Example is Name/FullName. Some *style* tag could set this property to be a computed one, namely "Name/GivenName Name/FamilyName". Another one could set it to "Name/CustomFullName". A last one to "Name/FamilyName, Name/GivenName".

My current idea is to use the syntax: `!Type/Subtype,OtherType/OtherSubtype,replacement=...,prefix=...,,suffix=...` (with some parts optional). This has to be developed. With this and a default tag, all settings for the database (default full name, preferred email).

If a property has to be substituted and does not exist, then the value is removed from the list of values. If no value exists, then the property has to be removed from the list of properties.

Example for Name/FullName, western style:

    !Name/Title,suffix=" "!!Name/GivenName,suffix=" "!!Name/AbbreviatedMiddleName,suffix=" "!!Name/FamilyName,replacement=""!


## Conveying the structure for editing

When the first value of 