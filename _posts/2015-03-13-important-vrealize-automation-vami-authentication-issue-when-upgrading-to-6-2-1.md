---
id: 101
title: 'Important &#8211; vRealize Automation VAMI authentication issue when upgrading to 6.2.1'
date: '2015-03-13T11:05:54-07:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=101'
permalink: /2015/03/important-vrealize-automation-vami-authentication-issue-when-upgrading-to-6-2-1/
categories:
    - 'Compliance and Security'
    - Tech
tags:
    - security
    - vra
---

For those of you who will be upgrading your vRealize Automation appliance to 6.2.1 now that the new version is available, please be aware of an issue that you may encounter.

After upgrading your appliance to 6.2.1 via the VAMI and rebooting, you may find that you are unable to authenticate to the VAMI as ‘root’.

This can be fixed by logging in to the vRA Appliance from the console (as root – this account is unaffected) and running the following command:

chage -M 99999 root

This will reset the expired root password account and allow you to authenticate to the appliance VAMI interface again.

Alternatively, you could change the root password to reset the expiration.

Sorry, no wine content on this one – it’s WAY too early. This message pairs nicely with a cup of coffee.

Happy automating!

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=Important+-+vRealize+Automation+VAMI+authentication+issue+when+upgrading+to+6.2.1)</div></div>