# Active Directory Administration with Splunk Detection & Monitoring

A personal cybersecurity lab focused on **Active Directory administration and security monitoring using Splunk**.

The project covers building an Active Directory environment, managing domain-joined Windows machines, collecting Windows security telemetry, and creating detection rules in Splunk.

---

## Lab Environment

* **DC01** — Active Directory Domain Controller
* **WIN01** — Windows workstation
* **WIN02** — Windows workstation
* **Splunk** — Splunk Enterprise server running on Ubuntu
* **Domain:** `lab.local`
* Sysmon
* Splunk Universal Forwarder
* VirtualBox

### Architecture

```text
                         DC01
                  Active Directory
                      lab.local
                          |
              +-----------+-----------+
              |                       |
              v                       v
            WIN01                   WIN02
              |                       |
              | Windows Logs          | Windows Logs
              | Sysmon                | Sysmon
              | PowerShell Logs       | PowerShell Logs
              |                       |
              +-----------+-----------+
                          |
                          v
                Splunk Universal
                    Forwarder
                          |
                          v
                       Splunk
                       Server
                          |
                          v
              Parsing / Normalization
                          |
                          v
                Detection Rules
                          |
                          v
                       Alerts
```

---

## Project Areas

### Active Directory Administration

Built and managed the `lab.local` Active Directory environment using DC01.

This included:

* Domain and DNS configuration
* User and group management
* Organizational Units
* Group Policies
* Joining WIN01 and WIN02 to the domain

### Splunk

Configured Splunk Enterprise as the central SIEM for the lab.

This included:

* Splunk server configuration
* Splunk Universal Forwarder setup
* Windows Event Log collection
* Sysmon and PowerShell telemetry
* Log parsing and field extraction
* Data normalization
* Detection rules and alerts

### Security Testing

Used **Atomic Red Team** to generate controlled security-related activity within the isolated lab environment.

I also attempted to simulate **Credential Dumping followed by Pass-the-Hash** using Mimikatz to validate the monitoring and detection capabilities.

The generated activity was collected by Splunk and used to test the configured detection rules.

---

## Detection Workflow

```text
Windows Activity
       ↓
Windows Event Logs / Sysmon
       ↓
Splunk Universal Forwarder
       ↓
Splunk
       ↓
Parsing & Normalization
       ↓
Detection Rules
       ↓
Alerts
```

---

## What I Learned

This project gave me hands-on experience with:

* Active Directory administration
* Windows domain environments
* DNS
* Group Policy
* Windows Event Logs
* Sysmon
* PowerShell logging
* Splunk Enterprise
* Universal Forwarder
* Log collection
* Parsing
* Normalization
* Detection engineering
* Security monitoring and alerts

The project helped me understand the full process of collecting endpoint telemetry and turning it into useful security detections through a SIEM.

---

## Disclaimer

This project was created in an isolated virtual lab for educational and defensive cybersecurity purposes.

All testing was performed against systems within the lab environment.

