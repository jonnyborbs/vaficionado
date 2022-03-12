---
id: 61
title: 'Monitoring vRealize Automation with vRealize Operations and Hyperic'
date: '2015-01-25T15:36:38-08:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=61'
permalink: /2015/01/vrealize-operations-inventory-tree-for-vrealize-automation/
categories:
    - Stories
    - Tech
tags:
    - hyperic
    - integration
    - vra
    - vrops
    - Wine
---

Have you ever deployed vRealize Automation? If so, then you know that it has a highly complex architecture, made up of dozens of individual components – and has historically been a bit of a hassle to properly monitor.

That said, there’s good news for administrators who have both the vRealize Automation and the vRealize Operations Advanced edition – VMware has released a brand-new way to integrate the two, via the vRealize Automation Management Pack. This new management pack brings detailed application-aware monitoring of the full architecture of vRealize Automation, and includes a set of plugins for vRealize Hyperic as well as an updated vRealize Operations Management Pack for Hyperic. With the helo of this management pack and set of plugins, users gain the following capabilities:

- vRealize Hyperic platform service monitoring for vRealize Automation related services
- An inventory tree object in vRealize Operations Manager specifically tailored to vRealize Automation
- A set of pre-defined symptoms, alerts, and recommendations for vRealize Operations specifically revolving around vRealize Automation monitoring

Before diving into implementation details, here are a couple of quick screenshots of what you can expect after deploying the new management pack and plugins.

[![vRealize Automation Environment View in vRealize Operations](/vaficionado/assets/images/2015/01/vRA-vROps-EnvironmentList.png)](/vaficionado/assets/images/2015/01/vRA-vROps-EnvironmentList.png)  
(Click the above image for a larger version)

[![vRealize Automation Inventory Tree View in vRealize Operations](/vaficionado/assets/images/2015/01/vRA-Tree.png)](/vaficionado/assets/images/2015/01/vRA-Tree.png)

As you can see, it monitors the following high-level capabilities and their sub components :

- vRealize Automation Appliance
- vRealize Automation Infrastructure-as-a-Service (IaaS) Server
- vRealize Business (Formerly ITBM) Appliance
- vSphere Single Sign-On (SSO)
- vRealize Orchestrator

