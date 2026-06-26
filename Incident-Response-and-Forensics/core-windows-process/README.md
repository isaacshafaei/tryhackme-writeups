### Windows Core Processes — Short Notes

This room teaches the normal behaviour of **Windows processes**, helping analysts distinguish legitimate activity from malicious processes.

### Why This Matters

* Windows is widely used and often targeted by attackers.
* Traditional antivirus is no longer sufficient against modern malware.
* Antivirus mainly detects known threats and can be bypassed.
* Modern defence uses a layered approach, including **EDR**.

### Role of Security Analysts

When security tools alert on a suspicious process or binary, analysts must:

* Investigate the process
* Compare it with expected Windows behaviour
* Determine whether it is benign or malicious
* Decide the appropriate response

Relevant roles include:

* SOC Analyst
* Security Analyst
* Threat Hunter
* Detection Engineer

**Key point:** Understanding normal Windows processes makes it easier to detect abnormal or malicious endpoint activity.
---
### Windows Task Manager — Short Notes

**Task Manager** is a built-in Windows utility used to:

* View running processes
* Monitor CPU and memory usage
* End unresponsive processes
* Inspect process details

### Process Categories

Processes are grouped into:

* Apps
* Background processes
* Windows processes

### Important Columns

* **Type:** Process category
* **Publisher:** Software author
* **PID:** Unique process identifier
* **Process name:** Executable filename
* **Command line:** Full command used to launch it
* **Image path:** Executable location
* **CPU/Memory:** Resource usage

Image path and command line help detect suspicious process locations or arguments.

### Task Manager Limitation

Task Manager does not clearly show **parent-child process relationships**.

Example: A legitimate `svchost.exe` should normally have `services.exe` as its parent. An unexpected parent may indicate malicious activity.

For deeper analysis, use:

* **Process Explorer**
* **Process Hacker**

### Command-Line Alternatives

```powershell
tasklist
Get-Process
ps
wmic process
```

**Key point:** Check process name, path, command line, PID, and parent process—not only the filename.
---
### Windows System Process — Short Notes

The **System** process hosts kernel-mode threads used by the Windows kernel and device drivers.

### Normal Behaviour

* **PID:** Always `4`
* **Instances:** Only one
* **User:** `Local System`
* **Session:** Session `0`
* **Start time:** During system boot
* **Parent:** None or `System Idle Process (PID 0)`
* **Image path:** May appear as `N/A` or `C:\Windows\System32\ntoskrnl.exe`

### Suspicious Indicators

* PID is not `4`
* Multiple System processes exist
* Unexpected parent process
* Running outside Session `0`

**Practice answer:** The System process should always have **PID 4**.
---
### `smss.exe` — Session Manager Subsystem

`smss.exe` is the **first user-mode process** started by the Windows kernel. It creates and initializes Windows sessions.

### Main Functions

* Creates user and system sessions
* Starts paging files and environment variables
* Launches:

  * **Session 0:** `csrss.exe` and `wininit.exe`
  * **User sessions:** `csrss.exe` and `winlogon.exe`
* Starts configured Windows subsystems

### Normal Behaviour

* **Path:** `C:\Windows\System32\smss.exe`
* **Parent:** `System` — PID `4`
* **User:** `Local System`
* **Instances:** One master process; temporary child instances create new sessions and then exit
* **Start:** Within seconds of boot

### Suspicious Indicators

* Parent is not `System`
* Executable runs outside `System32`
* Multiple persistent instances
* User is not `SYSTEM`
* Unexpected subsystem registry entries

**Practice answer:** In Session 1, `smss.exe` spawns `winlogon.exe` alongside `csrss.exe`.
---
### `csrss.exe` — Client Server Runtime Process

`csrss.exe` is a critical user-mode Windows process. Terminating it can cause system failure.

### Main Functions

* Manages Win32 console windows
* Creates and deletes processes and threads
* Provides parts of the Windows API
* Maps drive letters
* Supports the Windows shutdown process

### Normal Behaviour

* **Path:** `C:\Windows\System32\csrss.exe`
* **Created by:** `smss.exe`
* **Parent:** Usually appears absent because `smss.exe` terminates after launching it
* **User:** `Local System`
* **Instances:** Two or more—typically one for Session 0 and one for Session 1
* **Start:** First instances start within seconds of boot

### Suspicious Indicators

* A currently running parent process is shown
* Executable is outside `System32`
* Misspelled names imitating `csrss.exe`
* Running under a user other than `SYSTEM`

**Practice answer:** Processes with PID `384` and `488` were `smss.exe`.
---
### `wininit.exe` — Windows Initialization Process

`wininit.exe` is a critical Windows process that initializes important system services in **Session 0**.

### Child Processes

It launches:

* `services.exe` — Service Control Manager
* `lsass.exe` — Local Security Authority
* `lsaiso.exe` — Credential Guard and KeyGuard process

`lsaiso.exe` only runs when **Credential Guard is enabled**.

### Normal Behaviour

* **Path:** `C:\Windows\System32\wininit.exe`
* **Created by:** `smss.exe`
* **Parent:** Usually appears absent because `smss.exe` terminates
* **Instances:** One
* **User:** `Local System`
* **Start:** Within seconds of boot

### Suspicious Indicators

* Visible running parent process
* Path outside `System32`
* Misspelled process name
* Multiple instances
* Not running as `SYSTEM`

**Practice answer:** The process that may be absent when Credential Guard is disabled is `lsaiso.exe`.
---
### `services.exe` — Service Control Manager

`services.exe` manages Windows services and device drivers.

### Main Functions

* Starts, stops, and interacts with services
* Loads auto-start device drivers
* Maintains the service database
* Updates the **Last Known Good Configuration** after successful login
* Stores service configuration under:

```text
HKLM\System\CurrentControlSet\Services
```

Services can be queried using:

```cmd
sc.exe
```

### Common Child Processes

* `svchost.exe`
* `spoolsv.exe`
* `msmpeng.exe`
* `dllhost.exe`

### Normal Behaviour

* **Path:** `C:\Windows\System32\services.exe`
* **Parent:** `wininit.exe`
* **Instances:** One
* **User:** `Local System`
* **Start:** Within seconds of boot

### Suspicious Indicators

* Parent is not `wininit.exe`
* Path is outside `System32`
* Misspelled process name
* Multiple instances
* Not running as `SYSTEM`

**Practice answer:** Only **one** instance of `services.exe` should be running.
---
### `svchost.exe` — Service Host

`svchost.exe` hosts and manages Windows services implemented as DLLs.

### Main Details

* Service DLLs are defined under:

```text
HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>\Parameters
```

* The `ServiceDLL` value identifies the DLL used by the service.
* The `-k` parameter groups related services under a service-host group.

Example:

```cmd
C:\Windows\System32\svchost.exe -k DcomLaunch
```

### Normal Behaviour

* **Path:** `C:\Windows\System32\svchost.exe`
* **Parent:** `services.exe`
* **Instances:** Many
* **Users:** `SYSTEM`, `Network Service`, `Local Service`, or sometimes the logged-in user
* **Start:** Usually during boot, although new instances may start later

### Suspicious Indicators

* Parent is not `services.exe`
* Path is outside `System32`
* Misspelled names such as `scvhost.exe`
* Missing expected `-k` parameter
* Unknown or malicious `ServiceDLL`

**Practice answer:** The expected command-line parameter is `-k`, so the letter is **k**.

