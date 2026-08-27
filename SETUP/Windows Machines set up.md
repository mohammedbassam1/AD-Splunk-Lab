# WINDOW MACHINES SETUP

# After  installing Windows 11 PRO on both virtual machines  


**make sure that the Windows version is Pro because you cant join a domain using Home Version** 





## Configuring  the machines to use  domain controller as  DNS  

- we configured  WIN01  IP And  the domain controller  as Preferred DNS server :

![[Pasted image 20260812222339.png]]


 - we did the same thing for  WIN02 


![[Pasted image 20260812222654.png]]





## Joining  the machines to  domain



#### ON BOTH MACHINES: 


```
 Press  WIN +R   then type sysdm.cpl

 ###to open  System Proprties windows
``` 


- after that  click  change >   Domain   lab.local


![[Pasted image 20260812223514.png]]



- Windows then prompted for **Domain Administrator credentials**.
- Entered the **Domain Administrator username and password**.
- Restarted the machine to complete the domain join.






## Install sysmon




https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon  from here  u download sysmon 


then u  download sysmon config file 

https://github.com/SwiftOnSecurity/sysmon-config  and put it inside Sysmon folder




![[Pasted image 20260822192043.png]]



#### Start Sysmon


in cmd: 

```
Sysmon.exe -accepteula -i sysmonconfig-export.xml
```


![[Pasted image 20260822192158.png]]




#### Verify its working 


IN  CMD

```

sc query sysmon

```



![[Pasted image 20260822192548.png]]





