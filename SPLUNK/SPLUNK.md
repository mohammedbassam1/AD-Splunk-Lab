# Splunk Complete Guide

    
Hostname: 
Splunk OS: Ubuntu
IP: 192.168.56.66 

Role: SIEM / Log Management




## installing Splunk Server



#### 1 downloading the package 


https://www.splunk.com/en_us/download/splunk-enterprise.html  


create an account then install the package for your OS in our case its Ubuntu  `deb` 


after that open a terminal where u downloaded the package and run 


```

sudo apt install ./splunk-10.4.2-33c3bf42cd73-linux-amd64.deb
```

then run it 
```
sudo /opt/splunk/bin/splunk start --accept-license

```

while installing  it will   ask u to create a  username and password for administrator 

![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-25-48.png>)


##### 2- Accessing Splunk from web



http://localhost:8000 and enter the username and password  u created


![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-27-09.png>)







#### 3-Enable Receiving Port


in Splunk web:


**Settings → Forwarding and receiving → Configure receiving → Add new**


specify the port u gonna receive logs on  by forwarders



Port: 9997 which is the default port for Splunk

![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-28-38.png>)




- make sure ur  Host is listening  with this command

```
sudo ss -lntp | grep 9997

```

![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-29-44.png>)




### 4- enabling Forwarder Monitoring

- Logged in to **Splunk Web**.
- Navigated to **Settings → Monitoring Console**.
- Opened **Forwarders → Forwarder Monitoring**.
- Enabled **Forwarder Monitoring**.
- This allows us to monitor the **Universal Forwarders** connected to the Splunk server.
- Connected devices: **WS01, WS02, and SRV01**.



![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-31-06.png>)










---- 


#### Installing Forwarders on Agents


#### 1. Download Splunk Universal Forwarder

Download the **Splunk Universal Forwarder** from the official Splunk website:

