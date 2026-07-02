Here is a better version with **each answer explained clearly**:

## Note: Investigating Potential C2 Communication in Kibana

In this TryHackMe scenario, we investigated a possible **C2 communication** from user **Browne** in the HR department. The available evidence was one week of HTTP connection logs stored in the `connection_logs` index in Kibana.

First, I connected to the TryHackMe network using **OpenVPN**, opened the machine IP in the browser, and used **Kibana Discover** to analyze the logs.

---

### 1. How many events were returned for March 2022?

To answer this, I opened **Kibana Discover** and changed the time range in the top-right corner to cover **March 2022**.

After applying the date filter, Kibana showed the total number of matching events.

**Answer: 1482**

This was found directly from the total event count displayed in Discover after filtering the logs for March 2022.

---

### 2. What is the IP associated with the suspected user?

To find the suspected user’s IP, I checked the `source_ip` field in the logs.

At first, there were only two IP addresses. The IP with the highest number of hits looked normal, so the suspicious one was the IP with fewer but more unusual events.

**Answer: 192.166.65.54**

This IP was associated with the suspicious activity related to user **Browne**.

---

### 3. What legit Windows binary was used to download a file?

Since the question mentioned a file download, I filtered the logs for HTTP `GET` requests from the suspicious IP.

I used the suspicious IP and checked the event details. In the `user_agent` field, I found:

`bitsadmin`

BITSAdmin is a legitimate Windows command-line tool used to create download or upload jobs. Attackers can abuse it to download files from a C2 server.

**Answer: bitsadmin**

This was found from the `user_agent` field in the suspicious HTTP GET request.

---

### 4. What file-sharing site acted as the C2 server?

Using the same suspicious event, I checked the destination domain or host field.

The infected machine connected to:

`pastebin.com`

Pastebin is a legitimate file-sharing site, but malware authors sometimes abuse it to host commands, payloads, or configuration files.

**Answer: pastebin.com**

This was found by checking the domain contacted by the suspicious IP.

---

### 5. What is the full C2 URL?

After finding the domain `pastebin.com`, I checked the URI/path field in the same HTTP event.

The URI was:

`/yTg0Ah6a`

By combining the domain and URI, the full URL became:

`pastebin.com/yTg0Ah6a`

**Answer: pastebin.com/yTg0Ah6a**

This was achieved by combining the destination host with the accessed URI path.

---

### 6. What is the name of the file accessed?

After opening the full Pastebin URL in the browser, the page showed the accessed file name.

The file name displayed was:

`secret.txt`

**Answer: secret.txt**

This was found by visiting the C2 URL and checking the page/file title.

---

### 7. What secret code was inside the file?

The accessed file contained a secret flag in the format `THM{_____}`.

After opening the file, the secret code was visible inside the content.

**Answer: THM{SECRET__CODE}**

This was found by reading the content of `secret.txt` on Pastebin.

---

### Key Lesson

The main lesson from this investigation is that the most frequent IP is not always the suspicious one. In SOC investigations, we should focus on unusual behavior, context, user activity, HTTP methods, user-agent values, and suspicious external domains.
