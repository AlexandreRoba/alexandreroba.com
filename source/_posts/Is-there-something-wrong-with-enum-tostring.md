---
title: Is there something wrong with Enum.ToString()?
date: 2012-05-24 21:20:25
tags: [C#, .NET]
---
[1]: /public/images/20120524-capture01.gif

This morning, while we where working on the creation of Key/Value pair table containing different kind of entities one of my senior team meber ran into an “Exotic” behavior. 
The key of our table is a string build on the composition of the entity Id and the entity type. The entity type is simply an enum value converted to a string. 

This looked like this:

```csharp
_ToString = string.Format("[{0}/{1}]", id, contactType.ToString());

```

**contactType** is here the enum variable. Nothing really fancy or complicate untill we ran the performance monitor on it.

We saw that this line, according to its simplicity was taken too much time to  process.

We have to perform this operation for millions of records and every millisecond counts.

At first sight I though it was the string.Format() that was taking all the processing and we tried a quick optimization.

```csharp
_ToString = "[" +  id + "/" + contactType.ToString() + "]";

```
Same result. I had to face it, the ToString() of the enum variable was taking most of the processing.

The number of entries in enum being short, we tried the following code:

```csharp
switch (contactType)
{
    case ContactTypeEnum.Undefined:
        _ToString = "[" + id + "/Undefined]";
        break;
    case ContactTypeEnum.Organisation:
        _ToString = "[" + id + "/Organisation]";
        break;
    case ContactTypeEnum.NaturalPerson:
        _ToString = "[" + id + "/NaturalPerson]";
        break;
    case ContactTypeEnum.OrganisationContact:
        _ToString = "[" + id + "/OrganisationContact]";
        break;
    default:
        throw new ArgumentOutOfRangeException("contactType");
}

```

The difference was impressive. This long piece of code is almost 8 time faster.

The following image gives the performance numbers of each of the code block.

![Enum.ToString performance capture](/images/Is-there-something-wrong-with-enum-tostring/20120524-capture01.gif)

