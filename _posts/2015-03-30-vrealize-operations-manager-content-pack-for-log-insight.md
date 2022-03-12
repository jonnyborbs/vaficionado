---
id: 166
title: 'vRealize Operations Manager Content Pack for Log Insight'
date: '2015-03-30T16:36:42-07:00'
author: jon
layout: post
guid: 'http://www.vaficionado.com/?p=166'
permalink: /2015/03/vrealize-operations-manager-content-pack-for-log-insight/
categories:
    - Integration
    - Tech
tags:
    - 'log insight'
    - 'operations management'
    - vrops
    - Wine
---

Get ready, ops-heads… another exciting announcement from the VMware team. There’s now a formal content pack for Log Insight that will allow the import and visualization of the logs from vRealize Operations Manager 6.x.

As an added bonus, if you are running vROps 6.0.1 or later – the Log Insight agent is already pre-installed on your appliance – all you have to do is configure it! If you’re on an earlier version, you can still manually install and configure the agent. Instructions for doing that can be found [here](https://solutionexchange.vmware.com/store/products/vrealize-operations-manager-content-pack-for-log-insight/files/21977).

Given the incredible volume and depth of the data that’s being imported and analyzed by this content pack, the configuration file is pretty complex. The official installation notes are in a PDF format that was a little difficult to copy and paste all the elements from, so I’ve created a properly formatted file and attached it below.

There are a few tags you will need to change to make this work – I’ve included the tag names as well as the current find-and-replace value below so you can easily tailor the file to your needs. When you’re done, just save it as **/var/lib/loginsight-agent/liagent.ini** on each node and restart the Log Insight agent (by running **/etc/init.d/liagentd restart**)

Here’s a helpful screenshot of where you can find several of these parameters for your cluster nodes. Keep in mind that if you have a multi-tier deployment, you will need to customize the below config file for each node.

[![vRealize_Operations_Manager_Cluster_Administration](/vaficionado/assets/images/2015/03/vRealize_Operations_Manager_Cluster_Administration.png)](/vaficionado/assets/images/2015/03/vRealize_Operations_Manager_Cluster_Administration.png)  
(Click the image for a larger version)

Here are the paramters that need to be changed:

- **hostname** – this is the IP or FQDN of your Log Insight server. Note that this only needs to be changed in the **\[server\]** section at the top of the file, and not throughout the entire file. Below, it is set to <span style="color: #ff0000;">**&lt;YOUR LOGINSIGHT HOSTNAME HERE&gt;**</span>
- **vmw\_vr\_ops\_clustername** – this is the \*name\* of your vRealize Operations cluster. This can be anything you like here and can be used to distinguish one cluster from another if you have multiples. Below, it is <span style="color: #ff0000;">**&lt;YOUR CLUSTER NAME HERE&gt;**</span>
- **vmw\_vr\_ops\_clusterrole** – this is the role that the node you are installing this file on fills. The choices are “**Master**“, “**Replica**“, “**Data**“, or “**Remote** **Collector**” – on a single-node installation, use Master. Below, it is set to <span style="color: #ff0000;">**Master<span style="color: #000000;">. </span>**<span style="color: #000000;">This value can be found on the **Administration &gt; Cluster Management** page in the vRealize Operations Manager UI (see above image)</span></span>
- **vmw\_vr\_ops\_hostname** – this is the hostname of your vRealize Operations Manager cluster. This hostname can also be found <span style="color: #ff0000;"><span style="color: #000000;">on the **Administration &gt; Cluster Management** page in the vRealize Operations Manager UI (see above image). Below, it is set to **<span style="color: #ff0000;">&lt;YOUR VROPS HOSTNAME HERE&gt;</span>**</span></span>
- **vmw\_vr\_ops\_nodename** – this is the node name of the node you are installing this file on. This name can be found <span style="color: #ff0000;"><span style="color: #000000;">on the **Administration &gt; Cluster Management** page in the vRealize Operations Manager UI (see above image). Below, it is set to <span style="color: #ff0000;">**&lt;YOUR NODE NAME HERE&gt;**</span></span></span>

And here’s the config file itself:

```
<pre class="brush: bash; title: ; notranslate" title="">
; Client-side configuration of VMware Log Insight Agent
; See liagent-effective.ini for the actual configuration used by VMware Log Insight Agent

[server]
; Log Insight server hostname or ip address
; If omitted the default value is LOGINSIGHT
hostname=<YOUR LOGINSIGHT HOSTNAME HERE>

; Set protocol to use:
; cfapi - Log Insight REST API
; syslog - Syslog protocol
; If omitted the default value is cfapi
;
;proto=cfapi

; Log Insight server port to connect to. If omitted the default value is:
; for syslog: 512
; for cfapi without ssl: 9000
; for cfapi with ssl: 9543
;port=9000

;ssl - enable/disable SSL. Applies to cfapi protocol only.
; Possible values are yes or no. If omitted the default value is no.
;ssl=no

; Time in minutes to force reconnection to the server
; If omitted the default value is 30
;reconnect=30

[storage]
;max_disk_buffer - max disk usage limit (data + logs) in MB:
; 100 - 2000 MB, default 200
;max_disk_buffer=200

[logging]
;debug_level - the level of debug messages to enable:
;   0 - no debug messages
;   1 - trace essential debug messages
;   2 - verbose debug messages (will have negative impact on performace)
;debug_level=0

[filelog|messages]
directory=/var/log
include=messages;messages.?

[filelog|syslog]
directory=/var/log
include=syslog;syslog.?

[filelog|ANALYTICS-analytics]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"ANALYTICS","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = analytics*.log*
exclude_fields=hostname

[filelog|COLLECTOR-collector]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"COLLECTOR","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = collector.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|COLLECTOR-collector_wrapper]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"COLLECTOR","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = collector-wrapper.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\.\d{3}

[filelog|COLLECTOR-collector_gc]
directory = /data/vcops/log
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"COLLECTOR","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
include = collector-gc*.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\w]\d{2}:\d{2}:\d{2}\.\d{3}

[filelog|WEB-web]
directory = /data/vcops/log
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"WEB","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
include = web*.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|GEMFIRE-gemfire]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"GEMFIRE","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = gemfire*.log*
exclude_fields=hostname

[filelog|VIEW_BRIDGE-view_bridge]
tags = {"vmw_vr_ops_appname":"vROps","vmw_vr_ops_logtype":"VIEW_BRIDGE","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = view-bridge*.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|VCOPS_BRIDGE-vcops_bridge]
tags = {"vmw_vr_ops_appname":"vROps","vmw_vr_ops_logtype":"VCOPS_BRIDGE","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = vcops-bridge*.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|SUITEAPI-api]
directory = /data/vcops/log
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"SUITEAPI","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
include = api.log*;http_api.log*;profiling_api.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|SUITEAPI-suite_api]
directory = /data/vcops/log/suite-api
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"SUITEAPI","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
include = *.log*
exclude_fields=hostname
event_marker=^\d{2}-\w{3}-\d{4}[\s]\d{2}:\d{2}:\d{2}\.\d{3}

[filelog|ADMIN_UI-admin_ui]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"ADMIN_UI","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log/casa
include = *.log*;*_log*
exclude_fields=hostname

[filelog|CALL_STACK-call_stack]
tags = {"vmw_vr_ops_appname":"vROps","vmw_vr_ops_logtype":"CALL_STACK", "vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>","vmw_vr_ops_clusterrole":"Master", "vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>","vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log/callstack
include = analytics*.txt;collector*.txt
exclude_fields=hostname

[filelog|TOMCAT_WEBAPP-tomcat_webapp]
tags = {"vmw_vr_ops_appname":"vROps","vmw_vr_ops_logtype":"TOMCAT_WEBAPP","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log/product-ui
include = *.log*;*_log*
exclude_fields=hostname

[filelog|OTHER-other1]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"OTHER","vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master","vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = aim*.log*;calltracer*.log*;casa.audit*.log*;distributed*.log*;hafailover*.log;his*.log*;installer*.log*;locktrace*.log*;opsapi*.log*;query-service-timer*.log*;queryprofile*.log*;vcopsConfigureRoles*.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3} 

[filelog|OTHER-other2]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"OTHER", "vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master", "vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = env-checker.log*
exclude_fields=hostname
event_marker=^\d{2}\D{1}\d{2}\D{1}\d{4}\s\d{2}:\d{2}:\d{2}

[filelog|OTHER-other3]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"OTHER", "vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master", "vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log
include = gfsh*.log*;HTTPPostAdapter*.log*;meta-gemfire*.log*;migration*.log*
exclude_fields=hostname

[filelog|OTHER-watchdog]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"OTHER", "vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master", "vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log/vcops-watchdog
include = vcops-watchdog.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|ADAPTER-vmwareadapter]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"ADAPTER", "vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master", "vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log/adapters/VMwareAdapter
include = *.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|ADAPTER-vcopsadapter]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"ADAPTER", "vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master", "vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log/adapters/VCOpsAdapter
include = *.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}

[filelog|ADAPTER-openapiadapter]
tags = {"vmw_vr_ops_appname":"vROps", "vmw_vr_ops_logtype":"ADAPTER", "vmw_vr_ops_clustername":"<YOUR CLUSTER NAME HERE>", "vmw_vr_ops_clusterrole":"Master", "vmw_vr_ops_nodename":"<YOUR NODE NAME HERE>", "vmw_vr_ops_hostname":"<YOUR VROPS HOSTNAME HERE>"}
directory = /data/vcops/log/adapters/OpenAPIAdapter
include = *.log*
exclude_fields=hostname
event_marker=^\d{4}-\d{2}-\d{2}[\s]\d{2}:\d{2}:\d{2}\,\d{3}
```

See what I mean about complex? And speaking of which… (come on, you had to know this was coming)

Today’s message has been brought to you by Talisman’s 2010 Adastra Vineyard Pinot Noir. The amazing folks at Talisman produce incredible small batch Pinot Noir from several vineyards across northern California. Their philosophy is to focus on the terroir of their fruit, so they produce every wine under precisely the same conditions – from crushing to aging to the oak in the barrels, everything is identical but the fruit itself. This allows the complexities afforded by each individual vineyard to really shine through. This is one of my favorites, with vanilla, dark fruit, spices and a nose that almost makes you forget to take a sip.

[![Talisman_2010_Adastra](/vaficionado/assets/images/2015/03/Talisman_2010_Adastra-768x1024.jpg)](/vaficionado/assets/images/2015/03/Talisman_2010_Adastra.jpg)

Now. Once you’ve configured and restarted your Log Insight agents on the vRealize Operations Manager cluster nodes, all you have to do is import the Content Pack into Log Insight. It is available for direct download from the VMware Solution Exchange [here](https://solutionexchange.vmware.com/store/products/vrealize-operations-manager-content-pack-for-log-insight#.VRnJJUaM67Q), or you can install it directly from your Log Insight console by accessing the Content Pack Marketplace and selecting the **VMware – vR Ops 6.x** Content Pack.

[![Content_Pack_Marketplace](/vaficionado/assets/images/2015/03/Content_Pack_Marketplace.png)](/vaficionado/assets/images/2015/03/Content_Pack_Marketplace.png)

When that’s complete, you’re ready to start leveraging the 12 Dashboard Groups, 81 Dashboard Widgets, 18 Queries, 8 Alerts and 31 Extracted Fields that this content pack exposes to you. Check it out!

[![Log_Insight_vRealize_Operations_Dashboards](/vaficionado/assets/images/2015/03/Log_Insight_vRealize_Operations_Dashboards-1024x455.png)](/vaficionado/assets/images/2015/03/Log_Insight_vRealize_Operations_Dashboards.png)  
(Click the image for a larger version)

It’s also worth noting that if you had previously configured vROps 6.0.x to send its logs to Log Insight directly by editing the logger configuration, you should now **undo** this configuration. Leaving it in place will result in some logs being sent to Log Insight twice, and may even confuse the content pack.

[![vRealize_Operations_Edit_Logger_Configuration](/vaficionado/assets/images/2015/03/vRealize_Operations_Edit_Logger_Configuration.png)](/vaficionado/assets/images/2015/03/vRealize_Operations_Edit_Logger_Configuration.png)

Cheers, and happy analyzing!

<div class="twttr_buttons"><div class="twttr_followme"> [Follow me](https://twitter.com/@vaficionado) </div></div><div class="twttr_buttons"><div class="twttr_twitter"> [Tweet](http://twitter.com/share?text=vRealize+Operations+Manager+Content+Pack+for+Log+Insight)</div></div>