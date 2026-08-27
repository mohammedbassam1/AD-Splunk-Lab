#Active Directory Administration











### Creating Organizational Units (OUs)

First of all i created AN Organizational Units to organize domain-joined computers based on their roles.



![Screenshot](<Active directory images/Screenshot From 2026-08-12 23-11-16.png>)




The following OUs were created:

- Workstations — for WS01 and WS02. 

- Servers — for SRV01.

- Users: for users

- Groups: 


![Screenshot](<Active directory images/Screenshot From 2026-08-12 23-18-54.png>)



This structure allows different Group Policies (GPOs) and administrative permissions to be applied based on the type of device.







## Creating Users


inside AD-users  


Right click > New > User   > provide user info and setup the password



![Screenshot](<Active directory images/Screenshot From 2026-08-12 23-25-26.png>)



![Screenshot](<Active directory images/Screenshot From 2026-08-12 23-27-28.png>)




Users created : sarah.user ,  john.user , IT admin


## Creating Groups



-inside OU groups unit


  Right click on  AD-Groups    > new > group  >  name the group and add it 


![Screenshot](<Active directory images/Screenshot From 2026-08-12 23-31-48.png>)

**Security Group**  :

we used  security and global because  we want admins to have to access :

- Folder
- Files
- RDP
- Applications
- Printers

**Global Group**  

to add multiple users that have the same  role





## Adding  user to the group



right click on the user > Properties > click on member of   > add    > IT-Admins


![Screenshot](<Active directory images/Screenshot From 2026-08-12 23-50-21.png>)






## Adding group Policies




## workstation Security Policy


#### Audit Policy



- Logon failure and success on workstations


 on server manager    click  : tools >  group policy management 


![Screenshot](<Active directory images/Screenshot From 2026-08-20 16-42-23.png>)






Click on Workstations OU  >  Create a GPO in the domain  and link it here


![Screenshot](<Active directory images/Screenshot From 2026-08-20 16-43-27.png>)



name the policy then  then right click on and   click on Edit


![Screenshot](<Active directory images/Screenshot From 2026-08-20 16-44-55.png>)




open   policies > windows settings >  Security settings  > Advanced Audit Policy Configuration >   Audit Policies


set Audit Credential validation to    success . failure



![Screenshot](<Active directory images/Screenshot From 2026-08-20 16-48-01.png>)



#### User Right Assignment 


- allowing local login from domain accounts only


-  denying  guest login



 go to windows settings > security settings > local  policies >  User Rights Assignment  


click on : Allow log on locally




![Screenshot](<Active directory images/Screenshot From 2026-08-20 16-58-55.png>)


check  the define these policy settings box



add the users / groups 


![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-00-56.png>)




Now click on  Deny log on locally 


 and add guests group 

![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-01-55.png>)




#### Security Options


- disabling  local  users and guest locally 


go to Windows settings >  Security settings  >  local policies > Security Options


click on :


`Accounts: Administrator account status ` : Disabled





![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-07-13.png>)






click on  

`Accounts : Guest account  status Properties`  : Disabled 


![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-08-48.png>)







-   Screen auto-lock workstations when inactive 


from security options  


click on : `Interactive logon: Machine inactivity limit `

put it to : 900 sec  ( 15 min ) 


![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-14-56.png>)





-  UAC elevation  : to  prevent users to download as administrator  for untrusted app


from security options

click on :   `User Account Control: Behavior of the elevation prompt for standard users`




![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-20-57.png>)







#### Windows Defender Firewall  management


From windows settings > security settings   


click on Windows Defender Firewall with Advanced Security 


for all profiles: 

 firewall state to : `ON`   

inbound connetions : `block`

outbound connection : `allow`


![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-25-14.png>)





###### Creating inbound rules 




- **allowing WMI**  


WMI: allows system administrators to query system information and automate administrative tasks both locally and remotely.


**Common Use Cases:**

- **System Inventory:** Retrieving hardware and software details (e.g., installed software, CPU temperature, hard drive serial numbers, RAM usage).
    
- **Remote Control:** Starting or stopping Windows services, running remote scripts, and applying Group Policies (`gpupdate`).
    
- **Event Monitoring:** Triggering automated actions when specific events occur (e.g., when disk space drops below 10%).






now in Windows Defender Firewall with Advanced Security  

click on inbound rules  

right click   > New Rule    > choose Predefined  and choose   `Windows Management Instrumentation (WMI)`  > next    > click on allow the connection 



![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-36-09.png>)




- allow **Remote Scheduled Task management**


follow the same steps     and choose  predefined   `Remote Scheduled Task Management `




WMI  And remote Scheduled Task management are really important services  for administration but   it can be used by attackers for lateral movement  , persistence so we  gonna restrict it use only from DC  



right click on rules   > properties  > scope  > remote  address >   put the ip  for the administrator Machine 



![Screenshot](<Active directory images/Screenshot From 2026-08-20 17-47-45.png>)






## Powershell logging   Policy


we added another policy for workstations  to  Log PowerShell   Script Block logging  , module logging . its very useful for    to detect powershell commands



- 1- go to  : Computer Configuration → Policies → Administrative Templates → Windows Components → Windows PowerShell 

- 2-   enable: module logging , Powershell Script Block logging

![Screenshot](<Active directory images/Screenshot From 2026-08-22 19-55-25.png>)



#### Another audit policy i forgot to add 


in the same windows security policy go to   policies > windows settings > security settings > local policies > audit policy    


-  audit account logon events  both success and failure 
- audit account management  both success and failure 
-  audit logon events  both success and failure 
- audit privilege use both success and failure 

all these are important evidence in security channel 

![Pasted image](<Active directory images/Pasted image.png>)


## Testing 


to enforce  the group policies  : on workstations 

in cmd:

```
gpupdate / force
```



![Pasted image (2)](<Active directory images/Pasted image (2).png>)



from DC01  Remote Scheduled tasks is working 


![Pasted image (3)](<Active directory images/Pasted image (3).png>)





also   the normal user cant  open as administrator

![Pasted image (4)](<Active directory images/Pasted image (4).png>)



also  credential validation policy we allowed is working


![Pasted image (5)](<Active directory images/Pasted image (5).png>)



