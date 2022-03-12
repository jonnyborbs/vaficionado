---
id: 543
title: 'vRealize Automation 7.3 REST API Documentation'
date: '2017-05-18T05:55:18-07:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=543'
permalink: /2017/05/vrealize-automation-7-3-rest-api-documentation/
categories:
    - Integration
    - Tech
tags:
    - api
    - documentation
    - extensibility
    - rest
    - vmware
    - vra
    - 'vrealize automation'
---

With a lion’s roar, a new version of vRA went GA on May 16th – complete with dozens of new features, hundreds of bugfixes, and a heaping helping of love and care. If you’d like more info on anything but that last part, please see the release notes [here](https://pubs.vmware.com/Release_Notes/en/vra/73/vrealize-automation-73-release-notes.html?src=vmw_so_vex_akjaer_1025). But one thing that may be overlooked are the significant improvements that have been made in the vRealize Automation 7.3 REST API Documentation.

As of the time of writing, we have made several samples available (in the form of Postman collections) containing REST API calls for our most common vRA use cases. These samples are hosted on GitHub at <https://github.com/vmwaresamples/vra-api-samples-for-postman>

For more detailed information on these samples, please see this blog post by our very own Sudershan Bhandari on what he was trying to accomplish with this collection and how you can use it to accelerate your use of the vRA APIs. <https://blogs.vmware.com/management/2017/05/vrealize-automation-api-samples-for-postman.html>

Some examples of the API samples provided include:

- Create and entitle a composition blueprint
- Create and entitle a parameterized blueprint (using the all-new component profiles)
- Export/Import blueprints and other content/components
- Perform various day 2 operations on catalog resource including reconfigure, Scale-In/Scale-Out and others
- Manage endpoint configuration
- Create approval policy and approve or reject an approval request
- Create reservations of various types
- Create and manage a tenant, including creating authentication directories for the tenant
- Manage users and their roles
- Configure a NSX provisioning setup including endpoint, reservation, network profiles and sample blueprints
- Create property definitions and retrieve values backed by vRO script actions
- Create and manage reclamation requests
- Register event topics and subscribe/delete subscription to event topics
- API tips on bearer token management, pagination, sorting, filtering

We have also entirely revamped our API documentation reference on VMware{code}, so it now shows the APIs per service, an overview of each service, the API listing and relevant sample code snippets all in a very organized and easily searchable manner. Check that out at <https://code.vmware.com/apis/vrealize-automation>

Our API programming guides have also been completely reworked for ease of use and friendlier navigation – to get you started faster and support you more easily.

So, while there are tons of amazing new capabilities in our new flagship release of vRealize Automation, I hope you won’t overlook the huge investment we’ve made in this vital area. Check it out today!

As always, this post was brought to you by Tropikalia IPA by White Stork Brewing Company. It’s pretty much my go-to while I’m working with our amazing vRA engineers in Sofia.

![tropikalia_ipa](/vaficionado/assets/images/2017/05/tropikalia_ipa-768x1024.jpg)

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=vRealize+Automation+7.3+REST+API+Documentation)</div></div>