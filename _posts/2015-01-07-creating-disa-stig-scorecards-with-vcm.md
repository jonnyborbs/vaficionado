---
id: 34
title: 'Creating DISA STIG Scorecards with vCM'
date: '2015-01-07T16:52:58-08:00'
author: jon
layout: post
image: /assets/images/cropped-va-header.jpg
guid: 'http://www.vaficionado.com/?p=34'
permalink: /2015/01/creating-disa-stig-scorecards-with-vcm/
categories:
    - 'Compliance and Security'
    - Tech
tags:
    - compliance
    - disa
    - security
    - stig
    - vcm
---

In my previous life as an InfoSec guy, I was responsible for assessing, enforcing, and ensuring continuous compliance with all the various baselines for which my organization was responsible. At the forefront of this list were a long list of DISA STIGs (Defense Information Systems Agency Security Technical Implementation Guides) – a daunting task in any size environment with any size staff. Of course, this particular environment was fairly large, and the information assurance technical staff consisted basically of… me… so automating these processes became something of a necessity.

This is where one of VMware’s most versatile products comes in – the vRealize Configuration Manager (vCM.) This gem of a tool provides unified, cross-platform configuration and compliance management and enforcement of over 80,000 distinct controls from a single interface, complete with fully customizable reports, dashboards, and a whole host of other fun features.

Anyway, enough sales. On to the how-to.

This tutorial assumes you already have vCM installed and configured and able to communicate with at least one managed system. For this example we will be creating a Windows 2008 R2 scorecard.

To start, we will need to download the STIG content and the viewer tool straight from DISA. The content is what’s known as a “Benchmark” and can be obtained from <https://iase.disa.mil/stigs/Pages/a-z.aspx>. The one we will want for this example is the “[Windows 2008 R2 MS STIG Benchmark – Version 1, Release 15](https://iase.disa.mil/stigs/Documents/u_windows_2008_r2_ms_v1r15_stig_scap_1-0_benchmark.zip)”

[![Benchmarks](/assets/images/2015/01/Benchmarks-300x182.png)](/assets/images/2015/01/Benchmarks.png)

A few notes here.

