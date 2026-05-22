# SOC Home Lab – Splunk & Sysmon Threat Detection

## Overview
This project demonstrates a mini Security Operations Center (SOC) home lab built using Splunk Enterprise, Sysmon, Kali Linux, and VirtualBox to simulate real-world attack and detection scenarios.

The objective of this lab was to gain hands-on experience in:
- SIEM configuration and log ingestion
- Endpoint monitoring and telemetry collection
- Threat detection and investigation
- SPL querying and event analysis
- Attack simulation and IOC identification

The environment simulates a real SOC workflow where logs are generated on a Windows endpoint, forwarded into Splunk, and analyzed for suspicious activity.

---

# Lab Architecture

## Environment
- Windows 11 Endpoint
- Kali Linux Attacker Machine
- Splunk Enterprise SIEM
- Sysmon
- Splunk Universal Forwarder
- Oracle VirtualBox

## Architecture Flow
```
Kali Linux (Attacker) → Windows 11 Endpoint → Sysmon → Splunk Universal Forwarder → Splunk Enterprise
```

---

# Lab Setup & Configuration

## Splunk Universal Forwarder – inputs.conf & Splunkd Service
![Splunk Forwarder Config](screenshots/splunk-forwarder-config.png)
> `inputs.conf` configured to forward Sysmon, Defender, PowerShell, Security, Application, and System logs. Splunkd service confirmed running.

## Splunk Endpoint Index Created
![Splunk Endpoint Index](screenshots/splunk-endpoint-index.png)
> The `endpoint` index is active in Splunk Enterprise, ready to receive forwarded telemetry from the Windows host.

---

# Attack Simulation – Kali Linux

## Step 1 – Nmap Reconnaissance Scan
![Kali Nmap Scan](screenshots/kali-nmap-scan.png)
> Aggressive Nmap scan (`-A -Pn`) against the Windows target `192.168.20.10`. Discovered open ports including 135 (RPC), 139 (NetBIOS), 445 (SMB), and 8000/8089 (Splunk).

## Step 2 – Payload Generation with msfvenom (Attempt 1)
![msfvenom Attempt 1](screenshots/kali-msfvenom-attempt1.png)
> First attempt at generating a reverse TCP payload — syntax error due to incorrect flag format (`lhost` vs `LHOST`).

## Step 3 – Payload Generation with msfvenom (Attempt 2)
![msfvenom Attempt 2](screenshots/kali-msfvenom-attempt2.png)
> Second attempt — error due to invalid type flag. Corrected on next attempt.

## Step 4 – Successful Payload Generation
![msfvenom Success](screenshots/kali-msfvenom-success.png)
> Successfully generated a Windows x64 Meterpreter reverse TCP payload saved as `resume.pdf.exe` (7680 bytes).

## Step 5 – Payload Confirmed on Attacker Machine
![Payload Confirmed](screenshots/kali-payload-confirmed.png)
> `ls` confirms `resume.pdf.exe` exists in the attacker's home directory, ready for delivery.

## Step 6 – Metasploit Listener & Python HTTP Server
![MSF Listener](screenshots/kali-msf-listener.png)
> Metasploit `multi/handler` configured with `LHOST=192.168.20.11` and `LPORT=4444`. Python HTTP server started on port 9999 to serve the payload to the victim machine.

---

# Payload Execution – Victim Machine (Windows 11)

## Payload Running in Task Manager
![Victim Payload Running](screenshots/victim-payload-running.png)
> Windows Task Manager showing `resume.pdf.exe` running as a background process on the victim machine, confirming successful execution.

## Post-Exploitation – Meterpreter Shell
![Meterpreter Post-Exploit](screenshots/meterpreter-post-exploit.png)
> Active Meterpreter session — attacker ran `net localgroup` (enumerating local groups) and `ipconfig` (confirming victim IP `192.168.20.10`) for situational awareness.

---

# Detection & Analysis – Splunk

## Splunk Log Ingestion – Multi-Source Events
![Splunk Log Ingestion](screenshots/splunk-log-ingestion.png)
> Splunk successfully ingesting Windows event logs from multiple sources including Security, Application, and System logs from host `ukim`.

## EventCode 4798 – Local Group Membership Enumeration
![Event 4798 Filter](screenshots/splunk-event4798-filter.png)
> Filtered Splunk search for EventCode `4798` — 130 events detected. This event indicates a user's local group membership was enumerated, a common post-exploitation recon technique.

## Sysmon Telemetry – index=endpoint (497 Events)
![Sysmon Events](screenshots/splunk-sysmon-events.png)
> `index=endpoint` search returning 497 Sysmon events. Raw XML telemetry visible, including process GUIDs, hashes (MD5, SHA256, IMPHASH), and parent process chains originating from Splunk and powershell.exe.

## C2 Beacon Detected – Outbound Connection to 192.168.20.11
![C2 Detection](screenshots/splunk-c2-detection.png)
> Sysmon network connection events showing the victim (`192.168.20.10`) making outbound connections to the attacker's C2 server at `192.168.20.11:4444`. The source process is identified as `Resume.pdf (2).exe` — confirming the C2 callback.

## Payload Artifact – Resume.pdf.exe (31 Events)
![Payload Artifact](screenshots/splunk-payload-artifact.png)
> 31 Sysmon events tied to `Resume.pdf.exe`. Events include file creation with Zone.Identifier metadata (downloaded from the internet), confirming the payload was fetched from the attacker's HTTP server at `http://192.168.20.11:9999/`.

---

# Log Sources Monitored

The following logs were ingested into Splunk:
- Sysmon Operational Logs
- Windows Defender Logs
- PowerShell Operational Logs
- Windows Security Logs
- Windows System Logs
- Windows Application Logs

---

# Splunk Configuration

Example `inputs.conf` configuration:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Windows Defender/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-Windows Defender/Operational
blacklist = 1151,1150,200,1002,1001,1000

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-Windows-PowerShell/Operational
blacklist = 4100,4105,4106,40961,40962,53504
```

---

# Key SPL Queries Used

```spl
index=endpoint
```
> Retrieve all events from the endpoint index.

```spl
source="WinEventLog:*" host="ukim" 4798
```
> Hunt for EventCode 4798 — local group enumeration activity.

```spl
index=endpoint 192.168.20.11
```
> Identify all Sysmon events referencing the attacker's IP — reveals C2 beaconing.

```spl
index=endpoint Resume.pdf.exe
```
> Track all activity related to the malicious payload across 31 events.

---

# Attack Chain Summary

| Stage | Tool | Action | Detection |
|---|---|---|---|
| Reconnaissance | Nmap | Port scan of victim | Sysmon network events |
| Weaponization | msfvenom | Generated `resume.pdf.exe` | — |
| Delivery | Python HTTP Server | Hosted payload on port 9999 | Sysmon file create + Zone.Identifier |
| Execution | Victim clicked payload | `resume.pdf.exe` ran | Task Manager / Sysmon Process Create |
| C2 | Metasploit handler | Reverse TCP shell on port 4444 | Sysmon network connection to 192.168.20.11 |
| Post-Exploitation | Meterpreter | `net localgroup`, `ipconfig` | EventCode 4798 (group enumeration) |
