---
id: 293
title: 'An Elephant Named Multitenancy &#8211; Multitenancy in vRealize Automation'
date: '2016-02-10T09:41:50-08:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=293'
permalink: /2016/02/an-elephant-named-multitenancy-multitenancy-in-vrealize-automation/
categories:
    - Tech
tags:
    - automation
    - multitenancy
    - segmentation
    - tenant
    - vra
    - 'vrealize automation'
---

I had the opportunity recently to spend a few days in sunny Florida with a group of VMware’s Professional Services leaders. The week was spent discussing, demonstrating and teaching them all about the newly released vRealize Automation 7. We focused on how this release could deliver on the promise of truly flexible, extensible automation and enable our customers’ journey to the cloud. But across many of my sessions and discussions, it became obvious that there was a looming question – an elephant in the room.

That elephant’s name? Multitenancy.

I wanted to take a little while and outline what Multitenancy really means, from the formal definition to the way that VMware implements the concept in vRealize Automation, and what that implementation really means for you, my readers, co-workers, customers and friends.

Let’s start with the formal definition, for which I will reach out to Gartner. I chose Gartner as my source for this definition because I believe that this is where many people first saw the term become popular.

According to <https://www.gartner.com/it-glossary/multitenancy> :

***“Multitenancy*** *is a reference to the mode of operation of software where multiple independent instances of one or multiple applications operate in a shared environment. The instances (tenants) are logically isolated, but physically integrated. The degree of logical isolation must be complete, but the degree of physical integration will vary. The more physical integration, the harder it is to preserve the logical isolation. The tenants (application instances) can be representations of organizations that obtained access to the multitenant application (this is the scenario of an ISV offering services of an application to multiple customer organizations). The tenants may also be multiple applications competing for shared underlying resources (this is the scenario of a private or public cloud where multiple applications are offered in a common cloud environment).”*

Whew, that’s a mouthful. Let’s break it down a little bit, as it specifically pertains to vRealize Automation (vRA,) starting with an example.

Frank works for the finance department of a large company. He is responsible for deploying a new instance of a SQL-based financial application. The database for this application contains very sensitive company data that must be protected from unauthorized disclosure – both within the company and to external parties.

Andy, on the other hand, is an intern with the application development department of the same company. He needs a new external, forward facing web server to be provisioned. This server will be available to everyone in the world – both external and internal users.

And finally, Isaac is a system administrator with the IT department at our fictional company. He is tasked with configuring and maintaining the vSphere environment, storage systems and vRA instance (busy guy!)

Now, as the management plane for a hybrid cloud solution, vRA positions itself as a manager of managers. It inherently has visibility into all of the resources that it is responsible for managing, allocating and providing access to. These resources, as defined above, are shared – that is the very nature of cloud computing. Things like compute power in the form of vSphere Clusters or vCloud Air blocks, abstracted storage capacity in the form of vSphere Datastores, network access, etc. One of vRA’s many responsibilities is to act as an authentication and authorization solution – ensuring that while Frank and Andy can both log in to the same portal, they can only see the resources that they have been granted access to.

That means that Isaac must guarantee that Andy cannot access that sensitive financial data that Frank is working with. And while Frank may be a skilled DBA and analyst, the last time he wrote any code was back in the Cobol days – what business does he have deploying web services? None, that’s what!

This is a classic multitenancy example in a cloud-oriented world. Apply the same principles to your own organization – you should be able to see the parallels immediately. Replace Finance and Development with Test and Production, or with your Palo Alto and Washington, DC branch offices.

Here’s where the good news starts. vRealize Automation has been designed to account for exactly these principles. Inside the platform, VMware has provided quite a few layers of access control. Let’s explore some of them in greater detail.

- **Tenants** – These are logical divisions that set boundaries for all of the stored objects and policies inside of vRA, with the exception of the endpoints and collected fabric groups. Tenants provide the opportunity to apply unique branding to the vRA Portal and login splash screen, have dedicated authentication providers and other Tenant-dedicated services (e.g. a unique vRO instance). A Tenant Administrator can delegate the management of objects and resources within their own tenant to other users inside their organization. While handy for logical separation, vRA’s Tenant construct is not frequently used in production deployments, as the following constructs usually provide more than adequate segregation without increasing administrative overhead. VMware’s best practice is to leverage a single Tenant and multiple Business Groups.

- **Fabric Groups** – A Fabric Group is a policy that defines the relationship between heterogeneous compute resources and the authorized administrators who can slice them up into virtual datacenters (VDCs) for consumption, also known as Reservations. This means that an administrator like Isaac can select certain “chunks” of available infrastructure and assign it for use by specific sets of consumers. Examples of an “infrastructure chunk” can include a vSphere Cluster, a vSphere Datastore, an AWS instance or a vCloud Air ovDC. Once these “chunks” are designated as part of a Fabric Group, the responsible Infrastructure Administrator can determine how they are allocated among consumers across Tenants.

