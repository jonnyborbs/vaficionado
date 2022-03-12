---
id: 197
title: 'Deploying vRealize Automation Workloads from Apple Watch'
date: '2015-04-25T14:57:04-07:00'
author: jon
layout: post
image: /assets/images/cropped-va-header.jpg
guid: 'http://www.vaficionado.com/?p=197'
permalink: /2015/04/deploying-vrealize-automation-workloads-from-apple-watch/
categories:
    - Integration
    - Tech
tags:
    - apple
    - integration
    - vra
    - workflow
---

Like many people (although not as many as would have liked, I suppose,) I got my shiny new Apple Watch yesterday.

Once it was set up and on my wrist, the first thing I thought of was naturally “How can I use this with vRA?”

Naturally.

It didn’t take long to figure this one out. I don’t know how valuable it will be in the real world, but you have to admit – it sure is cool, particularly for showing the amazing flexibility of vRealize Automation.

Basically, we started with this. A simple button on the Apple Watch that starts a [Workflow](https://itunes.apple.com/us/app/workflow-powerful-automation/id915249334?mt=8) which is then handed off to my iPhone.

[![Apple_Watch_Workflow_Red](/vaficionado/assets/images/2015/04/Apple_Watch_Workflow_Red-1024x768.jpg)](/vaficionado/assets/images/2015/04/Apple_Watch_Workflow_Red.jpg)

[![Apple_Watch_Workflow_Screenshot](/vaficionado/assets/images/2015/04/Apple_Watch_Workflow_Screenshot.png)](/vaficionado/assets/images/2015/04/Apple_Watch_Workflow_Screenshot.png)

[![Workflow_Running_on_Apple_Watch](/vaficionado/assets/images/2015/04/Workflow_Running_on_Apple_Watch.png)](/vaficionado/assets/images/2015/04/Workflow_Running_on_Apple_Watch.png)  
(Edit: I literally just this second learned how to screenshot on the Watch, so I have included both images. Just because)

The iPhone then connects via SSH to a Linux host running [CloudClient](https://developercenter.vmware.com/tool/cloudclient/3.2.0) and runs a deployment script I wrote

[![Workflow_Details_on_iPhone](/vaficionado/assets/images/2015/04/Workflow_Details_on_iPhone-576x1024.png)](/vaficionado/assets/images/2015/04/Workflow_Details_on_iPhone.png)

The script is quite basic and is as follows:

```
<pre class="brush: bash; title: ; notranslate" title="">
#!/bin/sh
#
# Autodeploy a vRA item CentOS-vCO Test
echo "---------------------------------------------" >> vRA-Deploy.log
echo "Deployment started at" `date` >> vRA-Deploy.log
/root/cloudclient-3.2.0-2594179/bin/cloudclient.sh vra catalog request submit --id '"CentOS-vCO Test"' --groupid '"Ops Managers"' --reason '"Deployed via Apple Watch"' --export /tmp/request.txt >> vRA-Deploy.log
echo "Deployment handed off to vRA at" `date` >> vRA-Deploy.log
echo "---------------------------------------------" >> vRA-Deploy.log
```

This uses the auto-login configuration of CloudClient to connect to my vRealize Automation instance and deploy a simple CentOS blueprint from my catalog.

[![Cloud_Client_Log](/vaficionado/assets/images/2015/04/Cloud_Client_Log.png)](/vaficionado/assets/images/2015/04/Cloud_Client_Log.png)  
(Click image for a larger version)

The details of the deployment come up in the vRA-Deploy.log file…

[![vRealize_Automation_Apple_Watch_Request_Successful](/vaficionado/assets/images/2015/04/vRealize_Automation_Apple_Watch_Request_Successful.png)](/vaficionado/assets/images/2015/04/vRealize_Automation_Apple_Watch_Request_Successful.png)

[![Deployed_Workload_in_vSphere](/vaficionado/assets/images/2015/04/Deployed_Workload_in_vSphere.png)](/vaficionado/assets/images/2015/04/Deployed_Workload_in_vSphere.png)

And voila! I’ve just provisioned a VM from my watch. The future is now, people.

Edit: 4/26/2015 – I just realized that the step of handing off the workflow from the Watch to the iPhone was unnecessary. The Watch can execute the SSH commands directly without the added handoff. I’ve updated the screenshots and the text accordingly. Cool!