Here’s today’s obligatory wine tie-in. Given to a friend when he departed the employ of [Viansa](https://www.viansa.com/), this bottle of 2005 Ossidiana was signed by his friends and co-workers from all aspects of the winery. It’s also a finely blended Bordeaux – representing the perfect marriage of the 5 noble French grapes. The blend is proprietary and not disclosed, but it was clearly more than a little Cab. All sorts of grapes, styles, workers, techniques and technology coming together to produce one harmonious and easily enjoyable product. Can you see why I was reminded of this exciting new marriage of Automation and Management when we opened this bottle last night?

[![IMG_4734](/vaficionado/assets/images/2015/01/IMG_4734-517x1024.jpg)](/vaficionado/assets/images/2015/01/IMG_4734.jpg)

All that aside, let’s get into some of the nuts and bolts of implementing this new connection.

First, we must assume that you have functioning instances of vRealize Automation **6.1 or above**, vRealize Operations Manager **6.0 or above** and vRealize Hyperic deployed. Getting all of those up and running in your environment is outside the scope of this article. You will also need Hyperic agents deployed to all of the appliances and servers involved in the vRealize Automation stack. These can include (but are not limited to):

- vSphere SSO
- vRealize Automation Appliance
- vRealize Orchestrator Appliance
- vRealize Business Appliance
- vRealize Automation Infrastructure-as-a-Service (IaaS) Server
- Any additional Distributed Execution Managers (DEM)
- External vRealize Automation IaaS Database Servers

Deploying these agents is also outside the scope of this article. Look for a forthcoming post on getting the agents onto the VMware appliances.

From there, you will log into your vRealize Hyperic server as an administrator with the rights to install plugins. Select the **Administration** tab and the **Plugin Manager** link.

Now, if you are currently running vRealize Hyperic 5.8.4, you may see some existing custom vRealize XML Plugins already present in the environment. These need to be removed first, and look like the following. If you don’t see these plugins, skip this step.

[![vRealize Hyperic XML Plugins for vRealize Monitoring](/vaficionado/assets/images/2015/01/vR-XML-Hyperic-1024x169.png)](/vaficionado/assets/images/2015/01/vR-XML-Hyperic.png)  
(Click the above image for a larger version)

To delete them, simply select the **Checkbox** to the left of each plugin and select **Delete Selected Plugin(s)** from the bottom left corner. This may take some time to complete.

Now click the **Add/Update Plugin(s)** button in the lower right corner and upload the two new **.JAR** plugin files.

After that’s complete, you should see something like the following image. Notice the two new custom JAR plugins, highlighted in red.

[![vRealize Hyperic JAR Plugins for vRealize Automation](/vaficionado/assets/images/2015/01/HQPlugins-1024x522.png)](/vaficionado/assets/images/2015/01/HQPlugins.png)  
(Click the above image for a larger version)

Now, switch over to your vRealize Operations console. Log in with a user who has the administrative rights to update solutions. Navigate to the **Administration** tab and select **Solutions** from the navigation pane. Click the **Green +** (Add) in the upper left corner of the solutions pane. Follow the wizard that is produced to install or update the solution.

[![vRealize Operations Solutions](/vaficionado/assets/images/2015/01/vROps-Solutions.png)](/vaficionado/assets/images/2015/01/vROps-Solutions.png)

If you already had the vRealize Hyperic solution installed and working, you’re done with this part! If this is your first time installing the solution, you will need to configure the adapter instance. To do so, highlight the **vRrealize Hyperic** solution and click on the **Gears** icon in the upper left. Fill in the requested details about your vRealize Hyperic server as seen here, of course using your own settings. Test and save the settings.

[![vRealize Hyperic Adapter Configuration](/vaficionado/assets/images/2015/01/HypericConfig.png)](/vaficionado/assets/images/2015/01/HypericConfig.png)

Now all you need to do is wait for vRealize Hyperic to auto-discover your new services. Check your Hyperic dashboard after a few minutes and import them; after a few more minutes they will start appearing in your vRealize Operations Manager.

You can confirm which vRealize Hyperic metrics are flowing into vRealize Operations by logging into it with an administrative account, then navigating to the **Administration** tab and **Environment Overview**. Expand the **Adapter Instances** and then your **Hyperic Adapter Instance**. You will see the name of the Hyperic instance that you configured in the last step – select it and view the related metrics.

[![vRealize Operations Manager Environment Overview](/vaficionado/assets/images/2015/01/EnvironmentOverview-1024x535.png)](/vaficionado/assets/images/2015/01/EnvironmentOverview.png)  
(Click the above image for a larger version)

That’s all there is to it – now you can navigate to your vRealize Operations **Content** tab and view the **vRealize Automation** inventory tree.

[![vRealize Operations Inventory Trees](/vaficionado/assets/images/2015/01/InventoryTrees.png)](/vaficionado/assets/images/2015/01/InventoryTrees.png)

From here you can explore the related tabs – environment, analysis, troubleshooting, etc – and begin leveraging the wealth of new metrics at your fingertips.

The new vRealize Operations and vRealize Hyperic integration packs can be downloaded from the VMware Solutions Exchange [here](https://solutionexchange.vmware.com/store/products/vmware-vrealize-operations-management-pack-for-vmware-vrealize-automation) and [here](https://solutionexchange.vmware.com/store/products/management-pack-for-vrealize-hyperic#.VNTbk3aM67Q).

Enjoy!

You can also see this article cross-posted on the VMware Management Blog at https://blogs.vmware.com/management/2015/02/monitoring-vrealize-automation-vrealize-operations-vrealize-hyperic.html

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=Monitoring+vRealize+Automation+with+vRealize+Operations+and+Hyperic)</div></div>