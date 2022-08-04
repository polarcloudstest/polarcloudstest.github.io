---
layout: post
title: "PowerShell Credential Handling" 
excerpt: "Simple Secure Usernames and Passwords"
tags: 
- Pro-Tip
- PowerShell
image:
 thumb: /images/powershell-credential-handling/powershell-credential-handling-01.png
comments: true
date: 2022-08-02T00:00:00+00:00
---
![PowerShell Credential Handling](/images/powershell-credential-handling/powershell-credential-handling-01.png){: width='200' height='200' style='max-width: 200px' .right}
One of the things I like to do on this site is to share handy PowerShell scripts. 

After all PowerShell allows for automation thus making life easier and who wouldn't want an easy life?

Quite often PowerShell scripts need to pass credentials to remote systems/services; for example logging onto an ESXi host or  a vCenter server to perform a task or two. 

How do we handle those credentials? Preferably not in plain text... 

Enter Credential Manager.

## Credential Manager
Credential Manager is accessed via Windows control panel:

![Credential Manager](/images/powershell-credential-handling/powershell-credential-handling-02.png)

The advantages of using Credential Manager to store our PowerShell credentials are as follows:

Credentials stored in credential manager are:

1. Associated with each Windows user account and not transferable between users
2. Not generally transferable between computers (possible if using roaming profiles)
3. Accessible from a full-windows environment that has Credential Manager built in (EG not in WinPE)
4. Relatively easily accessible from PowerShell  

To expand on points 1. and 2. above, remember when running a PowerShell script containing credentials, the credentials referenced must be available to the user account running the script.  For example, when running a PowerShell script as a scheduled task running under the local administrator account, the credentials must be available to the local administrator account used. 

## PowerShell Module Installation
To access credentials stored in Credential Manager from PowerShell we need to install a PowerShell Module. The module is available here in the [PowerShell Gallery](https://www.powershellgallery.com/packages/CredentialManager/2.0){:target="_blank"}.

Installation is simple enough:
{% highlight powershell%}
Install-Module -Name CredentialManager
{% endhighlight %}
![Install Credential Manager Module](/images/powershell-credential-handling/powershell-credential-handling-03.png)

That's it. Restart your PowerShell session to automatically load the module.

## Saving Credentials
Instead of using Credential Manager GUI to add credentials, the `New-StoredCredential` command can be used as follows.

As a bonus, teaming `New-StoredCredential` with `Get-Credential` pops up the credential request window for easy entry:
{% highlight powershell%}
New-StoredCredential -Target "TEST" -Persist "LocalMachine" -Credentials $(Get-Credential) | Out-Null
{% endhighlight %}
![Add Credential Prompt](/images/powershell-credential-handling/powershell-credential-handling-04.png)

Enter credentials as normal and click OK.

Checking Credential Manager afterwards:

![Add Credential Check](/images/powershell-credential-handling/powershell-credential-handling-05.png)

## Retrieving Credentials
Again using PowerShell, credentials can be retrieved using `Get-StoredCredential` command as follows:
{% highlight powershell%}
Get-StoredCredential -Target "TEST"
{% endhighlight %}

![Retrieve Credential](/images/powershell-credential-handling/powershell-credential-handling-06.png)

## Using Credentials
So how do we use the credentials that we can recover from Credential Manager?  For example, how can we use the recovered credentials to, say, logon to a VMware vCenter server?

In the following example, we will recover and use the following credential:

![Using Credential Check](/images/powershell-credential-handling/powershell-credential-handling-07.png)

The two line script is as follows:
{% highlight powershell%}
$Credentials = Get-StoredCredential -Target "vSphere-Admin"
Connect-VIServer -Server "vcenter.local" -Credential $Credentials
{% endhighlight %}
Yep that works nicely:

![Using Credential](/images/powershell-credential-handling/powershell-credential-handling-08.png)

Simple!

## Deleting Credentials
Finally, credentials can be deleted using `Remove-StoredCredential` command as follows:
{% highlight powershell%}
Remove-StoredCredential -Target "TEST"
{% endhighlight %}

![Delete Credential](/images/powershell-credential-handling/powershell-credential-handling-09.png)

Checking Credential Manager:

![Delete Credential Check](/images/powershell-credential-handling/powershell-credential-handling-10.png)

Yep, our test credential has been deleted.

## PowerShell Core on Linux
As Linux does not have a equivalent Credential Manager, we need to get creative when handling credentials in PowerShell core on Linux.

As luck would have it, a work around is available.  What's more is that we documented and used the workaround in part three of the UPS Triggered Shut Down of ESXi from Raspberry Pi series XXXX

## Conclusion and Wrap Up
A solution to implement and manage PowerShell credentials does exist. What's more it's simple to use.<br>
No more storing credentials in plain text inside scripts.

-Chris
