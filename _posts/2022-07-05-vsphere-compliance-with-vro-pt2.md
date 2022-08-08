---
layout: post
title: "vSphere Compliance with vRealize Operations and Tagging - Part 2" 
excerpt: "Monitoring and Reporting Regulatory Compliance"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- vRealize
image:
thumb: /images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-01.png
comments: true
date: 2022-07-05T00:00:00+00:00
---
![NSX-T Logo](/images/vsphere-compliance-with-vro/vsphere-compliance-with-vro-01.png){: width='200' height='200' style='max-width: 200px' .right}
Last time we looked at regulatory compliance standards and benchmarks, vSphere tag creation and application and finally for the majority of the post at configuring vRealize Operations to continually monitor for system hardening standard / benchmark compliance.

If you have not yet seen that post, catch up now. It’s a great read. :wink:

As mentioned, this post is part 2 of a multipart series. Find the other parts here:

- Part 1: [Creating Continual Regulatory Compliance](/posts/vsphere-compliance-with-vro/){:target="_blank"})
- Part 2: This part: Monitoring and Reporting Regulatory Compliance

## Monitoring Compliance
We need a dashboard. Let's create one! Luckily for us vRealize Operations (vRO) has just the thing.

From the vRO console, select **Visualise > Dashboards** and let's search for compliance:

![Find Compliance Dashboard](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-01.png)

Not sure why the built in compliance dashboard is deprecated, but hey lets use it as a base for a new custom dashboard anyway. Taking a look at the dashboard:

![vSphere Compliance Dashboard](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-02.png)

Nice. Let's clone the dashboard to create our own. Click **Manage** and filter on **compliance** to find the dashboard in the dashboard library:

![Find Compliance Dashboard in Library](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-03.png)

Use the three vertical dots to clone the dashboard:

![Clone Dashboard](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-04.png)

I'll name my new dashboard **PolarClouds CIS Security Compliance**:

![Name Clone Dashboard](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-05.png)

Again, using the three vertical dots, let's edit the PolarClouds CIS Security Compliance dashboard. I'm not going to go into super detail here as dashboards are a can be subject to personal taste, but here are the changes I've made (you are of course free to make your own!):

- Removed the vSphere and VM Compliance heat maps:

![Remove Heat Maps](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-06.png)

- Group the affected objects by "None":

![Group by None](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-07.png)

As this is a small environment with eventually zero compliance issues, I'm happy to put the hopefully empty list on non-compliances right on my dashboard!  I'll rename the widget "Non-Compliant Objects" too.

That's it. I'll save the dashboard and using the share button, setting the expiry to **Never Expire**, I'll grab the link to my nice new dashboard:

![Share Dashboard](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-08.png)

Lets take a look at the final dashboard:

![Final Dashboard](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-09.png)
	
Happy with that.  Yes, there is some hardening of the environment yet to do, I'll forward the link to the security and compliance departments.

## Reporting Compliance
Next, let's work on reporting. Reports are consist of one or more views, so we'll work on creating our views first.

For my report I'm going to include the following views:

- CIS Compliant Virtual Machines
- CIS Compliant ESXi Hosts
- CIS Compliant Virtual Switches
- CIS Compliant Virtual Switch Port Groups
- CIS Non-Compliant Virtual Machines
- CIS Non-Compliant ESXi Hosts
- CIS Non-Compliant Virtual Switches
- CIS Non-Compliant Virtual Switch Port Groups
- CIS Excluded Virtual Machines
- CIS Excluded ESXi Hosts
- CIS Excluded Virtual Switches
- CIS Excluded Virtual Switch Port Groups

Sure twelve views is a lot (remember CIS does not harden vCenter), but through the power of cloning and tweaking the filters we really only have to create four views.

### Views 
#### Virtual Machines
From the vRO console, select **Visualise > Views > Manage > Add** and let's create a view to add to a report. I'll call my view "PolarClouds CIS Compliant Virtual Machines" and set the description as "Virtual Machines with vSphere configuration compliant with CIS Hardening Standards":

