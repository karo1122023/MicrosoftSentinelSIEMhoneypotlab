<h1>Mircosoft Azure Sentinel Home Lab</h1>


<h2>Description</h2>
In this home lab, I setup Microsoft Azure Sentinel (SIEM) and connect it to a live virtual machine acting as a honey pot. We will observe live attacks (RDP Brute Force) from all around the world. We will use a custom PowerShell script to look up the attackers Geolocation information and plot it on the Microsoft Azure Sentinel Map!

<br />


<h2>Tools and Platforms Used</h2>

- <b>PowerShell</b> 
- <b>Microsoft Azure</b>
- <b>Microsoft Remote Desktop</b> 

<h2>Environments Used </h2>

- <b>Windows 10 Pro, version 22H2 - x64 Gen2</b> 

<h2>Home Lab walk-through:</h2>

I will create a Windows 10 virtual machine on Microsoft Azure and name it honeypot-vm: <br/>
<img src="https://i.imgur.com/9NTsvXA.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
I will then create a new open firewall (DANGER_ANY_IN) and create an inbound rule that will allow all traffic into the virtual machine from the internet: <br/>
<img src="https://i.imgur.com/eFkX3ba.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Next, I will navigate to Log Analyatics Workspace and create a Log Analytics Workspace called law-honeypot1. This is where all the audit logs from the Windows event logs and security logs will be stored. It will contain geographic details of where the attackers are coming from: <br/>
<img src="https://i.imgur.com/fIJkJUP.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Now I will head over to Microsoft Defender for Cloud > envrionment settings > azure subcription > expand down to click on law-honeypot1: <br/>
<img src="https://i.imgur.com/FGmFRCL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
I will turn on the servers in order to be able to do the data collection. Then I will navigate to the data collection tab and turn on all events, which will enable the ability for logs to be gathered from the Windows virtual machine straight into the Logs Analytics Workspace:  <br/>
<img src="https://i.imgur.com/6TZ3CLG.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/6P8WFGT.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Next, I will head back over to Log Analytics Workspace and have it connect to the virtual machine (honeypot-vm): <br/>
<img src="https://i.imgur.com/RmCTrgi.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Now I will set up Microsoft Azure Sentinel, which is the SIEM I will use to visualize the attack data. I will click microsoft sentinel > click create microsoft sentinel > click law-honeypot1 > click add:  <br/>
<img src="https://i.imgur.com/BQfLVSo.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/dtVYOv0.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Now it's time to navigate to the virtual machine. I will then click on honeypot-vm > click on the public IP address and copy it: <br/>
<img src="https://i.imgur.com/P3IFmqZ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Because I have a MAC, I need to download Microsoft Remote Desktop. After it's been downloaded and launched, I will paste in the IP address of the honeypot-vm I copied into add and then click add > double click on the virtual machine with the IP address > enter in user credentials set up for the honeypot-vm:  <br/>
<img src="https://i.imgur.com/lF0Q6t5.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
On the actual virtual machine, I will turn off the firewall so that people on the internet can discover it faster. I will click windows defender firewall > go to advanced settings > windows defender firewall properties > turn firewall state off in domain, private profile, public profile: <br/>
<img src="https://i.imgur.com/5yxQ47D.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
By clicking on event viewer to view the logs > windows logs > security, I can see all of the security events on this virtual machine. And specifically, event ID 4625, which says audit failure. This specific log represents all the failures of attackers trying to log into the virtual machine through the remote desktop. These audit logs even show me all of the IP addresses people are using to try to log into my virtual machine. The IP addresses from these audit logs will be taken using a powershell script and uploaded into an IP geolocation API (ipgeolocation.io). The powershell script then takes the API's results and outputs it into a file, which I will then upload into Log Analytics Workspace:  <br/>
<img src="https://i.imgur.com/JOdySKo.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/J3jH9Tt.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
I will now run a powershell script within the virtual machine on Windows Powershell ISE. [ https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1 ]. This link provides the script used to ingest the audit logs from my virtual machine into the geolocation API. In order for the script to be able to return the specific geographic information of the attackers, I need to create an account with ipgeolocation.io to receive my own API key which I will paste into the powershell script in the API key location. You can see the results of running the powershell script below. The script also creates a geodata logon log file of the results named failed_rdp: <br/>
<img src="https://i.imgur.com/l0xqGUP.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/3R9EhhF.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
I will copy the contents of the failed_rdp file and save it on a file on my personal computer. Next, I will navigate to Log Analytics Workspace on Microsoft Azure to create a custom log that will contain the geodata. I will click on tables under settings > click on create new custom log MMA based > and upload the failed_rdp file:  <br/>
<img src="https://i.imgur.com/pvXfkAF.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/w8sIcwc.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
After navigating to the logs tab, I will paste the code contained in the first picture below into the query. This code extracts the latitude and longitude, username, source host (IP address of host that attempted to login), etc values from the results; it parses out the log data. I will then run the query:  <br/>
<img src="https://i.imgur.com/b01NXOa.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/kz3nJio.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/1mEnHET.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Next, I will click on workbooks to set up the geomap > add workbook > edit > remove the default widgets > add query > paste previous code used into the query > run the query > name the workbook (FAILED RDP WORLD_MAP). The code once again parses out the data aggregated from the failed_rdp log file that contains all the geodata: <br/>
<img src="https://i.imgur.com/NI1KS2Y.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Finally, I will click visualize and select map > under map settings > metrics > select label as label to get the IPs of the countries. I have successfully used the Microsoft Azure Sentinel SIEM to create a world map containing attacker geodata from log files allowing me to see all of the countries the attacks were coming from on the map: <br/>
<img src="https://i.imgur.com/ioQ5cYD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />

<h2>Conclusions:</h2>
This was my first home lab, and this home lab taught me how to set up a virtual machine in the cloud, use Microsoft Azure SIEM to parse out and view data from logs, how to analyze log files, how Powershell works, and much more!

</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
