---
id: 405
title: 'vRealize Automation 7 Management Pack for vRealize Operations'
date: '2016-03-03T17:58:26-08:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=405'
permalink: /2016/03/vrealize-automation-7-management-pack-for-vrealize-operations/
categories:
    - Integration
    - Uncategorized
tags:
    - dashboard
    - integration
    - 'management pack'
    - tenant
    - vra
    - vrealize
    - 'vrealize automation'
    - 'vrealize operations'
    - vrops
---

If you’re an SDDC administrator, you probably already know about the power and operational visibility that vRealize Operations brings to your environment. With the newly-released vRealize Automation 7 Management Pack for vRealize Operations, that operational visibility can be extended to be tenant-aware and help monitor your vRA environment in a whole new way.

This new Management Pack gives you comprehensive visibility into both performance and capacity metrics of a vRA tenant’s business groups and underlying cloud infrastructure. By combining these new metrics with the custom dashboarding capabilities of vRealize Operations, you gain an unprecedented level of flexibility and insight when monitoring these complex environments.

The purpose of this post is to walk you through the implementation of this new Management Pack – so, let’s get right to it.

You can download the Management Pack from the VMware Solution Exchange [here](https://solutionexchange.vmware.com/store/products/management-pack-for-vrealize-automation).

# Part 1: Enabling vROps as your Metrics Provider

First, let’s review what you’ll see before you integrate vRA and vROps. Looking at the details of any deployed item, you can see the highlighted white space – space that can definitely be put to more productive use.

![Item_Details_Before_Integration](/assets/images/2016/03/Item_Details_Before_Integration.png)

Assuming you’re logged in as a vRA Tenant Administrator, click on the **Administration** tab, then the **Reclamation** button in the menu at the left. Select **Metrics Provider** and you’ll see the configuration panel for the vROps endpoint. Fill in the appropriate details for your vROps instance and click **Test Connection**. Once it succeeds, click **Save**.

![Set_Up_Metrics_Provider](/assets/images/2016/03/Set_Up_Metrics_Provider.png)

You will probably be prompted to accept the SSL certificate offered up by your vROps instance. Click **OK** to accept the certificate, provided you trust it!

![Accept_vROps_Cert](/assets/images/2016/03/Accept_vROps_Cert.png)

Now, if you click on the **Tenant Machines** option to the left, you’ll be presented with a list of all of your provisioned machines. You can see that now there’s a **Health** status badge for each machine. In my case, the Health is reporting an “Immediate” (orange) status for many of my virtual machines, due to very heavy utilization in my lab. You can also see the average CPU, Memory and Network consumption for each machine – data pulled directly from vROps. This consumption data can be used directly from within this view to initiate reclamation requests. For example, if a VM was identified here as idle, the VM owner could be notified and the resources recovered.

![Tenant_Machines_View](/assets/images/2016/03/Tenant_Machines_View.png)

Click back to the **Items** tab and view the same object you looked at earlier. You will see that the white space now contains a vROps-driven **Health** badge, with information about any possible issues. When you’re ready, log out of your vRA instance.

![Item_Details_After_Integration](/assets/images/2016/03/Item_Details_After_Integration.png)

# Part 2: Configuring vRealize Automation

You’ll need to log in as the default administrator for this next step – **administrator@vsphere.local**

![Log_In_vRA_Default_Administrator](/assets/images/2016/03/Log_In_vRA_Default_Administrator.png)

Click on the **Administration** tab, followed by the **Tenants** menu button at the left. Locate the **Tenant** that you plan to link vROps to and **Edit** it. In this example, I am modifying the **vsphere.local** Tenant.

![Locate_Target_Tenant](/assets/images/2016/03/Locate_Target_Tenant.png)

Now, select the **Local Users** tab. Click the **+New** button to add a new user and fill in the requested details. In this case, my new username is **“vropsmp**” – and since we are creating this local user in the **vsphere.local** tenant, the full account is “**vropsmp@vsphere.local**“. Click **OK** and then **Next**.

![Add_New_Local_User](/assets/images/2016/03/Add_New_Local_User.png)

This will place you on the **Administrators** tab. Using the **Search** boxes, find and add your new local account to both the **Tenant Administrators** and the **IaaS Administrators** role. Click **Finish** when you’re done, and then **Log Out** of vRA.

![Assign_Tenant_Rights](/assets/images/2016/03/Assign_Tenant_Rights.png)

Now you’ll need to log back in as your normal vRA **Tenant Administrator** to finish the configuration.

![Log_In_vRA_Tenant_Admin](/assets/images/2016/03/Log_In_vRA_Tenant_Admin.png)

Click the **Infrastructure** tab, then **Endpoints** from the menu on the left. Select **Fabric Groups** from the sub-menu and then click to edit your **Fabric Group**. In this example, the Fabric Group is named **Dev Cluster**.

![Navigate_To_Fabric_Groups](/assets/images/2016/03/Navigate_To_Fabric_Groups.png)

Search for and add your new local user to the list of **Fabric Administrators**. Remember, in this example the user is named **vropsmp@vsphere.local**. Click **OK** to save the Fabric Group.

![Edit_Fabric_Group](/assets/images/2016/03/Edit_Fabric_Group.png)

Now, click on the **Administration** tab, followed by **Users &amp; Groups** from the menu on the left. Select the **Directory Users and Groups** sub-menu and search for your new local user. Click the user’s name to edit it.

![Navigate_To_Directory_Users](/assets/images/2016/03/Navigate_To_Directory_Users.png)

In the list to the right titled “**Add roles to this User**“, scroll down until you find the **Software Architect** role. Select it and then click **Finish** to save the account.

![Assign_Software_Architect_Role](/assets/images/2016/03/Assign_Software_Architect_Role.png)

# Part 3: Configuring vRealize Operations

Once you’ve downloaded the new Management Pack (again, found [here](https://solutionexchange.vmware.com/store/products/management-pack-for-vrealize-automation)) you’ll need to import it into vROps and configure it to retrieve data from vRA.

Log in to vROps with an administrative user account.

![Log_In_vROps](/assets/images/2016/03/Log_In_vROps.png)

Click on the **Administration** tab, and ensure **Solutions** is selected. Click the **+** symbol to import a new Solution.

![Import_New_Solution](/assets/images/2016/03/Import_New_Solution.png)

Click the **Browse** button to select the downloaded Management Pack, then click **Upload**.

<span style="color: #ff0000;">NOTE! If you already had the earlier vROps Management Pack for vRA installed, you may have to do a “force install” by selecting the first checkbox.<span style="color: #000000;"> This is because the version number scheme was changed, and vROps recognizes the **NEW** MP as being an **OLDER** version. This is normal, if a bit cumbersome.</span></span>

Click **Next** when the upload is verified and you are ready to proceed.

![Upload_New_Solution](/assets/images/2016/03/Upload_New_Solution.png)

Accept the EULA (after reading it carefully first, of course) and click **Next** again.

![Accept_EULA](/assets/images/2016/03/Accept_EULA.png)

The installation will run for a while. When it shows “Completed”, click **Finish**.

![Complete_Installation](/assets/images/2016/03/Complete_Installation.png)

Locate the new Management Pack in the list of Solutions and highlight it. Click the **Configure** icon (gears) to bring up the configuration dialog. Fill in a **Display Name** and **Description** as well as your **vRA URL** and the name of the **Tenant** you want to connect to. In this example, the Tenant is **vsphere.local**. Click the **+** sign to start setting up credentials next.

![Configure_Solution_Basics](/assets/images/2016/03/Configure_Solution_Basics.png)

Fill in the credential details as shown – your **SysAdmin** should be the **administrator@vsphere.local** administrative account, and your **SuperUser** will be the local user you created at the beginning of these steps. In this example, that local user is **vropsmp@vsphere.local**. Click **OK** when you’re done.

![Manage_Credential](/assets/images/2016/03/Manage_Credential.png)

Click the ‘**Test Connection**‘ button. You’ll be prompted with two SSL certificate dialogs – accept them both, if you trust the certificates. You see two because the Management Pack is communicating with both your core vRA appliance as well as your IaaS server(s).

![Test_Connection_Accept_Cert_1](/assets/images/2016/03/Test_Connection_Accept_Cert_1.png)

![Accept_Cert_2](/assets/images/2016/03/Accept_Cert_2.png)

If you’ve set everything up properly, you’ll see a message like this one. Click **OK**.

![Test_Successful](/assets/images/2016/03/Test_Successful.png)

Click on **Save Settings** to save your adapter configuration. You’ll be prompted with a “Save Successful” dialog – click **OK** here as well – then click **Close**.

![Save_Solution_Settings](/assets/images/2016/03/Save_Solution_Settings.png)

If everything’s gone according to plan, you should now see that your Management Pack is configured and receiving data from your vRA instance.

![Solution_Details](/assets/images/2016/03/Solution_Details.png)

# Part 4: Reviewing Dashboards

Now that all of the configuration is complete, you’re ready to start consuming the rich data exposed by your new integration. Click on the **Home** tab in vROps, followed by the drop-down arrow for the **Dashboard List**. Hover over the **vRealize Automation** sub-menu to see the 4 available default dashboards.

![Navigate_To_vRA_Dashboards](/assets/images/2016/03/Navigate_To_vRA_Dashboards.png)

The **vRealize Automation Overview** dashboard shows information about the entire vRA instance – including component health and a whole host of metrics about each individual component of the instance. This is useful for troubleshooting and analyzing performance across your entire implementation of the vRA stack.

![vRealize_Automation_Overview](/assets/images/2016/03/vRealize_Automation_Overview.png)

The **vR Automation Tenant Overview** dashboard provides exactly that – an overview of the various risk and health metrics pertinent to each configured vRA Tenant.

![vR_Automation_Tenant_Overview](/assets/images/2016/03/vR_Automation_Tenant_Overview.png)

The **vR Automation Cloud Infrastructure Monitoring** dashboard allows you to see what impact infrastructure issues are having on tenant virtual machines, and what outstanding alerts may be present for those machines and infrastructure.

![vR_Automation_Cloud_Infrastructure_Monitoring](/assets/images/2016/03/vR_Automation_Cloud_Infrastructure_Monitoring.png)

Finally, the **vR Automation Top-N Dashboard** provides highlight Top-N metrics, such as the most popular Blueprints, most wasteful Tenants, the Business Group with the most alerts, etc.

![vR_Automation_Top-N_Dashboard](/assets/images/2016/03/vR_Automation_Top-N_Dashboard.png)

And, of course, all of the objects which are exposed by the Management Pack can be viewed in the **vRealize Automation Environment** view. These objects can all be referenced by Super Metrics, or custom dashboards, or scheduled reports – but those are all beyond the scope of this guide.

![vRealize_Automation_Environment](/assets/images/2016/03/vRealize_Automation_Environment.png)

That just about wraps it up – except, of course, for the most important part…

This post was brought to you by New Helvetia Brewing Company’s Mystery Airship 2.0 Imperial Chocolate Porter, brewed with Ginger Elizabeth’s Oaxacan Spicy Chocolate. This is quite possibly the single greatest beer I have ever tried – the darkness of the porter is supplemented by the brightness of the ginger and creamy feel of the chocolate. The flavors dance on your palate and then vanish in a fog of lingering, dark spice. I honestly think I found my desert island beer!

![New_Helvetia_Ginger_Elizabeth_Porter](/assets/images/2016/03/New_Helvetia_Ginger_Elizabeth_Porter-768x1024.jpg)

Happy Automating!

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=vRealize+Automation+7+Management+Pack+for+vRealize+Operations)</div></div>