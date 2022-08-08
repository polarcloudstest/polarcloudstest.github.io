---
layout: post
title: "UPS Triggered Shut Down of ESXi from Raspberry Pi - Part 3" 
excerpt: "Scripting for the Win... Or Should that be for the Failure?"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- Linux
image:
thumb: /images/esxi-rpi-ups-pt1/esxi-rpi-ups-pt1-00.png
comments: true
date: 2022-07-23T00:00:00+00:00
---
![APC UPS Triggered shut down of ESXi from Raspberry Pi](/images/esxi-rpi-ups-pt1/esxi-rpi-ups-pt1-00.png){: width='200' height='200' style='max-width: 200px' .right}
*Sorry about the tardiness of this post.  I had it written, lost it and then found it again...*
<br>
<br>
Last ~~time~~ year we looked at our Uninterruptible Power Supply (UPS) hardware setup and the installation of the required software for our solution. If you’ve not seen that post, catch up now. It’s a great read. :wink:

As mentioned, this post is part 3 of a multipart series. Find the other parts here:

- Part 1: [Hardware, Requirement, Software, Solution](/posts/esxi-rpi-ups-pt1/){:target="_blank"})
- Part 2: [Hardware Connectivity and Software Installation](/posts/esxi-rpi-ups-pt2/){:target="_blank"})
- Part 3: This part - Scripting for the win... or should that be for the failure?

First off a solution refresher of what we are trying to achieve in this series.

## Solution (Refresher)
![The Solution](/images/esxi-rpi-ups-pt1/esxi-rpi-ups-pt1-01.png)

1. Mains electricity fails... power cut!
2. The UPS signals to the Raspberry Pi that there is a power cut
3. The UPS signals its battery charge state to the Raspberry Pi 
4. The UPS battery charge falls below a predetermined threshold and signals this to the Raspberry Pi
5. The Raspberry Pi runs a script to shut down all powered on VMs 
6. The Raspberry Pi runs a script to shut down the ESXi host
7. The Raspberry Pi runs a script to shut itself down
8. The UPS stops supplying power from battery and shuts down which also shuts down the modem, router and network switch 

Let's get to it.

## Script Overview
Lets look at what we need our script to achieve. Quite simple when we boil it down:

1. Login to ESXi
2. Find and shutdown all powered on VMs
3. Shutdown ESXi server
4. Shutdown Raspberry Pi

Simples!

First a couple of notes:

## PowerShell Credential Handling
As the PowerShell script will run in unattended mode, we need to find a method of storing ESXi credentials. 

