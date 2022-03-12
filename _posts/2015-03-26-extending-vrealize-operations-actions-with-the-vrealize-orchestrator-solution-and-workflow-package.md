---
id: 109
title: 'Extending vRealize Operations Actions with the vRealize Orchestrator Solution and Workflow Package'
date: '2015-03-26T08:54:00-07:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=109'
permalink: /2015/03/extending-vrealize-operations-actions-with-the-vrealize-orchestrator-solution-and-workflow-package/
categories:
    - Integration
    - Tech
tags:
    - integration
    - 'operations management'
    - orchestrator
    - vro
    - vrops
---

When vRealize Operations Management 6.0 was released, VMware increased the flexibility afforded to administrators by adding the concepts of symptoms, recommendations and actions to the product. As you might expect, symptoms are thresholds or characteristics that define when a problem may have occurred or additional guidance may be needed. Recommendations are a customizable way to define what that additional guidance might be – and actions allow you to automate and carry out that guidance.

Since then, one of the most frequent questions from my customers has been “When will we be able to use vRealize Orchestrator for these?”

I’m pleased to report that VMware has now enabled that capability via the vRealize Orchestrator Solution and Workflow Package for vRealize Operations. This package is [available](https://solutionexchange.vmware.com/store/products/vro-solution-and-workflow-package-for-vrealize-operations-manager#.VRQQ2UaM67Q) at the [VMware Solution Exchange](https://solutionexchange.vmware.com/store) right now, and the purpose of this post is to guide you through the installation and configuration of it. The package adds many frequently-requested workflows, including:

- Decommission a Host
- Place a Host into Maintenance Mode
- Perform a Power Off or Reboot on a Host
- Manage VM or VM Group Snapshots
- Migrate a VM or VM Group
- Power Off, Power On or Reboot a VM or VM Group
- Reconfigure a VM or VM Group (CPU and Memory settings)
- Upgrade the VMware Tools for a VM or VM Group

Clicking the links above will bring you to the Solution Exchange portal where you can read more about and download the package. Click the blue “**Try**” button to initiate the download.

[![VSX_Download_vRealize_Orchestrator_Solution_and_Workflows_for_vRealize_Operations](/vaficionado/assets/images/2015/03/VSX_Download_vRealize_Orchestrator_Solution_and_Workflows_for_vRealize_Operations1-1024x625.png)](/vaficionado/assets/images/2015/03/VSX_Download_vRealize_Orchestrator_Solution_and_Workflows_for_vRealize_Operations1.png)

Once you have downloaded and extracted the ZIP file, it’s time to start the installation. The first thing you’ll want to do is ensure that both your vRealize Orchestrator and vRealize Operations Manager are registered to the same vCenter instance. This can be done by comparing the data shown in the two screenshots below.

[![Validate_vRealize_Operations_vCenter_Connection](/vaficionado/assets/images/2015/03/Validate_vRealize_Operations_vCenter_Connection.png)](/vaficionado/assets/images/2015/03/Validate_vRealize_Operations_vCenter_Connection.png)

[![Validate _vRealize_Orchestrator_vCenter_Connection](/vaficionado/assets/images/2015/03/Validate-_vRealize_Orchestrator_vCenter_Connection.png)](/vaficionado/assets/images/2015/03/Validate-_vRealize_Orchestrator_vCenter_Connection.png)

As you can see above, both systems are taking to the same vCenter. We’re ready to begin!

First, you will need to import the Workflow package into your vRealize Orchestrator instance. Start by logging in to the **Orchestrator Client**.

[![Log_Into_vRealize_Orchestrator](/vaficionado/assets/images/2015/03/Log_Into_vRealize_Orchestrator.png)](/vaficionado/assets/images/2015/03/Log_Into_vRealize_Orchestrator.png)

Ensure that your client view is set to **Administer**

[![Switch_to_Administrator_View](/vaficionado/assets/images/2015/03/Switch_to_Administrator_View.png)](/vaficionado/assets/images/2015/03/Switch_to_Administrator_View.png)

Then, click on the **Import Package** button in the upper left of the right-hand panel.

[![Import_vRealize_Orchestrator_Package](/vaficionado/assets/images/2015/03/Import_vRealize_Orchestrator_Package.png)](/vaficionado/assets/images/2015/03/Import_vRealize_Orchestrator_Package.png)

Select the **Remediation Actions Package** (default filename is **com.vmware.vrops.remediationactionsall-v15.package**) and select **Open**

[![Select_Package_to_Import](/vaficionado/assets/images/2015/03/Select_Package_to_Import.png)](/vaficionado/assets/images/2015/03/Select_Package_to_Import.png)

You will be prompted to verify the software signature. Continue by selecting **Import**

[![Accept_Package_Signature](/vaficionado/assets/images/2015/03/Accept_Package_Signature.png)](/vaficionado/assets/images/2015/03/Accept_Package_Signature.png)

vRealize Orchestrator will then present you with a list of all of the new and changed elements that this package import will affect. No changes here are necessary, simply continue by clicking **Import Selected Elements**

[![Import_vRealize_Orchestrator_Package_Elements](/vaficionado/assets/images/2015/03/Import_vRealize_Orchestrator_Package_Elements.png)](/vaficionado/assets/images/2015/03/Import_vRealize_Orchestrator_Package_Elements.png)

Once the import completes, you will be able to view the new workflows. Click the **Workflows** tab to verify that there’s a whole bunch of new vRealize Operations Manager goodness present.

[![View_Imported_Workflows](/vaficionado/assets/images/2015/03/View_Imported_Workflows-1024x246.png)](/vaficionado/assets/images/2015/03/View_Imported_Workflows.png)

You can also verify that the new workflows are present by switching back to the **Run** view, clicking the **Workflows** tab and expanding the new **vRealize Operations Manager** folder. You can see I already have a ton of great workflows by my friends Eric at [Cloud Relevant](https://cloudrelevant.com/) and Sid at [Daily Hypervisor](https://dailyhypervisor.com/) in here.

[![Switch_to_Run_View_and_View_New_Workflows](/vaficionado/assets/images/2015/03/Switch_to_Run_View_and_View_New_Workflows.png)](/vaficionado/assets/images/2015/03/Switch_to_Run_View_and_View_New_Workflows.png)

That’s it for the vRealize Orchestrator side of things. Now you will need to switch over to your vRealize Operations Manager portal. Log in as a user with appropriate rights to add/update solutions. An admin user will work nicely.

Click on the **Administration** button, followed by the **Solutions** section. Then, click the **Green +** to add a new solution.

[![Import_New_vRealize_Operations_Solution](/vaficionado/assets/images/2015/03/Import_New_vRealize_Operations_Solution.png)](/vaficionado/assets/images/2015/03/Import_New_vRealize_Operations_Solution.png)

Select the solution file using the **Browse** button and click **Upload**. Once the upload completes and the PAK file has been verified, click **Next** to proceed with the installation.

[![Select_Solution_PAK](/vaficionado/assets/images/2015/03/Select_Solution_PAK.png)](/vaficionado/assets/images/2015/03/Select_Solution_PAK.png)

Accept the EULA and click **Next** again. Wait for the installation to complete, then select **Finish**

[![Complete_Solution_Installation](/vaficionado/assets/images/2015/03/Complete_Solution_Installation.png)](/vaficionado/assets/images/2015/03/Complete_Solution_Installation.png)

You can now verify that your new solution is installed by locating the **vRealize Orchestrator Actions Adapter** in the solutions list. Note that you may have to scroll down to find it, if you have several solutions installed. You may also notice that the adapter instance is not yet configured. Let’s tackle that next!

[![Verify_New_Solution_is_Installed](/vaficionado/assets/images/2015/03/Verify_New_Solution_is_Installed.png)](/vaficionado/assets/images/2015/03/Verify_New_Solution_is_Installed.png)

To configure the adapter instance, ensure that the **vRealize Orchestrator Actions Adapter** is still selected, then click the **Gears** icon at the top, next to the **Green +** we clicked a few steps back.

Give your new adapter a name, and enter the IP or hostname of your vRealize Orchestrator instance. Be sure to use the same Orchestrator instance as we verified at the beginning of this process. Click the **Green +** to add credentials for the instance.

[![Configure_New_vRealize_Operations_Solution](/vaficionado/assets/images/2015/03/Configure_New_vRealize_Operations_Solution.png)](/vaficionado/assets/images/2015/03/Configure_New_vRealize_Operations_Solution.png)

Enter your credentials and click **OK**

[![Add_New_Credential](/vaficionado/assets/images/2015/03/Add_New_Credential.png)](/vaficionado/assets/images/2015/03/Add_New_Credential.png)

Next, click **Test Connection**. You may be presented with a certificate warning – click **OK** if you trust the certificate, and then your test should be successful!

[![Accept_vRealize_Orchestrator_Certificate](/vaficionado/assets/images/2015/03/Accept_vRealize_Orchestrator_Certificate.png)](/vaficionado/assets/images/2015/03/Accept_vRealize_Orchestrator_Certificate.png)

[![Solution_Test_Successful](/vaficionado/assets/images/2015/03/Solution_Test_Successful.png)](/vaficionado/assets/images/2015/03/Solution_Test_Successful.png)

Save your new adapter by clicking **Save Settings** and finally **Close** the configuration dialog.

That’s it for the installation! You can verify that the new actions are present by clicking on the **Content** tab inside vRealize Operations and selecting **Actions** from the list on the left. If all went well, you should see the 8 new actions present. These can now be combined with symptoms and recommendations to unlock many new possibilities for remediation inside your environment.

[![View_New_Available_Actions](/vaficionado/assets/images/2015/03/View_New_Available_Actions-1024x558.png)](/vaficionado/assets/images/2015/03/View_New_Available_Actions.png)  
(Click for larger image)

Since it’s not even 9am yet, today’s post will be brought to you by the Zesty Bacon Bloody Mary from the Boon Fly Cafe in Napa, CA. This exceptional libation combines top-shelf Vodka with Boon Fly’s own special spice blend, a celery salt rim and a massive slab of applewood smoked bacon to top it all off. Paired with Boon Fly’s fresh made donuts, it’s the best breakfast in the valley. Bloody Marys also have the (dubious?) honor of being the drink that’s OK to have first thing in the morning. After all, you’re not an alcoholic, you’re just a little tired.

[![Bacon_Bloody_Mary_Boon_Fly_Cafe](/vaficionado/assets/images/2015/03/Bacon_Bloody_Mary_Boon_Fly_Cafe-546x1024.jpg)](/vaficionado/assets/images/2015/03/Bacon_Bloody_Mary_Boon_Fly_Cafe.jpg)

I hope this guide has proved useful and that you have a chance to head out to [Boon Fly](https://www.boonflycafe.com/) and try their delicious concoctions.

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=Extending+vRealize+Operations+Actions+with+the+vRealize+Orchestrator+Solution+and+Workflow+Package)</div></div>