[Splunk Universal Forwarder Download](https://www.splunk.com/en_us/download/universal-forwarder.html)

Select the appropriate **Windows version**. In our lab, we use **Windows 11 and Windows Server**.

After downloading:

1. Start the installer from the download location.


![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-34-17.png>)



    
2. Create the **username and password** required for the Universal Forwarder.


![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-35-59.png>)



3. Configure the **Receiving Indexer** during the installation  and leave Deployment server empty 

![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-37-04.png>)

4- Complete the installation and ensure the **SplunkForwarder** service is running.



make sure its downloaded  

open a powershell :

```
Get-Service SplunkForwarder

```


```
Test-NetConnection 192.168.56.66 -Port 9997
```


if its


```
TcpTestSucceeded : True
```


then its sending  to the splunk server


![Screenshot](<Splunk images/Screenshot From 2026-08-12 20-40-07.png>)



**you  will do the same steps  on all the windows machines** 



---



## Configuring forwarders log Collection


in windows machines  we wanna forward from :


**1- Navigate to the `inputs.conf` Configuration Path**

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\
```

**2- Then open or create:**

```
inputs.conf
```



**3- add Logs  we want to forward to splunk :** 

```
[WinEventLog://Security]
disabled = 0
index = wineventlog
renderXml = true

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = wineventlog
renderXml = true

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = wineventlog
renderXml = true
```


![Screenshot](<Splunk images/Screenshot From 2026-08-22 20-20-14.png>)




**4- Restart Splunk-forwarder**  

```

Restart-Service SplunkForwarder
```










#### In Splunk  server



go to settings  >  indexes > new Index

![Screenshot](<Splunk images/Screenshot From 2026-08-22 21-26-14.png>)



add the index we put in input.conf for forwarders  which is  `wineventlog`


![Screenshot](<Splunk images/Screenshot From 2026-08-22 21-26-47.png>)




## Testing 



we tested if  the  logs we added is  collected using this query


`index=wineventlog | stats count by host sourcetype`


![Screenshot](<Splunk images/Screenshot From 2026-08-22 21-23-10.png>)






---

# Logs parsing and normalization



![Screenshot](<Splunk images/Screenshot From 2026-08-25 21-00-15.png>)

**The raw XML events are difficult to search and filter. Therefore, parsing is required to extract individual fields and make the logs easier to analyze and use for detection rules.**





## Parsing

we want to turn raw events  data to  fields



##### starting with Sysmon

on the Splunk server

- 1-  make a directory for local settings

```
sudo mkdir -p /opt/splunk/etc/apps/windows_security/local
```


we dont wanna edit the default  splunk files  . we want  to configure our own settings to manage it easier



- 2- creating  properties file:

config file for Splunk that we use to  tell Splunk : when u receive this type of file apply these configurations 


```
sudo nano /opt/splunk/etc/apps/windows_security/local/props.conf
```

![Screenshot](<Splunk images/Screenshot From 2026-08-26 03-50-56.png>)


put in side it :


```
[XmlWinEventLog:Microsoft-Windows-Sysmon/Operational]
REPORT-sysmon_xml = sysmon_xml_fields
```


first line  declares the source type

2nd line : field extraction file.



3-  creating transforms.conf file:

this file  configure   the fields extraction 

![Screenshot](<Splunk images/Screenshot From 2026-08-26 03-51-13.png>)



- to make sure config is working  properly 

```
sudo /opt/splunk/bin/splunk btool transforms list sysmon_xml_fields --debug
```

- it should shows the files we created if it worked 


![Screenshot](<Splunk images/Screenshot From 2026-08-26 03-54-29.png>)




```
sudo /opt/splunk/bin/splunk btool props list "XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" --debug

```

we check the config of prop.conf  if  it worked or not 


**==btool== is used  to inspect and validate Splunk effective configsand identify  which config file  each setting coming from**




4-  restart splunk

```
sudo /opt/splunk/bin/splunk restart
```



![Screenshot](<Splunk images/Screenshot From 2026-08-26 04-04-26.png>)




as u can see  fields  are being extracted automatically   by splunk from the sysmon xml events and can be searched for  , filtered  




#### Powershell and security  

we follow  the same steps  for sysmon parsing


![Screenshot](<Splunk images/Screenshot From 2026-08-26 04-15-55.png>)

![Screenshot](<Splunk images/Screenshot From 2026-08-26 04-16-26.png>)





---

## Normalization 


we want to make the name of the fields Normalized  between all sources  so  its easier for analyst to search , filter . 

```
 sudo nano /opt/splunk/etc/apps/windows_security/local/props.conf
```

ADD FIELDALIAS in the file:

#### 1-  we used FIELDALIAS 


 to group similar field names if there any difference from log sources
 

![Screenshot](<Splunk images/Screenshot From 2026-08-26 08-59-59.png>)


![Screenshot](<Splunk images/Screenshot From 2026-08-26 09-00-28.png>)



#### - 2- (Eventtypes)


  created a new config file 
  
```
sudo nano /opt/splunk/etc/apps/windows_security/local/eventtypes.conf

```




![Screenshot](<Splunk images/Screenshot From 2026-08-26 09-02-32.png>)

useful for: 

1- group events with similar function 

2-   easier searches  

3-  to use tags


```
index=wineventlog eventtype=winevent_process_create
```


![Screenshot](<Splunk images/Screenshot From 2026-08-26 09-06-47.png>)

1 search included  both Sysmon event ID : 1 and security logs event id : 4688




#### 3- tags


it declares  the category for  the eventtypes


useful for:

1-  grouping  eventypes for different  systems based on categories: 

example: authentication logs  from  linux,windows,Firewall, AWS

all can be searched  by using  `tag=authentication`


2- Hierarchical Tagging) :

using more than 1 tag on the same event 

example:  `tag = authentication , tag=failure or tag=Success`




- we added them   by creating a new config file


```

 sudo nano /opt/splunk/etc/apps/windows_security/local/tags.conf

```

![Screenshot](<Splunk images/Screenshot From 2026-08-26 09-15-42.png>)




![Screenshot](<Splunk images/Screenshot From 2026-08-26 09-18-03.png>)

as u can see it  shows  authentication logs  both faiil and success


---

#  Detection Rules - Alerts 


starts with search query for specific rule




## Brute force attempts


```
index=wineventlog tag=authentication tag=failure
| stats count as Failed_Attempts by host , src_ip, user
| where Failed_Attempts > 5
```


```
| stats count as Failed_Attempts by host , src_ip, user
```

this line counts the number of failed attempts events and puts it   in Failed_Attempts column 

and collects host - src_ip - user


```
| where Failed_Attempts > 5

```


this is the condition  to trigger  only when  the failed attempts  is more than 5


![Screenshot](<Splunk images/Screenshot From 2026-08-26 22-22-24.png>)



#### to make the alert :  

**2. Save As Alert**

- Navigate to the top right and select **Save As** > **Alert**.
    
- **Title:** `SOC - Suspicious Command / Hack Tool Execution` (or `SOC - Brute Force Attack Detected`)
    

**3. Schedule Settings**

- **Alert type:** `Scheduled`
    
- **Cron Schedule:** `*/5 * * * *` (Runs every 5 minutes)
    
- **Time Range:** `Last 5 minutes` (Only scans new events per run to prevent duplicate alerts)
    

**4. Trigger Conditions**

- **Trigger condition:** `Number of Results` > `is greater than` > `0` (Fires immediately upon detecting any matching event)
    
- **Trigger:** `Once`
    

**5. Alert Actions & Permissions**

- **Trigger Actions:** `Add to Triggered Alerts`
    
- **Severity:** `High` (or `Medium`)
    
- **Expires:** `24 hours` (or `7 days`)
    
- **Permissions:** `Shared in App` (Allows all SOC analysts in the workspace to view and investigate the alert)
    
- Click **Save**.



## Suspicious parent process spawned

```
index=wineventlog tag=process tag=report
| eval proc=lower(coalesce(process_name, Image)), parent=lower(coalesce(parent_process_name, ParentImage)), full_path=lower(Image)
| search NOT (full_path="*splunk*" OR parent="*splunk*" OR proc="*splunk*")
| where match(proc, "(\\bpowershell|\bcmd|\bwhoami|\bnet1?\.exe|\bnltest|\bvssadmin|\bmimikatz)")
| stats count by _time, host, user, ParentImage, Image, CommandLine
| sort - _time

```


- **`| eval proc=lower(...), parent=lower(...), full_path=lower(Image)`**
    
    - **Function:** Normalizes and converts all process names, parent names, and full binary execution paths to lowercase to enable case-insensitive filtering.
        
    - **CIM Compatibility:** Uses `coalesce` to dynamically pick either CIM field names (`process_name`, `parent_process_name`) or native Windows fields (`Image`, `ParentImage`).
        
- **`| search NOT (full_path="*splunk*" OR parent="*splunk*" OR proc="*splunk*")`**
    
    - **Function (Alert Tuning / Whitelisting):** Eliminates false positives by dropping any event originating from Splunk Enterprise, Splunk Universal Forwarder, or background operational scripts running from `C:\Program Files\Splunk...`.
        
- **`| where match(proc, "(\\bpowershell|\bcmd|\bwhoami|\bnet1?\.exe|\bnltest|\bvssadmin|\bmimikatz)")`**
    
    - **Function (Detection Logic):** Flags the execution of native reconnaissance utilities (LOLBins) and offensive attack tools.
        
    - **Regex `\b`:** Enforces word boundaries to match exact tool names and prevent accidental partial string matches.
        
- **`| stats count by _time, host, user, ParentImage, Image, CommandLine`**
    
    - **Function:** Aggregates and displays the investigation baseline: execution timestamp (`_time`), targeted asset (`host`), active account (`user`), parent binary (`ParentImage`), executed binary (`Image`), and full arguments (`CommandLine`).
        
- **`| sort - _time`**
    
    - **Function:** Sorts output in reverse chronological order (newest activity first).
        

**Alert Configuration:**

- **Title:** `SOC - Suspicious Command / Hack Tool Execution`
    
- **Alert Type:** `Scheduled`
    
- **Cron Schedule:** `*/5 * * * *`
    
- **Time Range:** `Last 5 minutes`
    
- **Trigger Condition:** `Number of Results` > `is greater than` > `0`
    
- **Trigger Actions:** `Add to Triggered Alerts` (Severity: `High`, Permissions: `Shared in App`)



![Screenshot](<Splunk images/Screenshot From 2026-08-26 23-24-44.png>)



I made another 7 alerts wont get to all of them  but u get the idea on how to make a  detection rule and  how to alert it




![Screenshot](<Splunk images/Screenshot From 2026-08-26 23-25-38.png>)



## Testing


i created a user account and added it  to administrator group to test user creation rule  



![Screenshot](<Splunk images/Screenshot From 2026-08-26 23-29-07.png>)


Alert is triggered 


![Screenshot](<Splunk images/Screenshot From 2026-08-26 23-29-27.png>)


![Screenshot](<Splunk images/Screenshot From 2026-08-26 23-30-53.png>)







---

# Dashboards

A Security Operations Center (SOC) dashboard transforms raw, high-volume event logs into actionable visual intelligence for real-time threat monitoring and fast investigation.

**Key Benefits of a SOC Dashboard:**

- **Real-Time Threat Visibility:** Aggregates critical security metrics, active alerts, and host activity into a single pane of glass without running manual queries repeatedly.
    
- **Accelerated Triage & MTTR:** Reduces Mean Time to Respond (MTTR) by clearly highlighting compromised assets, targeted users, and attack types for rapid decision-making.
    
- **Trend & Anomaly Detection:** Uses time-series charts to spot sudden spikes in malicious activity, such as brute-force logon attempts or mass process executions.
    
- **Operational Efficiency:** Streamlines daily analyst workflows using interactive dropdown filters (Time Range, Host, User, Severity) to quickly isolate incidents.
    
- **Executive & Compliance Reporting:** Delivers clear, high-level summaries of organizational security posture and incident trends for management without complex query syntax




u create it using XML : ngl i asked ai to do it for me since its iam not going to use for this lab but its really important to know how to make  one 



![Screenshot](<Splunk images/Screenshot From 2026-08-27 04-00-49.png>)

![Screenshot](<Splunk images/Screenshot From 2026-08-27 04-08-18.png>)
