---
title: Structural domain entities or context oriented domain entities?
date: 2011-10-26 21:20:25
tags: [Architecture, 'Application Design']
---


I’m working on a project that has been going on for couple of years now. We manage to deliver a solid and nice application composed of several building blocks. Those blocks  go from front end web application to mobile disconnected application all serviced by Web services components.

This web service components are build on top of a DDD Architecture. The challenges we are facing now is that our Domain entities were design based on a structural approach and our domain entities becomes quite complex. For example we have a project aggregate root made of work package entities and them self made of activity entities.

At the begining of project we had only a limited kind of project and activity type. The scope of the solution is constantly increasing and we are now having far more type of project and activities. Those domain entities become bigger and we have a lot behaviour which are partially overlapping each other and it is hard now to change one type specific behaviour without impacting the other inherited.

In my quest to make all those complex entities more manageable and less interdependent. I’m trying to restructure them in bounded context and to organize them in context oriented entities. Meaning I’m trying to build entities which are meant to support some dedicated context and processes. Instead of having a domain entity model that looks like my database model. Their are now behavioural oriented not structural oriented.

By doing this I’m ending up with far more aggregate roots but which are simplier and more maintainable. The repository are also now more complex as well. Before we had almost a one to one relation between a domain entity and the database model. It is not the case any more. We may end up by reading five different table to build a simple domain entity. I can live with that after all this is what the repository is there for. Serialiaze and deserialiez my domain entities.

The major issue i still have to face now is the resistance of the development team to work this way. They have the feeling that we will end up by duplicating a lot of business rules. I tried to show them and explain them that if a business rules needs to be duplicated they we might have to refactor the bounded context cause they might not be that bounded… But that did not do the trick :(

Has anyone gone throug the same experience? I would appreciate any advice on this.

