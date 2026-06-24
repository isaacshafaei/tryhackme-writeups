# Important Windows Event IDs for SOC Analysis

Windows has thousands of events. These are the **most important practical IDs** for SOC monitoring, threat hunting, and incident response. Some events appear only when the relevant audit policy or logging feature is enabled. ([Microsoft Learn][1])

## 1. Authentication and Logon

| Event ID | Meaning                                               |
| -------: | ----------------------------------------------------- |
| **4624** | Successful logon                                      |
| **4625** | Failed logon                                          |
| **4634** | Account logged off                                    |
| **4647** | User initiated logoff                                 |
| **4648** | Logon using explicit credentials, such as `runas`     |
| **4672** | Special/admin privileges assigned to a logon          |
| **4776** | Domain Controller validated credentials, usually NTLM |
| **4778** | RDP session reconnected                               |
| **4779** | RDP session disconnected                              |
| **4800** | Workstation locked                                    |
| **4801** | Workstation unlocked                                  |
| **4825** | RDP access denied                                     |

Important **4624/4625 Logon Types**:

|   Type | Meaning                                   |
| -----: | ----------------------------------------- |
|  **2** | Local interactive login                   |
|  **3** | Network login, such as SMB                |
|  **4** | Batch job                                 |
|  **5** | Service login                             |
|  **7** | Workstation unlock                        |
|  **8** | Network cleartext credentials             |
|  **9** | New credentials, such as `runas /netonly` |
| **10** | Remote Interactive/RDP                    |
| **11** | Cached domain login                       |

A single 4625 is normally insufficient for an alert. Look for repeated failures, unusual source IPs, followed by a successful 4624, or attempts against many accounts. ([Microsoft Learn][2])

## 2. Kerberos and Active Directory Authentication

| Event ID | Meaning                                       |
| -------: | --------------------------------------------- |
| **4768** | Kerberos TGT requested                        |
| **4769** | Kerberos service ticket requested             |
| **4770** | Kerberos service ticket renewed               |
| **4771** | Kerberos pre-authentication failed            |
| **4772** | Kerberos authentication ticket request failed |
| **4740** | User account locked out                       |
| **4767** | User account unlocked                         |

Investigate 4769 for unusual service names, encryption types, high ticket volume, or access to sensitive services such as `cifs`, `ldap`, and `host`. Event 4768 is generated on Domain Controllers when a TGT is issued. ([Microsoft Learn][3])

## 3. Process and Command Execution

| Event ID | Meaning                                    |
| -------: | ------------------------------------------ |
| **4688** | New process created                        |
| **4689** | Process terminated                         |
| **4697** | Service installed through the Security log |
| **7045** | New Windows service installed              |
| **7040** | Service startup type changed               |
| **7036** | Service started or stopped                 |

For **4688**, examine:

* Process name and full path
* Command line
* Parent process
* User and integrity level
* Encoded PowerShell or suspicious LOLBins
* Execution from Temp, Downloads, AppData, or public folders

Command-line auditing must be enabled to obtain the most useful 4688 details. ([Microsoft Learn][4])

## 4. PowerShell

Log: `Microsoft-Windows-PowerShell/Operational`

| Event ID | Meaning                            |
| -------: | ---------------------------------- |
| **4103** | PowerShell module/cmdlet execution |
| **4104** | PowerShell script-block content    |
| **4105** | Script-block execution started     |
| **4106** | Script-block execution completed   |
|  **400** | PowerShell engine started          |
|  **403** | PowerShell engine stopped          |

**4104 is the most valuable PowerShell event** because it can contain the actual executed script. Look for Base64, `IEX`, `DownloadString`, reflection, credential dumping, AMSI bypasses and hidden execution. ([Microsoft Learn][4])

## 5. User and Account Management

| Event ID | Meaning                   |
| -------: | ------------------------- |
| **4720** | User account created      |
| **4722** | User account enabled      |
| **4723** | Password change attempted |
| **4724** | Password reset attempted  |
| **4725** | User account disabled     |
| **4726** | User account deleted      |
| **4738** | User account changed      |
| **4781** | Account name changed      |
| **4741** | Computer account created  |
| **4742** | Computer account changed  |
| **4743** | Computer account deleted  |

Alert especially on unexpected **4720**, **4722**, **4724**, **4726**, and **4781**. ([Microsoft Learn][5])

## 6. Group and Privilege Changes

