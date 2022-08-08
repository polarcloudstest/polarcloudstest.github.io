---
layout: post
title: "vSphere Compliance with vRealize Operations and Tagging" 
excerpt: "Creating Continual Regulatory Compliance"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- vRealize
image:
thumb: /images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-01.png
comments: true
date: 2022-06-28T00:00:00+00:00
---
![vRO CIS and DISA Logos](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-01.png){: width='200' height='200' style='max-width: 200px' .right}
In these days of anywhere computing, one of the tools in the arsenal of the good guys is system hardening. But what is system hardening?

System hardening is defined as the practice of reducing a system's vulnerability by reducing its attack surface. Through a reduced attack surface, there is a lower risk of data breaches, unauthorized access, system hacking, or malware infection. 

Hardening may involve a reduction in attack surface through cutting unnecessary services or processes. Therefore for most environments a level of system hardening is required, but how much system hardening is required? How are systems hardened? Are there any system hardening compliance standards or guides that we could follow? Which standard best fits my scenario? 

As with so may things in life, there are options. How much hardening depends on your risk appetite. <br>**Spoiler Alert**: a list of compliance standards is included later on in this post (/posts[tl,dr])

For example, two popular hardening benchmarks are CIS and DISA STIG. These are available:
- CIS Benchmarks: [Here](https://downloads.cisecurity.org/){:target="_blank"}
- DISA STIG Benchmarks: [Here](https://public.cyber.mil/stigs/downloads/){:target="_blank"}

## Objectives
Whilst vRealize Operations (vRO) can be integrated into VMware Cloud on AWS, Azure VMware Solution, Google Cloud VMware Engine as well as native Azure and AWS environments, in this post we shall address the hardening of a Software Defined Data Center on premises vSphere environment.

In so doing, we shall be system hardening the following components that make up a vSphere environment:

- ESXi Hosts
- vCenter Servers
- Distributed Switches
- Distributed Switch Ports
- vSAN (if deployed)
- NSX-T (if deployed)
- Virtual Machine Configurations

We will **NOT** be system hardening the guest operating systems running inside the virtual machines. These operating systems have their own system hardening standards / benchmarks so are therefore outside the scope of this article.

## What is vRealize Operations?
From the VMware [marketing blurb](https://www.vmware.com/uk/products/vrealize-operations.html){:target="_blank"}:<br>
VMware vRealize Operations provides self-driving IT Operations Management across private, hybrid and multi-cloud environments with a unified operations platform that delivers continuous performance, capacity and cost optimization, intelligent remediation and integrated compliance through AI/ML and predictive analytics.

Chris' take:<br>
vRO monitors environments for alerts, symptoms and conditions. Compliance standards loaded into vRO are a set of alerts where by an object is not configured to satisfy the condition of the alert will cause the alert to be flagged.

## vSphere Configuration 
As discussed, in this post we shall address the hardening of a Software Defined Data Center on premises vSphere environment.

We will be using the following lab environment:
- vSphere 7 update 3 - Single vCenter and single ESXi host 
- vRO 8.6.3 - Single VM, extra small 

No licences have been procured for this environment. All software is in evaluation mode.

![vSphere](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-02.png)

Networking:

![vSphere Networking](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-03.png)

Yep, nothing complicated at all. Much, much larger environments are available! 

### Why vSphere Tags?
When it comes to compliance, much as we would like it, occasionally (often) there are objects that at any one particular point in time are not required to be compliant with the system hardening benchmark in use across the environment. For example we may be deploying a an isolated test/dev environment or a quick temporary virtual machine, network port group, whatever. At this point in time these do not need to be compliant. Compliance may/can come later.  

Having these known non-compliant environments being continually flagged in vRO is undesirable and may obscure issues elsewhere with objects that absolutely need to be compliant (think: "was it six or seven VMs and two or three port groups that we don't need to care about compliance failures on?!?").

Therefore, how can we "tune out the cruft" from vRO so that we can concentrate on the rest of the environment that absolutely needs to be compliant?

Simple:
1. Configure compliance monitoring so that ALL objects need to be compliant by default
2. Employ a vSphere tag to mark objects that don't need to be compliant
3. Manage by exception

### Configure vSphere Tags
Let's setup our vSphere tags for our compliance exceptions.<br>
From the vSphere Client, select **Menu > Tags & Custom Attributes > Tags** and **Categories**.  From here we will create a new tag category named **Compliance**:

![vSphere Compliance Category](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-05.png)

Next, we need to create a tag that should be applied to the vSphere objects that should be excluded from compliance. Select **Menu > Tags & Custom Attributes > Tags** and **Tags**: 

![vSphere Compliance Tag](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-06.png)

### Apply vSphere Tags
Within my test environment, I want to CIS compliance scan everything **except** the following two VMs:

- WIN-2022 - My Windows 2022 test/dev VM
- vCLS-386d4abf-c432-4d1f-a4a2-c6355100f4b4 - the vSphere Cluster Services VM (see [here](https://kb.vmware.com/s/article/80472){:target="_blank"} for details)

Find the VM Tags dialogue:

![Tag](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-07.png)

Apply the tag:

![Apply  Tag](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-08.png)

Confirm application:

![Confirm Tag](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-09.png)

To confirm which objects have the tag applied and which do not, simply search for the tag in the inventory and select **Objects**:

![List Objects with Tag](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-10.png)

This list can be then be exported and provided to my security/compliance department as proof of those objects NOT in compliance with the hardening applied across the rest of the environment. 
## vRealize Operations Configuration
I won't cover the deployment of the vRealize Operations appliance here. Should you need further info, see [Deployment of vRealize Operations](https://docs.vmware.com/en/vRealize-Operations/8.6/com.vmware.vcom.vapp.doc/GUID-49349FD7-7237-4022-A6A5-1B26D7AFC7DF.html){:target="_blank"} and the [vRealize Operations Sizing Tool](https://vropssizer.vmware.com/){:target="_blank"}.

To integrate vRO with vCenter, logon to vRO with the admin account, select **Data Sources > Integrations > Add Account > vCenter** and add your vCenter details.  

Once vCenter has been integrated, you should see the following:

![vSphere Integration](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-11.png)

### Activating Compliance Standard to Apply to vSphere
Next we need to select a compliance standard baseline to which we require our environment to conform to. 
From the vRealize Operations console, select **Optimise > Compliance**.  As you can see from below, there are several built in Regulatory Standards / Benchmarks available to choose from. These include:

- CIS Security Standards
- DISA Security Standards
- FISMA Security Standards
- HIPAA Compliance
- ISO Security Standards
- PCI Security Standards

As can be seen below:

![Regulatory Benchmarks](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-12.png)

In the following example we will be using the CIS Security Standard to harden our vSphere environment.

In the CIS Security Standard option box, select **Activate from Repository**, **Activate** and **Yes** to activate the CIS standard.  Confirm CIS compliance integration has been installed:

![CIS Activated](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-13.png)

Do not activate the standard just yet. We will handle activation later.

### Configuring a Compliance Standard to Apply to vSphere
As discussed above, given that vRO operates via alerting, lets go find our newly downloaded compliance alerts. From the vRO console, select **Configure > Alerts > Alert Definitions** and filter by compliance standard activated (in my case CIS):

![CIS Alert Definitions](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-14.png)

As can be seen in the case of CIS, there are four alerts that have been created.

Notice that there is no alert configuration for vCenter server. The compliance standard chosen - CIS - does not contain hardening standards for vCenter. Others hardening compliance standards such as STIG certainly do contain [hardening standards for vCenter](https://www.stigviewer.com/stig/vmware_vsphere_6.7_vcenter/){:target="_blank"}.

#### ESXi Host is Violating CIS Alert Definition
So that we may always have untouched alert definitions to go back to later should we need to, we will clone the alert definitions to create our own. 

Using the three dots to the right of each of the alert definitions, select clone:
![Clone Alert Definition](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-15.png)

To make my alert definitions easier to find in the future, I shall suffix my alert definitions with "PolarClouds":

![Name Clone Alert Definition](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-16.png)

Click **Next**. Here we can see the individual tests that constitute the "violating CIS" vRO alert:

![Name Clone Alert Definition Tests](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-17.png)

From here, we can remove any unwanted symptoms that we do not want to test for. Typically I remove the following tests:

- Use Active Directory for local user authentication <sup>1</sup>
- Enable vSphere Authentication Proxy when adding hosts to Active Directory <sup>1</sup>
- Enable bidirectional CHAP authentication for iSCSI traffic <sup>2</sup>

<sup>1</sup> *Whilst I agree AD integration is something that should be considered for ease of administration, in my experience it is not something regularly implemented. From a 10,000 foot view, I would argue that Active Directory is a  larger attack vector than ESXi.*<br>
<sup>2</sup> *I'm not using iSCSI in my lab.*

Use the **X** to remove the unwanted symptoms:

![Remove Test](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-18.png)

Click **Next > Next**, ensure no policies are selected to be enabled (again we will do this later) and finally **Create** to create the new PolarClouds ESXi Host is violating CIS policy. 

#### Virtual Machine is Violating CIS Alert Definition
Repeat above clone and modification action for the Virtual Machine is violating CIS alert definition. For the PolarClouds Virtual Machine is violating CIS alert definition, I shall remove the following symptoms:

- CD-ROM connected (5.5 Hardening Guide) <sup>1</sup>
- VGA only mode is not enabled (5.5 Hardening Guide) <sup>2</sup>
- USB controller connected (5.5 Hardening Guide) <sup>1</sup>

<sup>1</sup> *vSphere 7 CIS benchmark v1.1.0, section 8.3.1 discusses "disabling unnecessary system components". CD-ROM drives and USB adapters are necessary. Some methods of VMtools updates are not possible without a CD-ROM.*<br>
<sup>2</sup> *According to latest vSphere 7 CIS benchmark (v1.1.0) this test has been removed*<br>

Click **Next > Next**, ensure no policies are selected to be enabled (again we will do this later) and finally **Create** to create the new PolarClouds Virtual Machine is violating CIS policy. 

#### vSphere Distributed Port Group is Violating CIS Alert Definition
Repeat above clone and modification action for the vSphere Distributed Port Group is violating CIS alert definition. For the PolarClouds vSphere Distributed Port Group is violating CIS, you also need to set the Base Object Type to **vCenter Adapter > vSphere Distributed Port Group** and Alert Type and Subtype to **Network: Compliance**:

![PolarClouds vSphere Distributed Port Group is violating CIS](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-19.png)

Out of the box at the time of writing, the VMware CIS template has no symptoms set for this alert. Therefore for this alert rather than removing symptoms we will be adding them. This is simple to do. 

Select **Symptoms** in the right hand pane of the **Symptoms / Conditions** dialogue and use the filter to find the symptoms to add:

![Find Symptoms](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-20.png)

Once a symptom is found, use drag and drop to add the symptom to the alert.

For the PolarClouds vSphere Distributed Port Group is violating CIS alert I shall add the following and the set is met when **Any** of the symptoms / conditions are true:

- vNetwork.reject-forged-transmit-dvportgroup - The Forged Transmits policy is not set to reject <sup>1</sup>
- vNetwork.reject-mac-changes-dvportgroup - The MAC Address Changes policy is not set to reject <sup>2</sup>
- vNetwork.reject-promiscuous-mode-dvportgroup - The Promiscuous Mode policy is not set to reject <sup>3</sup>
- vNetwork.restrict-port-level-overrides - Port-level configuration VLAN overrides on VDS is not restricted <sup>4</sup>

<sup>1</sup> *vSphere 7 CIS benchmark v1.1.0, section 7.1*<br>
<sup>2</sup> *vSphere 7 CIS benchmark v1.1.0, section 7.2*<br>
<sup>3</sup> *vSphere 7 CIS benchmark v1.1.0, section 7.3*<br>
<sup>4</sup> *vSphere 7 CIS benchmark v1.1.0, section 7.8*<br>

My PolarClouds vSphere Distributed Port Group is violating CIS alert Symptoms now resembles the following:

![PolarClouds vSphere Distributed Port Group is violating CIS alert Symptoms](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-21.png)

Click **Next > Next**, ensure no policies are selected to be enabled (again we will do this later) and finally **Create** to create the new PolarClouds vSphere Distributed Port Group is violating CIS policy. 

#### vSphere Distributed Virtual Switch is Violating CIS Alert Definition
Repeat above clone and modification action for the vSphere Distributed Virtual Switch is violating CIS alert definition. For the PolarClouds vSphere Distributed Virtual Switch is violating CIS alert definition, I will go with the defaults, no symptom modifications required:

![PolarClouds vSphere Distributed Virtual Switch is violating CIS alert Symptoms](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-22.png)

Click **Next > Next**, ensure no policies are selected to be enabled (again we will do this later) and finally **Create** to create the new PolarClouds vSphere Distributed Virtual Switch is violating CIS. 

#### Alert Definition Wrap Up
Just to double check then, all in all we have four custom CIS policies:

![PolarClouds Policies](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-23.png)

To be 100% above board, submit details of the tests removed along with reasons for removal to your compliance and/or security department(s) for safe keeping. These will need logging as exceptions to the selected system hardening benchmark, in our case CIS.

### Activating Alert Policies
So let's active our custom CIS policies. From the vRO console, select **Configure > Policies > Add**. I shall name my policy **PolarClouds CIS Compliance Policy** and I shall inherit from **Base Settings**:

![PolarClouds CIS Compliance Policy](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-24.png)

After clicking **Create Policy**, lets add our custom alert definitions. Click **Alerts and Symptoms** and lets filter on "CIS" to see both the default and our custom PolarClouds CIS alert definitions:

![Alerts and Symptoms](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-25.png)

Using the drop downs, enable the PolarClouds policies:

![Enable Definition](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-26.png)

![PolarClouds Enabled Definitions](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-27.png)

Finally click **Save**.

### Custom Groups 
So that we can leverage our vSphere tags for compliance scanning, we need to create some custom groups in vRO.

From the vRO console, select **Environment > Custom Groups > Add**. I shall name my custom group **PolarClouds CIS Compliance Group**. 

The group type will be **Environment** and the policy will be our previously created **PolarClouds CIS Compliance Policy**. I shall tick **Keep group membership up to date**.

For membership criteria, I shall select object type **vCenter Adapter > Virtual Machine**, **Properties**, **Summary > vSphere Tag**, **does not contain** and **[< Compliance-CIS-Excluded >]**:

![Group VM Criteria](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-28.png)

Add another criteria set and repeat for vCenter Adapter > Host System, vSphere Distributed Port Group and vSphere Distributed Switch (you may need to copy and paste in [< Compliance-CIS-Excluded >] ):

![Group Criteria](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-29.png)

Let's preview our group membership:

![Group Preview](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-30.png)

We have a distributed switch, some port groups, a VM and an ESXi host in our group. 

Notice that the vCLS and Win-2022 VMs are not listed in our custom group as these both have the CIS-Excluded vSphere tag applied to them. Nice! 

Close the preview and OK the group.

### Custom Compliance Benchmark
From the vRO console, select **Optimise > Compliance > Add Custom Compliance > Create a new Custom Benchmark**. I shall name my benchmark **PolarClouds CIS Compliance**:

![Custom Compliance Benchmark Name](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-31.png)

I shall select my four PolarClouds alert definitions: 

![Custom Compliance Benchmark Definitions](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-32.png)

Finally I shall enable **PolarClouds CIS Compliance Policy**:

![Custom Compliance Benchmark Policy](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-33.png)

Once complete the initial assessment should begin:

![Initial Assessment Running](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-34.png)

## vRealize Operations vSphere CIS Compliance Results
So after all that configuration, let's see where we need to harden our vSphere environment. From the vRO console, select **Optimise > Compliance > Custom Benchmark** - in my case **PolarClouds CIS Compliance**:

![PolarClouds Initial Compliance](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-35.png)

OK, so one out of three isn't bad! Clicking on the alerts allows us to dig into the detail:

![Host Compliance Failures](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-36.png)

OK, so the ESXi host has five compliance failures.

![Port Group Compliance Failures](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-37.png)

My Distributed Port PG-TEST group has promiscuous mode enabled.

![VM Compliance Failures](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-38.png)

And finally some doughnut has added a serial port to a VM. :doughnut: :grin:

## Conclusion and Wrap Up
Much as I try not to, we are going to have to call a temporary stop to proceedings here.

In this post we looked at regulatory compliance standards and benchmarks, vSphere tag creation and application and finally (for the majority of the post) at configuring vRealize Operations to continually monitor for system hardening benchmark compliance.

Next time we will look at creating a dashboards and reports to publicise our example PolarClouds lab environment CIS compliance and and non-compliances further.

Sure compliance is a dry and often a difficult subject to crack, however hopefully with the use of a automated and continuous monitoring tool such as vRealize Operations, we can ensure that our vSphere environment is always meeting its required compliance standard, whatever standard that may be.

This post is part 1 of a multipart series. Find the other parts here:

- Part 1: This part: Creating Continual Regulatory Compliance
- Part 2: [Monitoring and Reporting Regulatory Compliance](/posts/vsphere-compliance-with-vro-pt2/){:target="_blank"})

-Chris
