---
id: 385
title: 'vIDM Attribute Mapping in vRA 7'
date: '2016-02-25T10:54:31-08:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=385'
permalink: /2016/02/vidm-attribute-mapping-in-vra-7/
categories:
    - Integration
    - Tech
tags:
    - 'active directory'
    - 'attribute mapping'
    - 'identity manager'
    - ldap
    - vidm
    - vra
    - 'vrealize automation'
---

It seems like the more time I spend with the new VMware Identity Manager (vIDM) in vRealize Automation 7, the more great new capabilities I discover. Today’s post comes directly from a customer request, and discusses how to use vIDM Attribute Mapping in vRA 7.

Due to complexities in this customer’s Active Directory environment, they have the “email” attribute in their user accounts populated – but it does not contain the user’s actual email address. This means that vRA is unable to send them notifications, as it automatically inherits this field and uses the information therein.

Have no fear, vIDM is here.

Here you can see my account with the default configuration. My email address is set to **jon@corp.local**, but that just isn’t where I receive my email.

![User_Account_Default_EMail](/assets/images/2016/02/User_Account_Default_EMail-1024x206.png)

Looking at my account inside of Active Directory, we can see that this address is set in the ‘E-mail’ field, which maps to the ‘**mail**‘ attribute in LDAP.

![User_AD_Email](/assets/images/2016/02/User_AD_Email-767x1024.png)

But, if we look at the Attribute Editor, we can see that the LDAP ‘**otherMailbox**‘ attribute contains my preferred email address of ‘**jon@vaficionado.com**‘

![User_AD_otherMailbox](/assets/images/2016/02/User_AD_otherMailbox-771x1024.png)

So, how can I change my vRA configuration to utilize that **otherMailbox** attribute instead? It’s very easy. Start by clicking on the **Administration** tab in vRA. Then select the **Directories** button on the left hand side and edit your **Active Directory** shown to the right.

![Navigate_To_Directory_Configuration](/assets/images/2016/02/Navigate_To_Directory_Configuration-1024x362.png)

Next, you’ll be presented with the Active Directory settings page. Click on the **Sync Settings** button.

![Sync_Settings](/assets/images/2016/02/Sync_Settings-1024x377.png)

Here you’ll see a whole host of advanced synchronization options that you can change. Click on the **Mapped Attributes** button at the top, then select the dropdown next to **email**. Select **Enter Custom Input…** from the menu.

![Mapped_Attributes](/assets/images/2016/02/Mapped_Attributes-293x300.png)

Now, enter the new Active Directory attribute name that you want to retrieve the email address from. In this example, the new attribute is named **otherMailbox**. Click **Save &amp; Sync** to save your settings and update the user accounts.

![Enter_New_Attribute_Name](/assets/images/2016/02/Enter_New_Attribute_Name-1024x635.png)

You’ll now be given the opportunity to review the proposed changes once the sync is completed. You can see in this example, there are 4 AD accounts that will have their attribute mappings updated. Click **Sync Directory**.

![Review_Sync_Settings](/assets/images/2016/02/Review_Sync_Settings-1024x620.png)

Once the sync is completed (this may take some time, depending on how many objects were being updated and the size of your AD, etc) go back to the **Administration &gt; Directory Users and Groups** view and find your user account again. You’ll notice that the email address has now been updated to reflect the contents of the preferred attribute.

![Updated_EMail_Address](/assets/images/2016/02/Updated_EMail_Address-1024x248.png)

Pretty cool, huh?

This post was brought to you by Terrapin Beer Co’s Poivre Potion, a very unique dry-hopped pink peppercorn Saison. I love the way the spicy and sweet notes of the peppercorns play off the bitterness of the hops and the farmhouse funk of the Saison yeast strains. Delicious and easy to drink.

![Terrapin_Poivre_Potion](/assets/images/2016/02/Terrapin_Poivre_Potion-225x300.jpg)

Happy automating!

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=vIDM+Attribute+Mapping+in+vRA+7)</div></div>