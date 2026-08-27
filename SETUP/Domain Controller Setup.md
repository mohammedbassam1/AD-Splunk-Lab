

# After  installing Windows 2022 server 


## WE  Installed Active Directory Domain Services

- 1- Opened **Server Manager**.

- 2- Selected **Add Roles and Features**.

- 3- Selected **Active Directory Domain Services (AD DS)**.

- 4- Installed the required AD DS role and management tools.

- 5- After installation, promoted the server to a **Domain Controller**.

	Created the new domain:

```
lab.local
```

- 6- Configured **DNS** as part of the domain controller setup.


- 7-  Restarted the server to complete the configuration.

![Image](<SETUP DC images/Screenshot From 2026-08-12 22-42-455555.png>)





## enabling Powershell logging for Domain controller


 in power shell:

- enabling  Powershell Script Block Logging

```
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force | Out-Null Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" ` -Name EnableScriptBlockLogging -Value 1 -Type DWord


```


-  enabling Powershell   Module  Logging


```
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -Force | Out-Null Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging" ` -Name EnableModuleLogging -Value 1 -Type DWord
```





## installing sysmon  


we did the same steps for WIndows machines