| Event ID | Meaning                                      |
| -------: | -------------------------------------------- |
| **4727** | Global security group created                |
| **4728** | Member added to global security group        |
| **4729** | Member removed from global security group    |
| **4731** | Local security group created                 |
| **4732** | Member added to local security group         |
| **4733** | Member removed from local security group     |
| **4735** | Local security group changed                 |
| **4737** | Global security group changed                |
| **4754** | Universal security group created             |
| **4755** | Universal security group changed             |
| **4756** | Member added to universal security group     |
| **4757** | Member removed from universal security group |

High-priority examples include additions to:

* Domain Admins
* Enterprise Admins
* Administrators
* Remote Desktop Users
* Backup Operators
* Account Operators

## 7. Scheduled Tasks and Persistence

| Event ID | Meaning                 |
| -------: | ----------------------- |
| **4698** | Scheduled task created  |
| **4699** | Scheduled task deleted  |
| **4700** | Scheduled task enabled  |
| **4701** | Scheduled task disabled |
| **4702** | Scheduled task updated  |

Inspect the task action, executable path, command-line arguments, creator and trigger. Tasks launching PowerShell, `cmd.exe`, scripts or binaries from writable directories are especially suspicious.

## 8. File, Registry and Object Access

| Event ID | Meaning                                |
| -------: | -------------------------------------- |
| **4656** | Handle requested for an object         |
| **4657** | Registry value modified                |
| **4660** | Object deleted                         |
| **4663** | File, registry or object accessed      |
| **4670** | Object permissions changed             |
| **4907** | Auditing settings on an object changed |
| **6416** | New external device recognized         |

Events such as **4663** require a suitable SACL on the monitored object. ([Microsoft Learn][6])

## 9. Network Shares and Lateral Movement

| Event ID | Meaning                             |
| -------: | ----------------------------------- |
| **5140** | Network share accessed              |
| **5142** | Network share created               |
| **5143** | Network share modified              |
| **5144** | Network share deleted               |
| **5145** | Detailed network-share access check |

Monitor access to:

* `ADMIN$`
* `C$`
* `IPC$`
* `SYSVOL`
* `NETLOGON`
* Sensitive file servers

Correlate these with **4624 Logon Type 3**, **4648**, **4769** and process-creation events.

## 10. Active Directory Object Changes

These are mainly generated on Domain Controllers.

| Event ID | Meaning                             |
| -------: | ----------------------------------- |
| **4662** | Operation performed on an AD object |
| **5136** | Directory object modified           |
| **5137** | Directory object created            |
| **5138** | Directory object undeleted          |
| **5139** | Directory object moved              |
| **5141** | Directory object deleted            |

Event **4662** can help detect sensitive directory operations such as replication-related access when proper auditing is configured.

## 11. Audit Policy and Log Tampering

| Event ID | Meaning                            |
| -------: | ---------------------------------- |
| **1102** | Security audit log cleared         |
|  **104** | Another Windows event log cleared  |
| **4719** | System audit policy changed        |
| **4616** | System time changed                |
| **4904** | Security event source registered   |
| **4905** | Security event source unregistered |
| **4907** | Object audit settings changed      |

**1102** is one of the highest-priority SOC events because attackers frequently clear logs to hide activity. Microsoft includes it even in its minimal recommended Sentinel event set. ([Microsoft Learn][1])

## 12. Windows Firewall and Network Filtering

| Event ID | Meaning                                |
| -------: | -------------------------------------- |
| **4946** | Firewall rule added                    |
| **4947** | Firewall rule modified                 |
| **4948** | Firewall rule deleted                  |
| **4950** | Firewall setting changed               |
| **4954** | Group Policy firewall settings applied |
| **5152** | Network packet blocked                 |
| **5154** | Application permitted to listen        |
| **5155** | Application blocked from listening     |
| **5156** | Network connection allowed             |
| **5157** | Network connection blocked             |

Events **5156** and **5157** can be high volume, so filter them by unusual applications, ports, destinations and paths.

## 13. RDP-Specific Events

`TerminalServices-RemoteConnectionManager/Operational`:

| Event ID | Meaning                      |
| -------: | ---------------------------- |
| **1149** | RDP authentication succeeded |

`TerminalServices-LocalSessionManager/Operational`:

| Event ID | Meaning                  |
| -------: | ------------------------ |
|   **21** | RDP session logon        |
|   **22** | RDP shell started        |
|   **23** | RDP session logoff       |
|   **24** | RDP session disconnected |
|   **25** | RDP session reconnected  |

Correlate these with **4624 Logon Type 10**, source IP, username and device.

## 14. AppLocker

