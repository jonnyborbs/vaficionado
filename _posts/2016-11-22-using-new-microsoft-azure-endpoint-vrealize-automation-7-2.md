---
id: 469
title: 'Using the new Microsoft Azure Endpoint in vRealize Automation 7.2'
date: '2016-11-22T04:10:03-08:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=469'
permalink: /2016/11/using-new-microsoft-azure-endpoint-vrealize-automation-7-2/
categories:
    - Integration
    - Stories
    - Tech
tags:
    - automation
    - azure
    - extensibility
    - hands-on
    - labs
    - 'public cloud'
    - vra
    - 'vrealize automation'
---

After months of planning and development, vRealize Automation 7.2 finally went GA today, and it feels so good! One of the most anticipated and spotlight features of this new release was the Endpoint for Microsoft Azure. I had the privilege of working very closely with the team who delivered this capability, and thought I would take some time to develop a brief POC type guide to help get you started using the new Microsoft Azure Endpoint in vRealize Automation 7.2

This guide will walk you through configuring a brand-new Azure subscription to support a connection from vRealize Automation, then help you set up your vRA portal and finally design and deploy a simple Blueprint. We will assume that you have already set up your Azure subscription. If not, you can get a free trial at <https://azure.microsoft.com/en-us/free/> – and that you have a vRealize Automation 7.2 install all ready to go. Certain steps outlined in this guide make assumptions that your vRA configuration is rather basic and is not in production. Please use them at your own risk and consider any changes you make before you make them!

# Part 1: Configuring Azure

![SelectSubscription](/assets/images/2016/11/SelectSubscription.png)  
Once you have your subscription created, log in to the Azure portal and click on the **Key** **(Subscriptions)** icon in the left-hand toolbar. These icons can be re-ordered, so keep in mind that yours may be in a different spot than mine. Note down the **Subscription ID** (boxed in red above) – you will need this later!

![OpenDiagnosticsTenantID](/assets/images/2016/11/OpenDiagnosticsTenantID.png)  
Next, click on the **Help** icon near the upper right corner and select **Show Diagnostics**. This will bring up some raw data about your subscription – and here is the easiest place I’ve found to locate your **Tenant ID**. Simply search for “**tenant**” and select the field shown above. Note this ID for later as well.

Now you’ll need to create a few objects in the Azure portal to consume from vRA. One of the great capabilities the new endpoint brings is the ability to create new, on demand objects per request – but to make things a little cleaner we will create just a few ahead of time. We’ll start with a Storage Account and a Resource Group.

![CreateStorageAccountAndResourceGroup](/assets/images/2016/11/CreateStorageAccountAndResourceGroup.png)  
Locate the **Storage Accounts** icon in the sidebar – again, keeping in mind that these icons can be reordered and you may have to poke around a bit to find it. Make sure the correct **Subscription** is selected and click **Add**.

You’ll be prompted with a sliding panel (Azure does love sliding panels) where you can fill in some important details about your Storage Account. This is basically a location where your files, VHDs, tables, databases, etc will be stored. Enter a **Name** for the Storage Account – you’ll need to make sure to follow the rules here. Only lowercase letters, must be globally unique, etc. You can choose to change any of the options presented here, but for the purposes of this guide we will leave the defaults and move on to the **Resource Group**. This is a logical grouping for deployed workloads and their related devices/items – and to keep things clean, we will specify a new one now. Note the name of this Resource Group for later. You’ll also need to choose a **Location** for the workloads – pick whatever is convenient or geographically reasonable for you. I chose **West US** – make a note of this as well! Click **Create**.

![CreateVirtualNetwork](/assets/images/2016/11/CreateVirtualNetwork.png)  
Now, let’s create a simple Virtual Network. Locate the **Virtual Network** icon on the panel to the left and click it. Ensure the correct **Subscription** is selected and click **Add**.