- **Reservations** – Reservations are a way for the Infrastructure Administrators to determine **how much** of an “infrastructure chunk” a Business Group can consume. For example, you might have a vSphere Cluster for your Production environment which contains 512GB of pRAM and 5 1TB Datastores. Once you have created a Production Fabric Group, a Reservation can be used to determine that the Finance Business Group gets 64GB of pRAM and 2 of those Datastores, while the Application Development Business Group gets 448GB of pRAM and access to the other Datastore. By doing this, Isaac the Infrastructure Administrator has ensured that the shared cloud infrastructure has been logically segregated, and that Andy can’t over-consume the shared resources and put Frank’s production application at risk.

- **Business Groups** – A Business Group is a logical grouping of users, which can contain both standard users as well as Managers. A standard user is able to log in to the vRA portal and select items from the catalog – subject to their Entitlement, which will be covered in a moment. Managers are responsible for configuring the governance of deployed workloads as well as the membership of the Business Group. Managers can also request and manage items on behalf of the users who they manage.

- **Entitlements** – Finally, the Entitlement is a way to refine exactly what a user of the platform can see and do in the catalog and with their deployed items. And, the actions granted in an Entitlement can be tied in at any point to vRA’s multi-level Approval engine to add management oversight and governance if desired. So, for example, a Java developer could be permitted to deploy only Linux servers with an Eclipse IDE pre-installed, while an MSSQL DBA could deploy only Windows servers with SQL Server installed, pending his manager’s approval. Both could be permitted to power cycle their machines, while only the Java developer might have access to destroy his system. These are just examples – an Entitlement can be configured with virtually limitless combinations of actions. abilities and approvals.

![vRA_MT_Example](/assets/images/2016/02/vRA_MT_Example.png)

So, in our scenario, Isaac would use the **Default vRA Tenant** for his organization – since it’s all the same company, no special branding is needed. He would then create a **Fabric Group** that encompasses the appropriate computing and storage capacity for his departments – whether public cloud based like vCloud Air or private cloud, like an on-premise vSphere environment. Then, he would make two **Business Groups** for the Finance and Development departments – each with a **Reservation** specifying how much of the fabric capacity the department could consume. Finally, he would configure **Entitlements** that determine which users/Business Groups can deploy and manage which types of **Blueprints**. The image above illustrates that while in some cases Blueprints may be shared, the ability to manage workloads provisioned from those shared Blueprints has been limited by the Business Group.

For example, in the scenario pictured, Developers can provision Blueprints 1, 2, and 3 – and Finance users can provision Blueprints 3 and 4. But each resulting object belongs solely to the Business Group that created it – the shared nature of the original Blueprint has no impact on the provisioned server.

![vRA_Isolation_Layout](/assets/images/2016/02/vRA_Isolation_Layout.png)

The segregation outlined in this scenario is **more than sufficient** to meet the multitenancy requirements of virtually every customer. And when you add the incredible power of NSX Micro-Segmentation to your environment, the levels of isolation you can achieve between deployed machines, network segments, organizational units, etc is simply unparalleled.

Now, one potential concern that I have heard customers raise is that an Infrastructure Administrator can “see” the available resources attached to every endpoint. Well, yes and no! In our scenario above, Isaac definitely knows that compute clusters and Datastores dedicated to both Development and Finance exist – that’s his job, after all! But vRA does not enable him to browse those datastores, or manipulate the provisioned servers owned by those Business Group members. Since vRA is positioned to be that manager of managers, owned and maintained by the cloud (or IT) team of an organization, does the simple awareness of the existence of these resources really present any risk?

My argument would be that no, outside of the very rare scenario of a formal carrier-grade service provider – it doesn’t.

I hope this post has helped to clear up some of the confusion around Enterprise Multitenancy in vRealize Automation and helps put some of the FUD around this concept to rest.

Please feel free to leave any feedback you might have in the comments by clicking “Leave a Comment” at the top of the article. I’m very interested in hearing what others think about this topic.

![Brewmaster Jack Garden of Grass](/assets/images/2016/02/Garden_Of_Grass-768x1024.jpg)

And of course… This post was brought to you by Brewmaster Jack’s Garden of Grass American IPA. This fantastic fresh hop beer sports a rare and “experimental” hop varietal known as HBC 452, which imparts a great and juicy watermelon flavor that mixes with the distinct piney-ness of Simcoe hops.

Happy Automating!

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=An+Elephant+Named+Multitenancy+-+Multitenancy+in+vRealize+Automation)</div></div>