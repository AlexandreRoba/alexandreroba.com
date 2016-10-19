---
title: Why an entity key should not be a technical id?
date: 2011-07-28 21:20:25
tags: ['Application Design']
---

Like most of application architect out there that I have meet, I used to add a primary key column to every table I added to the ER Model I was working on. It was like a reflex: I create a customer table then I add a column “Pkid” of type int. The values of that column will be given by the RDBMS.

That column had no meaning but the one of giving me a unique value for the table record. That table record was then linked one to one to my business entity. It was fine by that time. But… Then came distributed and loosely coupled systems. The instance of that entity may not be created centrally anymore. It may be created on a disconnected device and referenced by another disconnected instance. This is becoming more difficult to handle.

It is not impossible but more complex. The disconnected system will have to provide a temporary key and used it as the FK value by the referencing entity. Then after posting the entity to the entity owner systems, it will have to update those temporary key with the final values provided by this owner system…. Fine!!! It works… But… My central system may have been sized to support a well defined number of users and the processing of those entities creation request may be pooled… Meaning I do not get the finale Id right after the posting and I may get entities that were created on other disconnected systems. Those new ones may have used as Id the same value that I used… Oups… Trouble again…

Well you may go further and find a good solution for it but at what cost… My advice is to not use a RDBMS primary column as your entity key. You could do it on the RDBMS but this key should not be used outside your RDBMS. You should have a candidate key for which every external system is capable of creating new and unique key. Ideally a real business key or if that is not possible, a Unique identifier like a Guid.

This will save you a major headache when you will go from that centrally managed system to a distributed and lossely coupled system. Trust me.