![Compliance Report Title + Description](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-10.png)

Selecting step 2, lets create some views. 

Click **+** to add a view. We will name this first view **PolarClouds CIS Compliant Virtual Machines** and set the description ro **Virtual Machines with vSphere configuration compliant with CIS Hardening Standards**:

![Report Compliant VM View Name + Description](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-11.png)

Next, lets present some data in list form, so select **List**:

![Report Compliant VM View List](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-12.png)

Moving onto step 3, lets set our subjects as **vCenter Adaptor > Virtual Machine**

![Report Compliant VM Subject List](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-13.png)

Moving onto step 4, lets find some data to populate our list with. When we find our property / metric, simply drag to add to the list:

![Report Compliant VM Subject List Drag to Add](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-14.png)

Label the Metric and add a sort order:

![Metric + Order](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-15.png)
 
The full list of VM properties and metrics I use is, along with their paths is:

- VM IP Address : Properties > Summary > Guest Operating System>Guest OS IP Address
- VM Operating System : Properties > Summary > Guest Operating System > Guest OS from Tools
- VM CIS Compliance : Metrics > Badge > Compliance (%)

Next, lets filter to show only the VMs that are 100% compliant:

![Filter Compliant VMs](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-16.png)

The final configuration of the view:

![Compliant VMs View](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-17.png)

Finally click **Save** to save the view.

Next we'll create the CIS Non-Compliant Virtual Machines view.

Clone the previously created view:

![Clone Compliant VMs View](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-18.png)

Update Name and description:

![Non-Compliant VMs Name + Description](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-19.png)

Select **4. Data**<br>
Change compliance operator to **is not**<br>
Add **Properties > vSphere Tag > Current > is not > [< Compliance-CIS-Excluded >]**:

![VMs is Not Compliant Filter](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-20.png)

Finally save the view.

Given that we know that we have some non-compliant VMs, we can preview the view. Select the Non-Compliant view from the Recents list, click **Select preview source** and select **vCenter Adapter > vSphere World . vSphere World**. Looking good:

![VMs Not Compliant](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-21.png)

Just our VM with a serial port!

Finally lets clone and tweak the filter again to create our CIS Excluded Virtual Machines view. I've also removed the VM CIS Compliance column from the view.

![VMs Excluded 1](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-22.png)

Using vSphere World as a preview source:

![VMs Excluded 2](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-23.png)

Yep our two excluded VMs are listed. Hey presto we have our three VM views:

![Three VM Views](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-24.png)
#### ESXi Hosts
From the vRO console, select **Visualise > Views > Manage > Add** and let's create a view to add to a report. I'll call my view "PolarClouds CIS Compliant ESXi Hosts" and set the description as "ESXi Hosts with vSphere configuration compliant with CIS Hardening Standards".

Again I shall configure presentation in list form and I'll include the following properties and metrics:
- Host IP Address : Properties > Network > Management Address
- Host Operating System : Properties > System > Product String
- Host Version - Build Number: Properties > Summary > Version
- Host CIS Compliance : Metrics > Badge > Compliance (%)

![Host View](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-25.png)

Again we'll clone and set our filters accordingly to create the three views:
- CIS Compliant ESXi Hosts
  - Filter : Metrics > Badge > Compliance (%) > Current > is > 100
- CIS Non-Compliant ESXi Hosts
  - Filter :  Metrics > Badge > Compliance (%) > Current > is not > 100
  - And : Properties > Summary > vSphere Tag > Current > is not > [< Compliance-CIS-Excluded >]
- CIS Excluded ESXi Hosts
  - Filter : Properties > Summary > vSphere Tag > Current > is > [< Compliance-CIS-Excluded >]

#### Virtual Switches
Again three new views with the following properties and metrics:
- Switch Version - Build Number : Properties > Summary > Version
- Switch CIS Compliance : Metrics > Badge > Compliance (%)

Clone and set our filters accordingly to create the three views:
- CIS Compliant Virtual Switches
  - Filter : Metrics > Badge > Compliance (%) > Current > is > 100