Again, you’ll be prompted with some basic configuration. Enter a unique name for your new Virtual Network and record it for later. You can choose to modify the other options as necessary, but for this guide we will leave the defaults. It is important, however, that you select to **Use Existing Resource Group** and specify the group you created in the last step. You’ll also want to select the same **Location** as you did before. Azure will not deploy VMs (or other objects) if the Location doesn’t match logically between the various components that the object will consume. Click **Create**.

![CreateAppRegistrationDetails](/assets/images/2016/11/CreateAppRegistrationDetails.png)  
Now you need to set up an Azure Active Directory application so that vRA can authenticate. Locate the **Active Directory** icon on the left hand side and click it. Next, click **App Registrations** and select **Add**. The most astute readers will notice that there are certain parts of some of my screenshots deleted – sorry about that! Had to remove sensitive information.

Enter a **Name** for your AD App – it can be anything you like, as long as it complies with the name validation. Leave **Web app/API** as the Application Type. The **Sign-on URL** is not really important for the purposes of this configuration – you can enter really anything you want here. In this example, we are using a dummy vRA 7 URL. Click **Create** (not pictured above, but you should have the hang of it by now!)

![SetupADApp](/assets/images/2016/11/SetupADApp.png)Sorry the above image is a little squashed. You can always click them for larger resolution!

Now you need to create a secret key to authenticate to the AD Application with. Click on the name of your new AD Application (in this case **vRADevTest**) at the left. Make sure you note down the **Application ID** for later. Then, select the **All Settings** button in the next pane. Choose **Keys** from the settings list.

![CreateAppKey](/assets/images/2016/11/CreateAppKey.png)  
Now, enter a **Description** for your new key and choose a **Duration**. Once you have entered those, click **Save** in the upper left of the blade – but note the warning! You will not ever get another chance to retrieve this value. Save the **Key Value** for later.

![ConfigureAPIPermissions](/assets/images/2016/11/ConfigureAPIPermissions.png)  
Now, look back to the left and select the **Required Permissions** option for the AD App. Click **Add** to create a new permission.

![SelectAzureSMAPI](/assets/images/2016/11/SelectAzureSMAPI.png)  
Click **Select an API** and choose the **Windows Azure Service Management API**, then click **Select**

![AssignSMAPIPermissions](/assets/images/2016/11/AssignSMAPIPermissions.png)  
Click the **Select Permissions** step at the left, then tick the box for **Access Azure Service Management as organization users (preview)** – then click **Select**. Once you do this, the **Done** button on the left will highlight. Click that as well.

There’s one final step in the Azure portal. Now that the AD Application has been created, you need to authorize it to connect to your Azure Subscription to deploy and manage VMs!

![BackToSubscriptionsView](/assets/images/2016/11/BackToSubscriptionsView.png)  
Click back on the **Subscriptions** icon (the Key) and select your new subscription. You may have to click on the text of the name to get the panel to slide over. Select the **Access control (IAM)** option to see the permissions to your subscription. Click **Add** at the top.

![SelectRole1](/assets/images/2016/11/SelectRole1.png)  
Click **Select a Role** and choose **Contributor** from the list

![SelectUsers](/assets/images/2016/11/SelectUsers.png)  
Click the **Add Users** option and search for the name of your new AD Application. When you see it in the list, tick the box and click **Select**, then **OK** in the first blade.

![RolesAssigned](/assets/images/2016/11/RolesAssigned.png)  
Repeat this process so that your new AD Application has the **Owner, Contributor**, and **Reader** roles. It should look like this when you’re done.

# Part 2 – Azure CLI and Other Setup

To do the next steps, you will need the Azure CLI tools installed. These are freely available from Microsoft for both Windows and Mac. I won’t go into great detail on how to download and install a client application here – but you can get all the info you need at <https://docs.microsoft.com/en-us/azure/xplat-cli-install>. For the purposes of this guide, please remember that I use a Mac.

