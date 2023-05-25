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
 -Great Q, the 

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