| Event ID | Meaning                                   |
| -------: | ----------------------------------------- |
| **8001** | AppLocker policy applied                  |
| **8002** | EXE or DLL allowed                        |
| **8003** | EXE or DLL would be blocked in audit mode |
| **8004** | EXE or DLL blocked                        |
| **8005** | Script or MSI allowed                     |
| **8006** | Script or MSI would be blocked            |
| **8007** | Script or MSI blocked                     |

AppLocker events help identify unauthorized binaries, scripts and execution-policy violations. ([Microsoft Learn][1])

# Sysmon Event IDs

Sysmon logs are located at:

`Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`

|      ID | Meaning                            |
| ------: | ---------------------------------- |
|   **1** | Process creation                   |
|   **2** | File creation time changed         |
|   **3** | Network connection                 |
|   **4** | Sysmon service state changed       |
|   **5** | Process terminated                 |
|   **6** | Driver loaded                      |
|   **7** | DLL/image loaded                   |
|   **8** | CreateRemoteThread                 |
|   **9** | Raw disk access                    |
|  **10** | Process accessed another process   |
|  **11** | File created                       |
|  **12** | Registry object created or deleted |
|  **13** | Registry value set                 |
|  **14** | Registry key/value renamed         |
|  **15** | Alternate Data Stream created      |
|  **16** | Sysmon configuration changed       |
|  **17** | Named pipe created                 |
|  **18** | Named pipe connected               |
|  **19** | WMI event filter created           |
|  **20** | WMI event consumer created         |
|  **21** | WMI consumer bound to filter       |
|  **22** | DNS query                          |
|  **23** | File deleted and archived          |
|  **24** | Clipboard changed                  |
|  **25** | Process tampering detected         |
|  **26** | File deletion detected             |
|  **27** | Executable creation blocked        |
|  **28** | File shredding blocked             |
|  **29** | New executable file detected       |
| **255** | Sysmon error                       |

The most valuable Sysmon events for SOC analysis are generally **1, 3, 6, 7, 8, 10–15, 17–22, 25 and 29**. ([Microsoft Learn][7])

# Microsoft Defender Antivirus Events

Log: `Microsoft-Windows-Windows Defender/Operational`

| Event ID | Meaning                                    |
| -------: | ------------------------------------------ |
| **1116** | Malware or unwanted software detected      |
| **1117** | Defender action taken against threat       |
| **1118** | Threat remediation action failed           |
| **1119** | Critical threat remediation action failed  |
| **5001** | Real-time protection disabled              |
| **5004** | Real-time protection configuration changed |
| **5007** | Defender configuration changed             |
| **5008** | Defender engine failure                    |
| **5010** | Antispyware protection disabled            |
| **5012** | Antivirus protection disabled              |
| **5013** | Tamper Protection blocked a change         |
| **3002** | Real-time protection failure               |
| **2003** | Defender engine update failed              |

Unexpected **5001**, **5007**, **5010**, or **5012** may indicate defense evasion. ([Microsoft Learn][8])

## Top 20 to Memorize First

```text
1102  Logs cleared
4624  Successful logon
4625  Failed logon
4648  Explicit credentials
4672  Admin privileges assigned
4688  Process created
4697  Service installed
4698  Scheduled task created
4719  Audit policy changed
4720  User created
4724  Password reset
4728  Added to global group
4732  Added to local group
4740  Account locked
4768  Kerberos TGT
4769  Kerberos service ticket
4771  Kerberos pre-authentication failed
5140  Network share accessed
5145  Detailed share access
4104  PowerShell script content
```

Also prioritize **Sysmon 1, 3, 10, 11, 13, 22 and 25**.

[1]: https://learn.microsoft.com/en-us/azure/sentinel/windows-security-event-id-reference "Windows security event sets that can be sent to Microsoft Sentinel | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor "Appendix L - Events to Monitor | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4768?utm_source=chatgpt.com "4768(S, F) A Kerberos authentication ticket (TGT) was ..."
[4]: https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/windows/debug-customscriptextension-runcommand-scripts "Debug PowerShell scripts run by Custom Script Extension or Run Command - Virtual Machines | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4720?utm_source=chatgpt.com "4720(S) A user account was created. - Windows 10"
[6]: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4663?utm_source=chatgpt.com "4663(S) An attempt was made to access an object."
[7]: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon "Sysmon - Sysinternals | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/defender-endpoint/troubleshoot-microsoft-defender-antivirus "Microsoft Defender Antivirus event IDs and error codes - Microsoft Defender for Endpoint | Microsoft Learn"