![AzureLoginStep1](/assets/images/2016/11/AzureLoginStep1.png)  
Once you have the Azure CLI installed, you will need to authenticate to your new subscription. Open a **Terminal** window and enter **‘azure login’**. You will be given a URL and a shortcode to allow you to authenticate. Open the URL in your browser and follow these instructions to authenticate your subscription.

![EnterAuthCodeStep1](/assets/images/2016/11/EnterAuthCodeStep1.png)  
Enter your **Auth Code** and click **Continue**

![EnterAuthCodeStep2](/assets/images/2016/11/EnterAuthCodeStep2.png)  
Select and log in to your Azure account…

![AuthSuccessWeb](/assets/images/2016/11/AuthSuccessWeb.png)

![AuthSuccessCLI](/assets/images/2016/11/AuthSuccessCLI.png)  
And if all went well, you now have a success message in both your browser and the CLI. Nice work!

![AzureAccountSet](/assets/images/2016/11/AzureAccountSet.png)  
If you have multiple subscriptions, as I do, you’ll need to ensure that the correct one is selected. You can do that with the **‘azure account set &lt;subscription-name&gt;’** command. Be sure to escape any spaces!

![RegisterComputeProvider](/assets/images/2016/11/RegisterComputeProvider.png)  
Before you go any further, you need to register the **Microsoft.Compute** provider to your new Azure subscription. This only needs to be done once, which means it’s easy to forget! The command is just **‘azure provider register microsoft.compute’** – and it has timed out the first time in 100% of my test cases. So I left that Big Scary Error in the screenshot for you – don’t worry, just run it a second time and it will complete.

![AzureVMImageList](/assets/images/2016/11/AzureVMImageList.png)  
Now, let’s use the Azure CLI to retrieve an example VM image name. These will be used in the vRA Blueprints to specify which type of VM you’d like to deploy. To do this, you’ll use the **‘azure vm image list’** command. In my example, the full command was **‘azure vm image list –location “West US” –publisher canonical –offer ubuntuserver –sku 16.04.0-LTS’** – this limits the list of displayed options to only those present in my West US location, published by Canonical, of type Ubuntu Server, containing the string 16.04.0-LTS in their name.

Choose one of these images and record the URN provided for it. As an example: **canonical:ubuntuserver:16.04.0-LTS:16.04.201611150**

So, to recap – you have set up your Azure subscription and should have the following list of items recorded:

- Subscription ID
- Tenant ID
- Storage Account Name
- Resource Group Name
- Location
- Virtual Network Name
- Client Application ID
- Client Application Secret Key
- VM Image URN

Now, let’s move on to actually configuring vRA!

# Part 3 – Configuring vRA

This section assumes that you have already deployed vRA with the default tenant, have created your basic users and permissions, and have at least one business group ready. This basic level of vRA setup is outside the scope of this guide.

![AdministrationTab](/assets/images/2016/11/AdministrationTab.png)  
Once you are logged in as an Infrastructure/IaaS administrator, proceed to the **Administration** tab and select **vRO Configuration** from the menu at the left (not pictured.) Then, choose **Endpoints** and select **New** to set up a new endpoint.

The Azure endpoint is not configured from the traditional Infrastructure tab location because it is not managed by the IaaS engine of vRA – it is presented via vRO and XaaS.

![SelectAzureType](/assets/images/2016/11/SelectAzureType.png)  
Select the **Azure** plug-in type and click **Next**

![AzureEndpointName](/assets/images/2016/11/AzureEndpointName.png)  
Enter a **Name** for your Endpoint and click **Next** again

![EnterAzureSubscriptionDetails](/assets/images/2016/11/EnterAzureSubscriptionDetails.png)  
Now the fun part! Remember all that info you copied down earlier? Time to use it! Fill in the **Connection Settings** with the details from the subscription configuration you did earlier. You won’t need to change the Azure Services URI or the Login URL, and the Proxy Host/Port are optional unless you know you need one.

