---
id: 238
title: 'Creating new vRA Workloads in a specific AD OU'
date: '2015-07-31T10:45:21-07:00'
author: jon
layout: post
image: /assets/images/cropped-va-header.jpg
guid: 'http://www.vaficionado.com/?p=238'
permalink: /2015/07/creating-new-vra-workloads-in-a-specific-ad-ou/
categories:
    - Integration
    - Tech
tags:
    - automation
    - integration
    - vra
    - vro
    - workflow
---

This week, I’ve had several customers individually approach me with this question – how can they specify the OU which a Windows VM should land in when it’s created via vRA?

This is a great question and a very important operational task to accomplish – OU membership determines so much vital configuration for a Windows machine.

It seems like most of these customers have a tendency to assume they are going to create the VM first, and relocate it to a new OU later. But there’s a much more streamlined way to do it. By binding a workflow to the IaaS BuildingMachine lifecycle stage, you can pre-stage the computer object in AD before it’s even provisioned. That way, when it first adds to the domain it will already be present in the correct OU. This also has the added benefit of ensuring all group policies are inherited right away, rather than requiring additional reboots.

I’ve put together a quick example [here](/vaficionado/assets/downloads/2015/01/Create-OU-and-Stage-Machine.workflow "here") that should help you see how to do just this.

To use the example workflow attached above, you must already have your vRO instance registered with vRA and the extensibility customizations installed. We also assume that you have correctly configured the Active Directory plugin, and that the example vRA blueprint you will deploy has a vSphere Customization Specification attached which adds the VM to your AD domain.

First, import the workflow into a vRO folder of your choosing.

Then, browse to it in the workflow tree and select it. On the **General** tab, you can see there are two Attributes that must be updated for your own envirionment. Enter the AD Domain Name as the value for **domainName** and select the parent OU you want new OUs to be created in for **ou1**. You can see in my example, the domain is **lab.virtualwin.org** and the Parent OU is **Lab Machines**.

[![Configure_Workflow](/assets/images/2015/07/Configure_Workflow-1024x579.png)  ](/assets/images/2015/07/Configure_Workflow.png)[(Click for larger image)](/assets/images/2015/07/Configure_Workflow.png)

Next, use the **Assign a state change workflow to a blueprint and its virtual machines** workflow to attach the new workflow to the **BuildingMachine** stage of a test blueprint. This workflow is located under **root &gt; Library &gt; vCloud Automation Center &gt; Infrastructure Administration &gt; Extensibility** in the Workflow tree.

To do this, right-click the workflow and select **Start Workflow.**

[![Start_Workflow](/assets/images/2015/07/Start_Workflow.png)](/assets/images/2015/07/Start_Workflow.png)

Choose **BuildingMachine** as the stub to enable and choose your **IaaS host**. Remember that we are assuming your IaaS Plugin is already configured. If you don’t see any hosts in this list, you still need to do that! Click **Next**.

[![Select_Stub_and_vRA_Host](/assets/images/2015/07/Select_Stub_and_vRA_Host.png)](/assets/images/2015/07/Select_Stub_and_vRA_Host.png)

Now, select the blueprint(s) you wish to add this workflow to. In this example the selected blueprint is “**Add to OU Test**” and click **Next**.

[![Select_Blueprints](/assets/images/2015/07/Select_Blueprints.png)](/assets/images/2015/07/Select_Blueprints.png)

On the last screen, you will be prompted to select the workflow and some final options. Choose the new workflow you just imported (in this example, it is **Create OU and Stage Machine**). Be sure to choose **Yes** for **Add vCO workflow inputs as blueprint properties** and then click **Submit**.

[![Add_vRO_Workflow_Inputs](/assets/images/2015/07/Add_vRO_Workflow_Inputs.png)](/assets/images/2015/07/Add_vRO_Workflow_Inputs.png)

The workflow will complete. Now, switch to your vRealize Automation console and edit the blueprint which you just attached the workflow to. Select the **Properties** tab. You will be presented with a list of properties, some of which need to be adjusted. The total list of properties you see may vary from environment to environment. Here, we need to delete the following 4 properties:

- **ExternalWFStubs.BuildingMachine.vCACHost**
- **ExternalWFStubs.BuildingMachine.vCACVm**
- **ExternalWFStubs.BuildingMachine.vCenterVm**
- **ExternalWFStubs.BuildingMachine.virtualMachineEntity**

And also edit the one named **ExternalWFStubs.BuildingMachine.ouName** so that **Prompt User** is checked.

[![Blueprint_Custom_Properties_Before](/assets/images/2015/07/Blueprint_Custom_Properties_Before.png)](/assets/images/2015/07/Blueprint_Custom_Properties_Before.png)

When you’re done, the properties should look more like this:

[![Blueprint_Custom_Properties_After](/assets/images/2015/07/Blueprint_Custom_Properties_After.png)](/assets/images/2015/07/Blueprint_Custom_Properties_After.png)

Now, let’s make that variable a little more friendly. Open the **Property Dictionary** from the menu on the left. Click on **New Property Definition** and fill in the data as follows:

- Name: **ExternalWFStubs.BuildingMachine.ouName**
- Display Name: **Create New OU to host new VM**
- Control Type: **Textbox**
- Required: **Yes**

[![Property_Dictionary](/assets/images/2015/07/Property_Dictionary.png)](/assets/images/2015/07/Property_Dictionary.png)

That’s it! Now, if you navigate to your vRA Catalog and request the blueprint you’ve been working on, you should see something similar to this.

[![Request_New_Item](/assets/images/2015/07/Request_New_Item.png)](/assets/images/2015/07/Request_New_Item.png)

Click Submit and wait for provisioning to complete. When you’re done, you will see the new machine in your **Items** tab as usual:

[![Deployed_Items](/assets/images/2015/07/Deployed_Items.png)](/assets/images/2015/07/Deployed_Items.png)

But if you check out your Active Directory, you should also see that the new OU you selected was created, and the new machine was created inside it!

[![AD_Properties](/assets/images/2015/07/AD_Properties.png)](/assets/images/2015/07/AD_Properties.png)

Now, this example workflow is a very quick demonstration of concept. It doesn’t have any error handling (and suffice it to say, should NOT be used in any production environments and is provided without support or warranty of any kind) – but it should show you how a seemingly complex task like this can be accomplished relatively easily. The logic in the workflow could easily be amended to remove the OU creation step. ASD and vRO Dynamic Types could be leveraged to provide the user a list of OUs to choose from, rather than a free-form textbox. The sky’s the limit when it comes to vRA extensibility!

Today’s spicy orchestration experience was brought to you by the Habanero Mojito at [Havana, Walnut Creek](https://havanarestaurant.net/main/). 

[![Jon_Kate_Havana](/assets/images/2015/07/IMG_5723-e1438365368866-300x225.jpg)](/assets/images/2015/07/IMG_5723-e1438365368866.jpg)

I hope this post has been useful.