1. These benchmarks will update quarterly, on a fixed schedule found [here](https://iase.disa.mil/stigs/Pages/fso-schedule.aspx).
2. You will notice that there are “MS” and “DC” STIG bundles for Windows operating systems. MS refers to Member Servers, and DC to Domain Controllers. This is because there are additional and special requirements depending on which role the server fills. Make sure you select the appropriate bundle for your target system.
3. You will also notice that there are “Benchmark” and “STIG” bundles. The STIG bundle contains much more information as well as a whole host of manual (non-automated) checks which are out of scope for this guide.
4. Any time you see the \*PKI designation next to a link, this means that a DoD issued PKI certificate (smart card) is required to access this content. If you have one, great! If not, you simply won’t have access to this particular information.

Next, you will need to download the STIG Viewer. This is a Java based JAR application which will allow you to view, interact with and update your STIG scorecards. This can be obtained from <https://iase.disa.mil/stigs/Pages/stig-viewing-guidance.aspx>. Click the link for “STIG Viewer Version 1.2.0” in this example.

[![STIGViewer](/assets/images/2015/01/STIGViewer.png)](/assets/images/2015/01/STIGViewer.png)

Note that you will need to have a functioning JRE installed on your vCM server to use this tool.

Once you have the STIG Viewer and the appropriate benchmark for your guest operating system downloaded to the vCM server, we need to place the benchmark in vCM’s SCAP import folder. In a default install of vCM, this folder is found at **C:\\Program Files (x86)\\vmware\\vcm\\WebConsole\\L1033\\Files\\SCAP\\import**

[![SCAPImport](/assets/images/2015/01/SCAPImport.png)](/assets/images/2015/01/SCAPImport.png)

Tip: SCAP is the Security Content Automation Protocol, a standard designed to provide a framework for vulnerability management by the National Vulnerability Database.

Once the file is copied to the import location, it’s time to fire up the vCM console.

1. Log in as a user with the Admin role or a custom role with access to the Compliance tools.
2. Click on the **Compliance** slider on the left
3. Expand the **SCAP Compliance** spinner
4. Click on **Benchmarks**
5. Click **Import** on the right hand panel to bring up the list of available SCAP benchmarks
6. Using the arrow controls in the middle of the dialog, move the benchmarks you wish to import to the right hand side and click **Next**, followed by **Finish** on the next dialog[![BenchmarkImport](/assets/images/2015/01/BenchmarkImport.png)](/assets/images/2015/01/BenchmarkImport.png)

You will now see the new benchmarks listed in the **Compliance** slider on the left. If you expand them, you will notice that they are broken down into MAC (Mission Assurance Category) and CL (Confidentiality Level) categories. Be sure you know the MAC and CL for the systems you plan to audit – the affects the stringency of certain technical controls.

[![MACandCL](/assets/images/2015/01/MACandCL.png)](/assets/images/2015/01/MACandCL.png)

Now it’s time to run a collection against the systems you want to audit. There are many ways to accomplish this, and I’m going to assume you have your own preferred method – but here’s a quick one just in case.

1. Select **Collect** from the main toolbar at the top of the vCM interface
2. Select **Machine Data** and select **OK**
3. Choose the machine(s) you wish to audit – either from the list on the left, or use the filter, or machine groups, etc.
4. Select **Select a Collection Filter Set to apply to these machines** and click **Next**[![Collection1](/assets/images/2015/01/Collection1.png)](/assets/images/2015/01/Collection1.png)
5. Select the **Regulatory Baseline Filters – Windows** (for this example) filter set and click **Next**[![Collection2](/assets/images/2015/01/Collection2.png)](/assets/images/2015/01/Collection2.png)
6. Click **Finish**

Now you need to monitor the collection job until it completes successfully. Wait until the job disappears completely from the Jobs list before continuing to the next step. This ensures that the data is fully merged into the vCM database.

[![JobsView](/assets/images/2015/01/JobsView.png)](/assets/images/2015/01/JobsView.png)

Now we can return to the **Compliance** slider.

1. Expand the **SCAP Compliance** spinner, followed by the **Benchmarks** and the appropriate benchmark for the OS you are going to audit
2. Select the appropriate MAC and CL for the system in question. For this example, we will use **MAC-2\_Sensitive**
3. Click **Run Assessment** in the right hand panel
4. Select the machines you wish to audit from the upper list and move them to the lower list using the arrow controls  
    [![RunSCAP](/assets/images/2015/01/RunSCAP.png)](/assets/images/2015/01/RunSCAP.png)
5. Click **Next**
6. Select if you wish to run the action now or later. For this example we will select **Run Action Now** and click **Next**
7. Click **Finish**

A **Windows SCAP Assessment** job will be submitted to the Jobs list. Monitor this until it completes, then select the appropriate MAC and CL from the **Benchmarks** list again to refresh the view.

You should see a list of your assessed servers that looks like this:

[![ResultOptions](/assets/images/2015/01/ResultOptions.png)](/assets/images/2015/01/ResultOptions.png)

Now you have quite a few options. You can choose from the pre-configured result types that vCM provides for you – the OVAL HTML result is a nicely formatted human-readable report that’s suitable for a build book, hard copy, etc:

[![OVALResults](/assets/images/2015/01/OVALResults.png)](/assets/images/2015/01/OVALResults.png)

But, to generate content that will work with the DISA STIG Viewer, you need to export an XCCDF-formatted XML file. To do this:

1. Click **Export** from the toolbar
2. Select the machines you wish to export data for. Each machine will generate its own XML file
3. Click **Next**
4. Select **XCCDF Results – XML**
5. Click **Finish**

You will receive a dialog that looks like this when the export is complete.

[![ExportResults](/assets/images/2015/01/ExportResults.png)](/assets/images/2015/01/ExportResults.png)

Navigate to this folder: in a default vCM install it is **C:\\Program Files (x86)\\vmware\\vcm\\WebConsole\\L1033\\Files\\SCAP\\export**

Here you will see a list of the exported results files for the servers you selected in the last step.[![ExportedFiles](/assets/images/2015/01/ExportedFiles.png)](/assets/images/2015/01/ExportedFiles.png)

We’re almost there. Take a deep breath and another drink of your favorite adult beverage. Today I am personally drinking a truly excellent 2010 Miner Cab from the much-coveted Stagecoach vineyard. This vineyard in the eastern hills of the Napa Valley produces fruit for some of the biggest name wines around, and with good reason.[![MinerStagecoach](/assets/images/2015/01/MinerStagecoach.jpg)](/assets/images/2015/01/MinerStagecoach.jpg)

Refreshed? Good – back to the STIGs. Now you’re going to want to fire up that DISA STIG Viewer we downloaded at the beginning. Provided you have a properly installed JRE, you should just be able to double-click the JAR file.

You’ll then be greeted with this friendly government-issue GUI. Never fear.[![STIGViewerGUI](/assets/images/2015/01/STIGViewerGUI.png)](/assets/images/2015/01/STIGViewerGUI.png)

1. Select **File** and **Import STIG from ZIP** from the menu bar
2. If this is your first time importing a STIG bundle, you will be prompted to create a savepoint. Select **Yes**
3. Navigate to the folder where you stored the STIG Benchmarks we downloaded at the beginning of this guide. Be sure to select the one(s) which apply to the compliance results you exported earlier.
4. You’ll see the viewer is now populated with STIG controls.[![STIGControls](/assets/images/2015/01/STIGControls.png)](/assets/images/2015/01/STIGControls.png)
5. Now we must create a checklist from this raw data. Select **Checklist** and **Create Checklist – Current STIG** from the menu bar
6. You will now have a STIG Checklist which you can enter your own data into. Notice that the **Host Target Data** in the lower left corner is not populated, and all of the vulnerability statuses are set to **Not Reviewed**[![ChecklistView](/assets/images/2015/01/ChecklistView.png)](/assets/images/2015/01/ChecklistView.png)
7. This is it: the last step. Select **Import** from the menu bar, followed by **Import XCCDF Results**. Navigate to the XML file you exported from vCM earlier. Remember, by default it was located in **C:\\Program Files (x86)\\vmware\\vcm\\WebConsole\\L1033\\Files\\SCAP\\export**
8. **Voila!** You will see that the checklist has been filled out for you. You can now review the checklist, mark it up, make manual severity/status overrides, etc. If you save it as a .CKL file, it will be an acceptable artifact to most DoD certified auditors for the purposes of DIACAP/RMF. [![CompletedChecklist](/assets/images/2015/01/CompletedChecklist.png)](/assets/images/2015/01/CompletedChecklist.png)

Of course, you must be sure to be aware that every audit is different and you should check with your DOIM or local IA department to confirm these documents will be acceptable/sufficient for your purposes.

Tip: Much of what we just did can be scheduled inside of vCM. This removes a LOT of the manual work.

I hope this guide has been useful to you.