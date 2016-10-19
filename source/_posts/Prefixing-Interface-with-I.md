---
title: We should prefix our interface with an I
date: 2011-07-27 21:20:25
tags: [C#, .NET]
---

I’m currently reading the book “Clean Code A Handbook of Agile Software Craftsmanship” From Robert C. Martin and I read something that I would like to comment. In the section “Avoid Encoding” he mention the prefixing with an “I” the interface that should be avoided. Instead we should use, if needed, a suffix “imp” for the implementation class.

This is a convention which is quite used in the Java world. I’ve downloaded a couple of open source framework in Java just to read their code source and I’ve found that convention used on a lot of them and I do not like it much. I prefer to use the I on the interface. Why? Cause the only unique element in an interface and the class implementing it, is the interface. Imagine the example provided in the book.It gives ShopFactory for the interface and ShopFactoryImp for the implementation but what about another factory implementing the interface. a Mock for example. It will be called ShopFactoryMockImp? What if they implement multiple interface?… I don’t think so. We should call the interface IShopFactory and all the class implementing it may not have to reference the interface name in their names. This is cleaner and makes more sense.

The real question is should we use any prefix on the interface? And again… I believe so. Why When I’m browsing the solution files I like to see in the names what are the interfaces and what are the classes. Same thing when looking at the code, I like to know what is the base class and what are the interfaces amongst the inherited base class if any and the implemented interfaces.

I hope this comment will help you make your own opinion.