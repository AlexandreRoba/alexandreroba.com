---
title: When not asking the right question can lead to stupid answer
date: 2012-03-21 21:20:25
tags: [General]
---

I’m currently working on a project where the long ago choice to use SOAP as standard for all our services is making people stop thinking. I’m convincing that reconsidering technical choice when a valid and valuable context arises is an attitude that should be natural for all architects. But in the reality, it is not… Or maybe they might not be architects.

I’m the kind of guy who prefer understand the “why” instead of learning the “what to do”. I’m an architect and by definition I think ahead, and I design a minimum up front. If I found something illogical…Well I will challenge it.

Let me be more explicit on this. I need to build some services for a mobile middle ware. Those services are DATA services. They only provide CRUD operation on some entities that will made available to a mobile client. This mobile middle ware supports 4 kind of communication technologies (SOAP,SAP RFC, REST, SQL). It supports them in the same extend. Meaning it will not be able to support the full extent of features (transaction, security, reliability…) that each of those technologies support. It supports the bare minimum.

Because we want to be SOA……. (Yeah!!!!) We have to use SOAP…(Yeah… What a shortcut…). Fine we are building DATA Services, and we need to use the more complex protocol for our services… This is a non-sense. Those services are meant only for this middle ware. Their contracts are tailored for the middle ware. There is no possibility for them to be used by something else than the middle ware and we are choosing the most complex and universal protocol….

Why not considering something simpler like REST and a simple and optimized XML message? “Because this is not a company standard!”… What can I reply to this? I believe that this choice can make sense (Yes I say can make sense J) when the services you are building is meant to be a companywide service and used by different platform. But then it needs to be advertised like this. This is not our case… We are mainly providing services that list all instances of some entities. Those services will return hundreds of thousands of records. Let’s reconsider that companywide standard for this case and use it properly. Let’s build something which our customer will take advantage of for once. Let’s be “Thinker” and not follower this time.