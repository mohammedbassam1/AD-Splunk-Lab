# Challenges 

This project was far from smooth but that was the best part. From debugging tricky errors to rethinking parts of the setup, pushing through the unexpected roadblocks is where the real learning happened.



#### Problem 1 — Sysmon and Security Logs Not Being Collected

there were an issue  with Sysmon and security logs not being collected:


**The issue was identified in the Splunk Universal Forwarder's main log file:**

`C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log`


i searched the log file using:
```

Get-Content "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" -Tail 300 |
Select-String -Pattern "WinEventLog|winevt|Sysmon|Security"

```

we found this 

```
ERROR ExecProcessor - WinEventLogChannel::init:
Init failed, unable to subscribe to Windows Event Log channel
'Microsoft-Windows-Sysmon/Operational':
errorCode=5
```


error code=5   means that  access is denied :   searched for it in google 


![Pasted image](<images/Pasted image.png>)






### Solution

The issue was caused by insufficient permissions for the Splunk Forwarder service account.

We resolved it by adding:

```
NT SERVICE\SplunkForwarder
```

to the Windows **Event Log Readers** group, giving Splunk permission to read the Sysmon Event Log.

```
Add-LocalGroupMember -Group "Event Log Readers" -Member "NT SERVICE\SplunkForwarder"
```


![Pasted image (2)](<images/Pasted image (2).png>)


after adding  to  event log readers group



![Pasted image (3)](<images/Pasted image (3).png>)



---


#### Problem 2 — Extracted Fields Were Not Available During Searches





when i was searching for extracted fields it wasnt showing and didnt give any values



![Pasted image (4)](<images/Pasted image (4).png>)


- **root cause of problem**
settings was in windows_security app   with private/app  permission  which made Splunk search app  unable to use it


- **Solution** :

creating a  config file to make it global
```
sudo mkdir -p /opt/splunk/etc/apps/windows_security/metadata
```

```

sudo bash -c 'cat << EOF > /opt/splunk/etc/apps/windows_security/metadata/local.meta
[]
access = read : [ * ], write : [ admin ]
export = system
EOF'


```


then give it permission: 

```
sudo chown -R splunk:splunk /opt/splunk/etc/apps/windows_security
```

then restart :
```
sudo /opt/splunk/bin/splunk restart

```



![Pasted image (5)](<images/Pasted image (5).png>)



#### another tip for parsing

u can add  some add-ons such as  to the splunk server

`Splunk Add-on for Microsoft WindowsSplunk `



---


### Problem 3 — Credential Dumping and Pass-the-Hash Simulation

During the attack testing phase, I attempted to simulate **Credential Dumping followed by a Pass-the-Hash attack** using **Atomic Red Team and Mimikatz** in the isolated Active Directory lab.

The first challenge was getting Mimikatz to download and execute. Although Windows Defender's real-time protection had been disabled, the system continued to prevent the tool from being downloaded or executed.

After investigating the issue, I found that additional Windows security controls were still active. I therefore reviewed the relevant security settings through **Group Policy** and the **Windows Security application** within the isolated lab environment.

After making these changes, I was able to successfully download and execute Mimikatz on the Windows machine.

However, Mimikatz did not successfully perform the intended credential-dumping/Pass-the-Hash operation. After troubleshooting the issue, I determined that the problem was related to the specific Mimikatz version/build being used rather than the Active Directory or Splunk configuration.

Although the attack itself was not fully successful, the testing activity was still valuable for validating the monitoring environment. The security-related activity generated during the testing was successfully collected by the Splunk Universal Forwarder, processed by Splunk, and detected by the configured detection rules.

#### Lesson Learned

This experience demonstrated that endpoint security controls can involve multiple layers beyond real-time antivirus protection. It also highlighted the importance of **tool version compatibility** when performing attack simulations.

Most importantly, the exercise reinforced the value of validating SIEM detection rules independently from the success or failure of a specific attack tool. The generated activity was successfully captured and detected by the monitoring infrastructure, demonstrating that the Splunk detection pipeline was functioning as intended.




![Pasted image (6)](<images/Pasted image (6).png>)




![Pasted image (7)](<images/Pasted image (7).png>)





![Pasted image (8)](<images/Pasted image (8).png>)



![Pasted image (9)](<images/Pasted image (9).png>)