Click **Finish** and the connection should be created!

![FabricGroups](/assets/images/2016/11/FabricGroups.png)  
Next, navigate to the **Infrastructure** tab and select **Endpoints** (not pictured,) followed by **Fabric Groups**. In this example I don’t yet have a Fabric Group, so I will create one by clicking **New**.

![NewFabricGroup](/assets/images/2016/11/NewFabricGroup.png)  
Remember a little while ago that I mentioned the Azure Endpoint is not managed by IaaS – so you won’t need to select any Compute Resources here. You just need to ensure that your user account is a Fabric Administrator to continue the rest of the configuration. If you already have this right, you may skip this step.

Now, refresh the vRA UI so that your new Fabric Administrator permissions take effect.

![CreateNewReservation](/assets/images/2016/11/CreateNewReservation.png)  
Once that’s done, navigate to the **Infrastructure** tab and the **Reservations** menu. Select the **New** button and choose a reservation of type **Azure**.

![NewReservationGeneral](/assets/images/2016/11/NewReservationGeneral.png)  
Fill in a **Name** and select a **Business Group** and **Priority** for the reservation, then click on the **Resources** tab

![NewReservationResources](/assets/images/2016/11/NewReservationResources.png)  
Enter your **Subscription ID** – be sure this is the same subscription ID that was specified in your Endpoint configuration. Requiring this field allows the mapping of many reservations to many endpoints/subscriptions.

Then, add the **Resource Group** and **Storage Account** which you created earlier. This is not required, but it does save some steps when creating the Blueprint later.

Click on the **Network** tab.

![NewReservationNetwork](/assets/images/2016/11/NewReservationNetwork.png)  
Enter the name of the **Virtual Network** you created earlier. Also note that you can set up Load Balancers and Security Groups here. Click **OK** to save the reservation.

![CreateMachinePrefix](/assets/images/2016/11/CreateMachinePrefix.png)  
Next, you’ll need a Machine Naming Prefix. Click on the **&lt;Infrastructure** menu option (not pictured) and then select **Administration** (also not pictured) and finally **Machine Prefixes**. Enter a string, number of digits and next number that works for you – I used **AzureDev-\###** starting with the number 0. Be sure to click the **Green Check** to save the prefix.

This prefix will be applied to any objects provisioned in a request – whether they are VMs, NICs, storage disks, etc. This helps the grouped objects to be easily located in an often busy Azure environment.

![BusinessGroups](/assets/images/2016/11/BusinessGroups.png)  
Now, click the **Administration** tab, followed by the **Users and Groups** menu (not pictured) and the **Business Groups** option. Select the business group that you plan to deploy with – in this example I have three to choose from and will be using **Development**.

![ChooseBGMachinePrefix](/assets/images/2016/11/ChooseBGMachinePrefix.png)  
Select your new **Default Machine Prefix** and click **Finish.**

# Part 4 – Building a Blueprint

Now that the groundwork is laid, let’s build, entitle, and deploy a simple Azure blueprint!

![DesignTab](/assets/images/2016/11/DesignTab.png)  
Head over to the **Design** tab and make sure the **Blueprints** menu is open. It should be the default. Click **New** to begin designing a blueprint.

![BlueprintProperties](/assets/images/2016/11/BlueprintProperties.png)  
Give your blueprint a **Name** and click **OK**

![BlueprintCanvas](/assets/images/2016/11/BlueprintCanvas.png)  
Ensure the **Machine Types** category is selected and drag an **Azure Machine** to the canvas. Increase the **Maximum Instances** to 3 – this will make your Azure machine scalable! Click the **Build Information** tab to proceed.

![BuildInformation](/assets/images/2016/11/BuildInformation.png)  
Now you can begin filling out details about the machine itself. Select a **Location** – or one will be chosen for you from the reservation. You can also choose a **Naming Prefix** or allow the one you set up a moment ago to be the default. You can choose to select a **Stock VM Image** and paste the URN you retrieved from the Azure CLI, or you can specify a custom, user created one. Here you can also specify the **Authentication** options as well as the **Instance Size** configuration. If any of these options are left blank, they will be required at request time.

