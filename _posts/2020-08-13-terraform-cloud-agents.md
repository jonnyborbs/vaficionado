---
id: 579
title: 'Terraform Cloud Agents'
date: '2020-08-13T08:58:41-07:00'
author: jon
layout: post
image: /assets/images/cropped-va-header.jpg
guid: 'https://www.vaficionado.com/?p=579'
permalink: /2020/08/terraform-cloud-agents/
categories:
    - HashiCorp
    - Stories
    - Tech
    - Terraform
tags:
    - agents
    - cloud
    - hashicorp
    - on-prem
    - on-premises
    - 'self hosted'
    - terraform
    - 'terraform cloud'
---

***Updated November 16, 2020: Terraform Cloud Agents now supports user-configured multipool!***

Well hello there, readers, if any still remain. I’ve been gone a long time, but I’ve got some cool new stuff to show today – let’s talk about Terraform Cloud Agents.

First, some housekeeping. Did I mention that I’ve moved from VMware to HashiCorp? I didn’t, you say? Well, that seems important to note for context in this post. I joined the HashiCorp Terraform Cloud team in June of 2019 (time flies) and have been here ever since – loving every minute of it!

Now, on to the fun stuff. In mid-August, Terraform Cloud made its biggest announcement since we launched publicly in January – the Terraform Cloud Business tier. This new tier of service provides a whole host of additional business and enterprise focused features in our already awesome SaaS platform, and you can read all about it [here](https://www.hashicorp.com/blog/announcing-hashicorp-terraform-cloud-business/) if you like.

Of these new features, the one that I’ve spent the last 3 months working on is our new [Terraform Cloud Agents](https://www.terraform.io/docs/cloud/agents/index.html) capability. The goal of this project was to provide a way for our customers with significant investments in on-premises infrastructure (think vSphere, Nutanix, F5, OpenStack, etc) to take advantage of our SaaS management interface without having to expose all of those infrastructure components to the Internet.

This would be super useful for a whole host of scenarios – for example, a datacenter with a ton of bare metal or vSphere infrastructure that needs orchestration. Or, perhaps ROBO offices where periodic infrastructure management is needed – often enough to want to use an IaC approach through Terraform, but not often enough to justify a full installation of Terraform Enterprise. Or even just an isolated AWS VPC that doesn’t expose public access for data protection and security purposes. There are a ton of reasons why a network segment might be isolated, and Terraform Cloud Agents are meant to help ensure those segments may be lonely, but not forgotten.

So, how do you use a Terraform Cloud Agent? Well, let’s step through it. It’s crazy simple to do.

First, you’ll need to identify where you want to run the agent. We provide it both as a Docker container in the [public registry](https://hub.docker.com/r/hashicorp/tfc-agent) or as a standalone signed binary from [HashiCorp directly](https://releases.hashicorp.com/tfc-agent/). You can use these to run the agent in a number of ways – on a bare VM, in base Docker, in Kubernetes, in HashiCorp Nomad… you get the idea! If you’d like to use K8s to run the agent, I suggest checking out this great [Terraform module](https://registry.terraform.io/modules/redeux/terraform-cloud-agent/kubernetes/latest) published by my coworker Phil that makes it super easy.

For this example, we’ll just run a Docker container in a local Docker machine for simplicity’s sake. We won’t go into all the process supervisor options here, though we do recommend pairing the agent with a supervisor for reliability purposes.

Before we get started running the agent, we need to do some setup in our Terraform Cloud for Business environment. We’ll assume you’ve got an account on Terraform Cloud already (if not, you can get one [for free](https://app.terraform.io/signup/account), though it won’t have access to these agent capabilities unless you have the paid Business tier active). Navigate to the **Settings** &gt; **Agents** page, and click **New agent pool**.

An agent pool is a logical grouping of agents that work together to handle requests from your workspace(s). You can group your agents however you like, but one example could be to create a single agent pool for each on-premises data center you want to connect to. For example, **DC1\_NewYork** and **DC2\_HongKong**. We’ll get into how you divide work up in just a bit.

![create agent pool](/vaficionado/assets/images/2020/11/01_create_agent_pool-1024x493.png)
This will prompt you with the agent pool creation dialog, where you can enter an **agent pool name**. Do so, then click **Continue**.

![set agent pool description](/vaficionado/assets/images/2020/11/02_agent_pool_description.png)
Once you’ve done this, you’ll be prompted to create a new agent pool token. These tokens are used for the agents within a pool to authenticate to Terraform Cloud, and can be used by any agent **within** a pool but are not shared **across** pools. Enter a token **description** and click **Create token**.

![create a token description](/vaficionado/assets/images/2020/11/03_token_description.png)
Terraform Cloud will automatically generate a series of environment variables and commands you can use in your Docker environment as well. We’ll use these in a minute; keep this dialog visible on your browser and flip to a terminal environment. Remember that you never get to see this token again, so don’t close this or you’ll have to start over.

![token details](/vaficionado/assets/images/2020/11/04_token_details-852x1024.png)
Now that you’re in your terminal, make sure you have Docker installed. I’m personally on a Mac using Homebrew, so in this case I need to ensure I have `docker` and `docker-machine` installed. We’ll start by running `docker-machine start default` to bring up a basic machine to execute containers.

We’ll follow that quickly with `eval "$(docker-machine env default)"` to ensure that my interactive shell can interact with that running machine in the background.

![docker-machine start commands](/vaficionado/assets/images/2020/08/docker-machine-start-1.png)
Now, I’m going to use an `env.list` file to contain all my environment variables for this example. There are lots of different ways to handle environment variables for Docker, and you can pick whatever fits your architecture. You can export variables directly into the environment, use a file as I will here, or put them right on the command line (I don’t recommend this though, as your token will end up visible in process lists) if you like.

Here’s what `env.list` looks like on my machine. You can see I’ve exported the one required variable, `TFC_AGENT_TOKEN` and set it to the token copied from Terraform Cloud above. I’ve also set the optional `TFC_AGENT_NAME` variable so there will be a friendly name displayed in my agents list later on. The `TFC_ADDRESS` variable isn’t required, and this URL is actually the default anyway. And finally, the `TFC_AGENT_LOG_LEVEL` is set to `DEBUG`. This lets you see a bunch more information about the agent’s work, and is helpful but verbose. Once you’ve set at least your token and name, save and exit your editor.

![env.list file contents](/vaficionado/assets/images/2020/11/06_env_dot_list.png)
Here comes the fun part! Now we’ll run `docker run --env-file env.list hashicorp/tfc-agent:latest` and Docker will go do its magic. All the supporting slices will be pulled into your default running Docker machine, the agent will start up, self-check for upgrades, and register itself with Terraform Cloud.

*Note: if you’re using Docker Desktop 2.4 or greater on a Mac, you may need to run the container with the `-it` flag, or it may not exit properly. This appears to be an issue in Docker, but we’re still trying to find a workaround for it.*

![agent running at command line](/vaficionado/assets/images/2020/11/07_agent_running-1024x520.png)
Now flip back to your browser. You can close the **Create an agent pool** dialog if you haven’t already by clicking **Finish**. You’ll now see the list of agent pools and their agents registered to your environment.

![running agents within a pool in terraform cloud](/vaficionado/assets/images/2020/11/08_agents_and_pools_list.png)
Wow, is it really that easy? It sure is. Your new agent is now up and running, registered with your organization and waiting for work. You can see I have several agents listed here, some of which are in the **Exited** state. These are containers that I started up, used, and quit gracefully. They’ll automatically purge from the UI after a few hours of inactivity.

You’ll also see the note **1 out of 10 Purchased Agents** in the table header. The number of purchased agents is determined by your Terraform Cloud Business subscription, and the number of agents that count against it depends on those agents status. Exited or otherwise offline agents don’t count against your total – only agents in the **Busy**, **Idle** or **Unknown** state will count against your entitlement here. **Unknown** agents may or may not come back online; if they do, they’ll return to an **Idle** state. If they don’t, they’ll expire out as permanently offline.

**Optional**: You can also create additional agent pools and register agents against them. You can see here that I’ve created a second pool for **DC2\_HongKong** and there are agents waiting for work there as well.

![multiple pools displayed within terraform cloud](/vaficionado/assets/images/2020/11/09_agents_multiple_pools.png)
A quick note on agent architecture: it’s designed to require no inbound Internet access to your environment. It continuously polls the Terraform Cloud service using outbound TCP/443 calls to ask for work items, then retrieves them if available. Super cool.

So now that the agent pool is created and the agents are registered and ready, all that’s left is to tell your Terraform Cloud workspaces to use them. You’ll want to click on **Workspaces** at the top of the screen, then select the **name** of the workspace you want to configure. From there, choose **Settings &gt; General** and change your execution mode to **Agent**. Once this option is selected, you can also pick which **Agent pool** should be used for this workspace’s runs.

If your organization isn’t entitled to agents as a feature or if you haven’t configured any agents yet, you won’t be able to select this setting.

![configuring a terraform cloud workspace to use the agent execution mode](/vaficionado/assets/images/2020/11/10_workspace_settings-1024x642.png)
Click **Save Settings** at the bottom of the screen (not pictured) and you’re all set to go!

You can now queue a plan in this workspace via whatever mechanism you choose; the Queue Plan button in the UI, a VCS file-based commit, etc. For this example, we’ll use the **Queue Plan** button. Click it, fill in a reason if you like, then click **Queue plan** again.

![queueing a new plan](/vaficionado/assets/images/2020/08/queue-plan.png)
This will take you to the Run details view for this execution. You can see the plan has completed, Cost Estimation has run, any applicable Sentinel policies were evaluated, and the run is now just waiting for approval to actually execute.

You can also see exactly which agent pool the run executed in, along with the specific agent that ran it.

![run details](/vaficionado/assets/images/2020/11/11_run_details-1024x932.png)
But perhaps even more interesting is what’s going on over in that terminal window. Here you can see that the agent has received the run, determined which version of Terraform is required to carry it out and downloaded the binary release to do so.

Side note: in this case the Terraform version is `0.12.7` which I just realized as I was taking these images, and definitely need to update – did you know `0.14` [just released last ](https://www.hashicorp.com/blog/announcing-hashicorp-terraform-0-13/)[month](https://www.hashicorp.com/blog/announcing-hashicorp-terraform-0-14-beta) as well?

Once the Terraform binary is available, the agent will run a `terraform init` to download all required providers, etc, followed by the actually requested `terraform plan` – the logs and output of which are streamed back to Terraform Cloud for you to view in the friendly UI.

![tfc-agent job details output](/vaficionado/assets/images/2020/08/agent-run-output-1024x206.png)
Now if you wanted to, you could go back over to Terraform Cloud, approve the run to actually apply, and the agent will finish the job for you! We won’t explore that here, since I’m sure y’all know what a Terraform apply looks like.

And that’s pretty much it! You’ve created an agent pool, an agent pool token, deployed an agent, configured your workspace to use the pool, and run a plan/apply cycle against it.

A few important notes:

On pool and token management: you can manage your agent pools and tokens at any time by clicking the name of your agent pool in the agents list view. Revoking a token will cause any agents using it to exit, since they’ll no longer be able to communicate with Terraform Cloud. You can create a new token and start new agents with it at any time. You can rename agent pools at any time, but you cannot delete an agent pool that still has workspaces configured to use it.

On agent flexibility: a really cool use case for the binary agent here is to roll your own execution environment, complete with whatever sideloaded additional tools you might need. Want to leverage the AWS or Google Cloud CLI in your Terraform configs? Include them on a VM alongside the Terraform Cloud Agent binary and you’re good to go!

To learn more about Terraform Cloud Agents, you can visit the documentation page at <https://www.terraform.io/docs/cloud/workspaces/agent.html> or contact HashiCorp to [request more information](https://www.hashicorp.com/contact-sales/?utm_source=TFC4B).

And, of course, it wouldn’t be right to end without some kind of adult beverage – so today’s post is brought to you by [Urban Roots Brewing](https://www.urbanrootsbrewing.com/)‘s Floofster – an adorable German-style Hefeweizen with the name I can’t stop saying. Prost!

![urban roots brewing's floofster](/vaficionado/assets/images/2020/08/urban-roots-floofster-768x1024.jpeg)

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=Terraform+Cloud+Agents)</div></div>