My least preferred option is to place the credentials into the script in clear text. My preferred method of using the Windows Credential Manager module (available [here](https://www.powershellgallery.com/packages/CredentialManager/2.0){:target="_blank"}) unsurprisingly does not work when running PowerShell on Linux. Therefore we are going to have to go for a the middle of the road solution of storing the password as an encrypted string in a text file.

To do this, we simply need to run the following which will output our encrypted password to the file `/home/chris/cred.txt`:
{% highlight powershell %}
$credential = Get-Credential 
$credential.Password | ConvertFrom-SecureString | Set-Content /home/chris/cred.txt
{% endhighlight %}
To "reconstitute" the password and combine with our user ID so that it can be used with the `Connect-VIServer -Credential` parameter, we need to include the following in our script:
{% highlight powershell %}
$Username = "root"
$Credfile = "/home/chris/cred.txt"
$Encrypted = Get-Content $Credfile | ConvertTo-SecureString
$Credential = New-Object System.Management.Automation.PsCredential($Username, $Encrypted)
{% endhighlight %}

## Telegram Alerting
One thing we've not touched on in this series yet is the need for notifications and alerting. It is always good to know what is going on with the UPS and the Raspberry Pi during a power cut.

In my solution script below, I'm going to use Telegram for notifications. Thinking here is that I will still receive the notifications during a power cut on my phone via 4 or 5G as at this point my non-UPS backed WiFi access points will also have had their power cut. 

If you've not seen my post on sending Telegram messages from PowerShell, what are you waiting for? It's simple.  :wink:

Slight update to the Send-Telegram script to support Markdown formatting of messages:

<figure><figcaption><b>Filename:</b>/home/chris/send-telegram2.ps1</figcaption> 
{% highlight powershell%}
#! /usr/bin/pwsh
Param(
[Parameter(Mandatory=$true)]$Message,
[Parameter(Mandatory=$false,HelpMessage="Specify Markdown or HTML formatting (Default = Markdown)")][string]$ParseMode = "Markdown"
)
$Telegramtoken = "<TELEGRAM TOKEN>"
$Telegramchatid = "<TELEGRAM CHAT ID>"
# ================================
$payload = @{
"chat_id"    = $Telegramchatid;
"text"       = $Message
"parse_mode" = $ParseMode
}
# ================================
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$Response = Invoke-RestMethod `
-Uri ("https://api.telegram.org/bot{0}/sendMessage" -f $Telegramtoken) `
-Method Post `
-ContentType "application/json" `
-Body (ConvertTo-Json -Compress -InputObject $payload) `
-ErrorAction Stop
{% endhighlight %}
</figure>
Don't forget to mark the script executable using `chmod +x send-telegram2.ps1`
## The Shut Down Script
Rather than post script snippets and talking about them for sections and sections, here is the complete script: 
<figure><figcaption><b>Filename:</b> /home/chris/shutdown.ps1</figcaption> 
{% highlight powershell%}
#! /usr/bin/pwsh
# == Complete These ==============
$ESXi = "esxi-server.local"
$Username = "root"
$Credfile = "/home/chris/cred.txt"
$Waittime = "120" 
# ================================
Function Send-Update {
    Param($Message)
    $time = (get-date -Format "dd/MM/yy HH:mm")
    $status = "*" + $Message + "* - $time"
    /home/chris/send-telegram2 -Message $status -ParseMode Markdown
}
# ================================
Import-Module VMware.PowerCLI
$Encrypted = Get-Content "$Credfile" | ConvertTo-SecureString
$Credential = New-Object System.Management.Automation.PsCredential($Username, $Encrypted)
/home/chris/send-telegram2 -Message "VM Shutdown Sequence Started. VMs to be Shutdown:" -ParseMode Markdown
Connect-VIServer $ESXi -Credential $Credential
$PoweredVMs = (Get-VM).where{$_.PowerState -eq 'PoweredOn'}
Send-Update ($PoweredVMs.Name |Out-String)

ForEach ($VM in $PoweredVMs){
    Send-Update "Shutting down $VM"
    $VM | Shutdown-VMGuest -Confirm:$false > $null
    $looptime = $Waittime
    do {
        sleep 10
        $looptime = $looptime - 10
     } until ((Get-VM $VM).PowerState -eq 'PoweredOff' -or $looptime -eq 0)
     Send-Update "$VM is $((Get-VM $VM).PowerState)"
}

$KillVMs = (Get-VM).where{$_.PowerState -eq 'PoweredOn'}

If ($KillVMs){
    ForEach ($VM in $KillVMs){
        Send-Update "Killing $VM"
        Stop-VM -kill $VM -Confirm:$false
    }
}
Send-Update "Shutting down $ESXi"
Stop-VMHost $ESXi -Force -Confirm:$false
Disconnect-VIServer * -confirm:$false
Stop-Computer
{% endhighlight %}
</figure>

Don't forget to mark the script executable using `chmod +x shutdown.ps1`

## Calling the PowerShell Shutdown Script
Next we need to configure calling the above PowerShell script from the apcups daemon:

<figure><figcaption><b>Filename:</b> /etc/apcupsd/doshutdown</figcaption> 
{% highlight shell%}
#!/bin/sh
# This shell script if placed in /etc/apcupsd will be called by /etc/apcupsd/apccontrol when the UPS is running on batteries 
# and one of the limits expires (time, run, load), this event is generated to cause the machine to shutdown.
HOSTNAME=`hostname`
MSG="$HOSTNAME UPS $1 calling for controlled shut down"

now=$(date +"%d/%m/%y %H:%M")
now="$now" pwsh -file /home/chris/send-telegram2.ps1 -Message "<b>$now</b> - UPS <code>${1}</code> calling for control>
pwsh -file /home/chris/shutdown.ps1
${SHUTDOWN} -h now "apcupsd UPS ${1} initiated shutdown"
#
(
   echo "$MSG"
   echo " "
   /sbin/apcaccess status
) | $APCUPSD_MAIL -s "$MSG" $SYSADMIN
exit 0
{% endhighlight %}
</figure>
## Testing
Remarking out the shutdown commands and adjusting the timeout to 10 seconds per loop:

<table>
<tr>
<td style="height:50%;  width:50%;">
<a target="_blank" href="/images/esxi-rpi-ups-pt3/esxi-rpi-ups-pt3-01.jpg"><img style="display:block;" src="/images/esxi-rpi-ups-pt3/esxi-rpi-ups-pt3-01.jpg" alt="Telegram 1"/></a></td>
<td style="height:50%;  width:50%;">
<a target="_blank" href="/images/esxi-rpi-ups-pt3/esxi-rpi-ups-pt3-02.jpg"><img style="display:block;" src="/images/esxi-rpi-ups-pt3/esxi-rpi-ups-pt3-02.jpg" alt="Telegram 2"/></a></td>
</tr>
</table>
Nice!

Of course **PoweredOn** will read **PoweredOff** when VMs are actually shutdown and there won't be as much VM killing going on (second screenshot), but you get the idea.
## Conclusion and Wrap Up
There we have it: UPS initiated ESXi shutdown handled by a Raspberry Pi!

### Bonus Rounds
There are several other event called scripts contained in `/etc/apcupsd/` that can be modified to provide UPS visibility. For example:
<figure><figcaption><b>Filename:</b> /etc/apcupsd/onbattery</figcaption> 
{% highlight shell%}
#! /usr/bin/pwsh
$time = (get-date -Format "dd/MM/yy HH:mm")
$status = "*UPS Power failure - running on batteries!* - $time"
/home/chris/send-telegram2 -Message $status -ParseMode Markdown
{% endhighlight %}
</figure>
<figure><figcaption><b>Filename:</b> /etc/apcupsd/offbattery</figcaption> 
{% highlight shell%}
#! /usr/bin/pwsh
$time = (get-date -Format "dd/MM/yy HH:mm")
$status = "*UPS Power returned - running on mains power* - $time"
/home/chris/send-telegram2 -Message $status -ParseMode Markdown
{% endhighlight %}
</figure>
A full list of supported events can be found in [APCUPSD Manual - Customizing Event Handling](http://www.apcupsd.org/manual/manual.html#customizing-event-handling){:target="_blank"}.

This post was *a belated* part 3 of a multipart series. Find the other parts here:

- Part 1: [Hardware, Requirement, Software, Solution](https://polarclouds.co.uk/esxi-rpi-ups-pt1){:target="_blank"}
- Part 2: [Hardware Connectivity and Software Installation](https://polarclouds.co.uk/esxi-rpi-ups-pt2){:target="_blank"}
- Part 3: This part - Scripting for the win... or should that be for the failure?

-Chris