Note that when editing a field, you will see an editing dialog appear on the right of the blueprint form. This is to allow you additional flexibility in the configuration; please be sure to click ‘**Apply**‘ to save any changes. Also note that there are many helpful tooltips throughout the blueprint designer to help you along.

Click the **Machine Resources** tab to move on.

![MachineResources](/assets/images/2016/11/MachineResources.png)  
Here you can specify your **Resource Group** and **Availability Set** – and as before, you can fill in the one you created manually or allow vRA to create new ones for you. Remember to fill in the information on the right hand side and click **Apply** to save the values!

Click **Storage** to move to the next step.

![MachineStorage](/assets/images/2016/11/MachineStorage.png)  
The Storage tab allows you to specify details about your machine’s storage capabilities. You can specify the **Storage Account** here if you choose – or it can be inherited from the Reservation. If you explore this tab, you’ll see you can also create additional data disks as well as enable/disable the boot diagnostics functionality. For this example we will just create a simple OS disk configuration.

Now, click on the **Network** tab.

![MachineNetwork](/assets/images/2016/11/MachineNetwork.png)  
This is where you can configure advanced networking capabilities. In this example, you won’t fill anything in and we will instead allow the Azure reservation to apply the networking properties you specified earlier. Click **Finish** to save your blueprint.

![PublishBlueprint](/assets/images/2016/11/PublishBlueprint.png)  
Select your new blueprint and **Publish** it.

Now you must **entitle your new blueprint**. Because the steps to complete this operation can be highly dependent on the environment you’re doing it in, we will skip the details on how to create an entitlement and add this blueprint to it. Let’s move right ahead to provisioning the VM!

# Part 5 – Deploying a Blueprint

I hope you’re glad you stuck with me this far! To recap, so far you have:

- Created and configured your Azure subscription for vRA
- Collected up a list of all the important pieces of data needed to provision to Azure
- Configured vRA to deploy to Azure
- Built your first Azure blueprint

There’s just one thing left to do….

![vRACatalog](/assets/images/2016/11/vRACatalog.png)  
Navigate to the **Catalog** tab, locate your new Azure blueprint and click **Request**.

![RequestDetails](/assets/images/2016/11/RequestDetails.png)  
Feel free to click around the request details – you’ll see that anything you specified in the blueprint itself is now a locked field. Other fields are still open and available for editing. You can create some seriously flexible requests by locking and unlocking only specific fields – the form is highly customizable.

When you’re done exploring, click **Submit**!

![vRARequestStatusSuccessful](/assets/images/2016/11/vRARequestStatusSuccessful.png)  
You can monitor the status of the request as you normally would, in the **Requests** tab.

![vRADeployments](/assets/images/2016/11/vRADeployments.png)  
After the provisioning completes, you’ll be able to see your new Azure VM in vRA…

![AzureProvisioned](/assets/images/2016/11/AzureProvisioned.png)  
…as well as in the Azure portal itself! You can see that the Naming Prefix was applied to both the VM and the vNIC that was created to support it.

![SouthernTierPumking](/assets/images/2016/11/SouthernTierPumking-768x1024.jpg)  
This post was brought to you courtesy of Southern Tier Brewing’s Pumking – possibly the only good pumpkin beer ever. It hits all the natural squash and spice notes without ever feeling extracted, artificial, or overwhelming. And it gets bonus points for being from my home town. Yum!

I hope this guide has been helpful and that you’re as excited as I am about this great new addition to vRealize Automation’s repertoire. Please leave any feedback in the comments, and don’t forget to follow me on Twitter!

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=Using+the+new+Microsoft+Azure+Endpoint+in+vRealize+Automation+7.2)</div></div>