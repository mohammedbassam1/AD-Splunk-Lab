# WINDOW MACHINES SETUP

# After  installing Windows 11 PRO on both virtual machines  


**make sure that the Windows version is Pro because you cant join a domain using Home Version** 





## Configuring  the machines to use  domain controller as  DNS  

- we configured  WIN01  IP And  the domain controller  as Preferred DNS server :

![Image](<setup images/Screenshot From 2026-08-12 22-23-30.png>)


 - we did the same thing for  WIN02 


![WIN02 DNS](<setup images/Screenshot From 2026-08-12 22-26-48.png>)





## Joining  the machines to  domain



#### ON BOTH MACHINES: 


```
 Press  WIN +R   then type sysdm.cpl

 ###to open  System Proprties windows
``` 


- after that  click  change >   Domain   lab.local


![Domain Join](<setup images/Screenshot From 2026-08-12 22-35-14.png>)



- Windows then prompted for **Domain Administrator credentials**.
- Entered the **Domain Administrator username and password**.
- Restarted the machine to complete the domain join.






## Install sysmon




https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon  from here  u download sysmon 


then u  download sysmon config file 

https://github.com/SwiftOnSecurity/sysmon-config  and put it inside Sysmon folder




![Sysmon Files](<setup images/Screenshot From 2026-08-22 19-20-39.png>)



#### Start Sysmon


in cmd: 

```
Sysmon.exe -accepteula -i sysmonconfig-export.xml
```


![Start Sysmon](<setup images/Screenshot From 2026-08-22 19-21-33.png>)




#### Verify its working 


IN  CMD

```

sc query sysmon

```



![Verify Sysmon](<setup images/Screenshot From 2026-08-22 19-25-36.png>)





