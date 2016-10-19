---
title: SOA Patterns Entity Linking
date: 2011-08-16 21:20:25
tags: [SOA]
---

[1]: http://www.infoq.com/news/2011/08/soa-patterns-update
[2]: http://www.soapatterns.org/masterlist_a.php
[3]: http://www.soapatterns.org/entity_linking.php

Amongst the recently [5 new SOA patterns][1] that were promoted from the candidate list to the [Master list][2] of  the Master SOA Design Pattern Catalog, there is one pattern that I found pretty interesting. It is the [Entity Linking][3].

The idea behind this pattern is that, if you have defined several entity services in your SOA landscape, you could provide with one of the entities the id of the references entities AND the links to get access to those referenced entities. To be more concrete you could have two services, the order entity services and the product entity service, and when requesting an order entity you will get with the order entity the links to all product entities which are referenced in the order.

This sounds interesting but can it be implemented? Are there any person who have tried to implement this pattern? I’m curious about the technic they have used to manage their links. I will be investigating more on this and I will keep you informed on my findings.