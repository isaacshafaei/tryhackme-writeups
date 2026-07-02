## Elastic Alert Investigation Short Notes

### Scenario

* Investigate web alerts in Kibana.
* Suspicious IP:

```text
203.0.113.55
```

* First alert: POST requests to:

```text
proxyLogon.ecp
```

* Possible ProxyLogon exploitation.

---

## Query for POST Requests

```kql
_index:weblogs and client.ip:203.0.113.55 and http.request.method:POST
```

### Table Fields

Add these fields as columns:

* `client.ip`
* `user.agent`
* `http.request.method`
* `url.path`
* `http.response.status_code`

### Findings

* IP made automated POST requests.
* User-Agent suggests scripted activity:

```text
python-requests/2.25.1
```

---

## Second Alert: Possible Web Shell

* Same IP triggered another alert 7 minutes later.
* Suspicious file:

```text
errorEE.aspx
```

* Suspicious query parameter:

```text
cmd=
```

`cmd=` is commonly used in web shells to execute commands.

---

## Query for Web Shell Activity

```kql
_index:weblogs and client.ip:203.0.113.55 and http.request.method:GET and errorEE.aspx
```

### Action

* Sort results **Old → New**.
* Check `url.path` for commands.

### Finding

* Commands were executed through `errorEE.aspx`.
* This confirms likely compromise.
* Classification: **True Positive**
* Escalate to senior/SOC L2.

---

## Answers

* POST requests to `proxyLogon.ecp`: **3**
* User-Agent used: **python-requests/2.25.1**
* Logs containing `cmd=`: **20**
* Command run at `Jul 20, 2025 @ 04:45:50.000`: **hostname**
---
## Host-Based Windows Investigation Short Notes

### Scenario

* Web investigation showed suspicious traffic.
* Pivot to Windows host logs to check attacker activity.
* Target host:

```text
winserv2019.some.corp
```

* Suspicious IP:

```text
203.0.113.55
```

---

## Confirm Administrator Logon

Search Windows Security Event ID `4624`:

```kql
@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:4624 and host.name:winserv2019.some.corp and winlog.event_data.TargetUserName:Administrator
```

### Important Fields

* `winlog.event_id` = Windows Event ID
* `host.name` = target host
* `winlog.event_data.TargetUserName` = logged-in user
* `winlog.logon.type` = logon method
* `winlog.event_data.IpAddress` = source IP

### Finding

* `Administrator` logged in from:

```text
203.0.113.55
```

* Same IP as previous web attacks.

---

## Sysmon Process Creation Check

Search Sysmon Event ID `1`:

```kql
@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:1 and user.name:Administrator
```

### Important Fields

* `user.name`
* `process.parent.name`
* `process.command_line`

### Purpose

* Confirms what processes started after Administrator login.
* Helps understand post-login activity.

---

## New User Account Investigation

Search User Account Management logs:

```kql
@timestamp >= "2025-07-20T05:13:10.000" and winlog.channel:Security and winlog.task:User Account Management
```

### Table Fields

* `winlog.event_id`
* `winlog.task`
* `message`

### Finding

* A new user account was created.
* This indicates possible persistence.

---

## Answers

* Administrator `4624` logon `winlog.record_id`: **17166**
@timestamp >= “2025–07–20T05:11:22” and winlog.event_id:4624 and host.name:winserv2019.some.corp and winlog.event_data.TargetUserName:Administrator

* Sysmon Event ID `1` `process.pid` at `05:11:27.996`: **964**
Using time stamp filter the log to see process pid
`event.code: 1 AND @timestamp: “2025–07–20T05:11:27.996"`

* New user account event ID: **4720**
Event id of new user creation in window is 4720

* New user account name: **svc_backup**
i used this code `@timestamp >= "2025-07-20T05:11:27.996" and winlog.event_id : 4720`
and then from left sidebar add the `winlog.event_data.TargetUserName`
---
