### Volatility — Short Notes

**Volatility** is a memory forensics framework used to extract digital artifacts from RAM dumps.

### Main Purpose

It helps analysts investigate the runtime state of a system, such as:

* Running processes
* Network connections
* Loaded DLLs
* Malware traces
* Command history
* Registry and credential artifacts

### Key Points

* Works on **volatile memory/RAM samples**
* Analysis is done **offline**, separate from the original system
* Uses many plugins to extract different types of evidence
* First step is usually identifying the memory image type/profile
* Then analysts run plugins to investigate the dump

### Volatility Versions

* **Volatility 2:** Python 2, older syntax
* **Volatility 3:** Python 3, newer syntax

For modern usage, **Volatility 3** is recommended.

**Key point:** Volatility gives deep visibility into what was happening on a system at the time the memory dump was captured.
---
### Memory Dump Extraction — Short Notes

A **memory dump** captures RAM for forensic analysis.

### Common Tools for Bare-Metal Systems

* **FTK Imager**
* **Redline**
* **DumpIt.exe**
* **win32dd / win64dd**
* **Memoryze**
* **FastDump**

Most tools produce a `.raw` memory file, but some, like **Redline**, may use their own agent/session format.

### Important Point

Memory acquisition from a physical machine can take time, so consider investigation urgency and system impact.

### Virtual Machine Memory Files

For lab VMs, memory can often be collected from the hypervisor files:

* **VMware:** `.vmem`
* **Hyper-V:** `.bin`
* **Parallels:** `.mem`
* **VirtualBox:** `.sav` — partial memory only

**Key point:** Handle memory files carefully because they contain volatile forensic evidence from the running system.
---
### Volatility 3 Plugin Structure — Short Notes

Volatility 3 changed how memory analysis plugins work.

### Main Changes

* **No more OS profiles:**
  Volatility 3 automatically detects the operating system and build from the memory image.

* **OS-specific plugin names:**
  You must specify the operating system before the plugin name.

Examples:

```bash
windows.info
linux.info
mac.info
```

### Old vs New Syntax

* **Volatility 2:** plugin name only, e.g. `pslist`
* **Volatility 3:** OS + plugin name, e.g. `windows.pslist`

### Plugin Discovery

Use the help menu to see available plugins:

```bash
vol.py -h
```

### Key Point

Volatility 3 is easier because it removes manual profile selection, but you must use OS-specific plugin syntax.
---
### Volatility Image Info — Important Notes

* In **Volatility 2**, you must identify the correct OS profile before analysis.
* The `imageinfo` plugin suggests possible Windows profiles from a memory dump.
* `imageinfo` is not always accurate, so test multiple suggested profiles if needed.
* In **Volatility 3**, OS profiles are removed; Volatility detects the OS/build automatically.
* To get host information in Volatility 3, use:

```bash
python3 vol.py -f <file> windows.info
```

Other OS options:

```bash
linux.info
mac.info
```

**Key point:** Volatility 3 is easier because it does not require manual profile selection.

----
**Volatility process/network plugins — short note**

* **`pslist`**: shows normal active/recent processes. Can miss hidden/rootkit processes.
* **`psscan`**: scans memory for process structures. Can find hidden processes, but may show false positives.
* **`pstree`**: shows parent-child process relationships. Good for understanding what started what.
* **`netstat` / `netscan`**: shows network connections from memory. Useful for finding C2 IPs and suspicious connections.
* **`dlllist`**: shows DLLs loaded by a process. Useful for checking suspicious libraries or injected components.

Use in your setup:

```bash
vol -f "$MEM" windows.pslist
vol -f "$MEM" windows.psscan
vol -f "$MEM" windows.pstree
vol -f "$MEM" windows.netscan
vol -f "$MEM" windows.dlllist --pid <PID>
```
----
### Volatility Hunting Plugins — Short Notes

Volatility has plugins useful for detecting malware, injection, and suspicious memory activity.

### `malfind`

Used to detect possible **code injection** or **fileless malware**.

It shows:

* Injected process
* PID
* Memory offset
* Hex/ASCII/disassembly view
* Suspicious memory permissions like `RWE` or `RX`

Run:

```bash id="xk71v6"
python3 vol.py -f <file> windows.malfind
```

**Important:** An `MZ` header may indicate an injected Windows executable. Shellcode needs deeper analysis.

### `yarascan`

Used to scan memory with **YARA rules** for known strings, patterns, or malware signatures.

Run:

```bash id="y2nixw"
python3 vol.py -f <file> windows.yarascan
```

**Key point:** `malfind` helps find injected code, while `yarascan` helps match memory content against known detection rules.
---
### Advanced Volatility Hunting — Short Notes

Advanced malware/rootkits may hide using **hooking** and **driver manipulation**.

### Hooking Types

* SSDT hooks
* IRP hooks
* IAT hooks
* EAT hooks
* Inline hooks

### `ssdt`

Detects possible **SSDT hooking**.

SSDT = **System Service Descriptor Table**, used by the Windows kernel to locate system functions. Malware/rootkits may hook it to redirect execution.

```bash
python3 vol.py -f <file> windows.ssdt
```

**Note:** Some hooks can be legitimate, so compare with a baseline or investigate suspicious entries.

### Driver Hunting

#### `modules`

Lists loaded kernel modules/drivers.

```bash
python3 vol.py -f <file> windows.modules
```

Useful for finding active malicious drivers.

#### `driverscan`

Scans memory for drivers that may be hidden or missed by `modules`.

```bash
python3 vol.py -f <file> windows.driverscan
```

### Other Useful Plugins

* `modscan`
* `driverirp`
* `callbacks`
* `idt`
* `apihooks`
* `moddump`
* `handles`

**Key point:** Use these plugins after initial investigation to hunt advanced malware, rootkits, hidden drivers, and kernel-level manipulation.
---