- CIS Non-Compliant Virtual Switches
  - Filter :  Metrics > Badge > Compliance (%) > Current > is not > 100
  - And : Properties > Summary > vSphere Tag > Current > is not > [< Compliance-CIS-Excluded >]
- CIS Excluded ESXi Hosts
  - Filter : Properties > Summary > vSphere Tag > Current > is > [< Compliance-CIS-Excluded >]
  
  #### Virtual Switch Port Groups
Finally three more views with the following properties and metrics:
- Port Group VLAN ID : Properties > Configuration > Policies > Security > VLAN ID
- Port Group VLAN Trunk : Properties > Configuration > Policies > Security > VLAN trunk range
- Port Group CIS Compliance : Metrics > Badge > Compliance (%)

Clone and set our filters accordingly to create the three views:
- CIS Compliant Virtual Switch Port Groups
  - Filter : Metrics > Badge > Compliance (%) > Current > is > 100
- CIS Non-Compliant Virtual Switch Port Groups
  - Filter :  Metrics > Badge > Compliance (%) > Current > is not > 100
  - And : Properties > Summary > vSphere Tag > Current > is not > [< Compliance-CIS-Excluded >]
- CIS Excluded Virtual Switch Port Groups
  - Filter : Properties > Summary > vSphere Tag > Current > is > [< Compliance-CIS-Excluded >]
  
Phew! Done!

![PolarClouds Views](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-26.png)

### Reports
Now that we have sliced and diced out data into views, lets bundle the results into a report for review and action.

From the vRO console, select **Visualise > Reports > Manage > Add** and let's create a report. I'll call my report "PolarClouds vSphere Estate CIS Compliance" and set the description as "Status of vSphere CIS Compliance across the PolarClouds vSphere Estate"

Next, add the twelve views previously created:

![PolarClouds Report Views](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-27.png)

Finally, I'll enable PDF and CSV formats, add a cover page, table of contents and a footer.

![PolarClouds Report Layout Options](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-28.png)

Save, find in the list and run the report against vSphere World:

![Run the Report](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-29.png)

And here is the report:

![PolarClouds Report](/images/vsphere-compliance-with-vro-pt2/vsphere-compliance-with-vro-pt2-30.png)

Grab your copy of the report: [PDF](/documents/PolarClouds-Lab-CIS-Compliance.pdf){:target="_blank"} [CSV])

As you can see, I added a slightly tweaked version of the dashboard to the report too, thus making a nice summary page :sunglasses:
## Conclusion and Wrap Up
So there we have it. 

In this post we looked at creating dashboards, views and reports to publicise our example PolarClouds lab environment CIS compliance and and non-compliances further.  

All that is left is to harden the environment keeping an eye on our dashboard and reports as we go.

As you will have seen over the course of this series, vRealize Operations is able to assist with meeting and reporting security compliance across the vSphere estate and ensuring that compliance remains in place via dashboards and reports.

### Chris' Final Thought 1
[Trust, but verify](https://en.wikipedia.org/wiki/Trust,_but_verify){:target="_blank"}. 
Compliance standards can and will change. 

Therefore don't take VMware's vRO compliance as the final word for a compliant environment. Third party compliance scanning solutions such as those available from [Tenable](https://www.tenable.com/solutions/compliance){:target="_blank"} and [Qualys](https://www.qualys.com/solutions/compliance/){:target="_blank"} exist for a reason.

### Chris' Final Thought 2
Usual disclaimer applies.
<br>
And that's it!<br>

As mentioned, this post is part 2 of a multipart series. Find the other parts here:

- Part 1: [Creating Continual Regulatory Compliance](/posts/vsphere-compliance-with-vro/){:target="_blank"})
- Part 2: This part: Monitoring and Reporting Regulatory Compliance

As I said at the end of part 1:<br>
*Sure compliance is a dry and often a difficult subject to crack, however hopefully with the use of a automated and continuous monitoring tool such as vRealize Operations, we can ensure that our vSphere environment is always meeting its required compliance standard, whatever standard that may be.*

-Chris
