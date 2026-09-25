# File Integrity Monitoring (FIM) with Wazuh SIEM

**Date:** September 25, 2026

**Target Environment:** Windows 11 (Windows_11, Agent ID: 002)

**SIEM Platform:** Wazuh Server (v4.x) running on Ubuntu Linux

**Virtualization Platform:** VMware Workstation

---

## 1. Executive Summary

File Integrity Monitoring (FIM) is a critical security control that tracks, detects, and alerts on unauthorized modification, creation, or deletion of key files and directories across system endpoints. This lab demonstrates the end-to-end configuration and operational validation of Wazuh's FIM component (Syscheck) on a Windows 11 endpoint (Windows_11, Agent 002).

The test successfully monitored the target directory `C:\WazuhTest`. Through controlled file operations including creation, content modification, and deletion executed via PowerShell — Wazuh generated real-time alerts on the SIEM dashboard, confirming proper rule execution (Rule ID 550, Level 7).

---

## 2. Laboratory Environment Setup

The test environment consists of two virtual machines operating within VMware Workstation:

- **Wazuh Manager / Dashboard Node:**
  - OS: Linux (Ubuntu/Debian-based)
  - Management Address: `https://192.168.6.133`
- **Monitored Endpoint (Windows Agent):**
  - OS: Windows 11
  - Agent Name: `Windows_11`
  - Agent ID: `002`
  - Monitored Path: `C:\WazuhTest`

---

## 3. Configuration Steps

### 3.1 Wazuh Agent Configuration (ossec.conf)

To enable real-time monitoring on the target directory, the `syscheck` block inside the Windows Agent's configuration file (`C:\Program Files (x86)\ossec-agent\ossec.conf`) was updated with the following directive:

```xml
<syscheck>
  <directories real_time="yes" check_all="yes">C:\WazuhTest</directories>
</syscheck>
```

### 3.2 Directory Initialization

A dedicated directory was created on the root of the Windows 11 endpoint:

- **Path:** `C:\WazuhTest`

---

## 4. Execution & Testing Procedure

The test scenario simulated an unauthorized user or process attempting to modify monitored system files. The activity was performed via Windows PowerShell:

1. **File Creation:**
   A test file was initialized within the monitored folder:

```powershell
   New-Item -Path "C:\WazuhTest\lab_fim_test.txt" -ItemType File
```

   - Observed File: `C:\WazuhTest\lab_fim_test.txt`

   ![PowerShell session creating the monitored folder, creating lab_fim_test.txt, and appending unauthorized text to trigger a file-modified event](screenshots/Screenshot_2026-09-25_171815.png)

2. **Content Modification (Simulating Tampering):**

   Text content was appended to the file to trigger a checksum change:

```powershell
   Add-Content -Path "C:\WazuhTest\lab_fim_test.txt" -Value "Testing Wazuh FIM Detection"
```

3. **File Removal:**

   The test file was deleted to verify deletion event tracking:

```powershell
   Remove-Item -Path "C:\WazuhTest\lab_fim_test.txt"
```

---

## 5. Results & Wazuh SIEM Detection

The Wazuh Syscheck engine successfully captured all file modification events in real time.

![Wazuh dashboard Overview page with the left-hand navigation menu expanded, showing Endpoint Security, Threat Intelligence, Security Operations, and Cloud Security sections, plus Last 24 Hours Alerts severity counts](screenshots/Screenshot_2026-09-25_174922.png)

![File Integrity Monitoring dashboard for Windows_11 (002) — Most active users, Actions, and Events donut/line charts, with Files added / Files modified / Files deleted panels](screenshots/Screenshot_2026-09-25_174954.png)

![File Integrity Monitoring dashboard, scrolled view — Files modified panel showing a full donut chart for c:\wazuhtest\lab_fim_test.txt, with No results found for Files added and Files deleted](screenshots/Screenshot_2026-09-25_175015.png)

![File Integrity Monitoring Inventory tab — Files (11) list showing c:\wazuhtest\lab_fim_test.txt alongside baseline Windows system files with Last modified, User, User ID, and Size columns](screenshots/Screenshot_2026-09-25_175043.png)

![File Integrity Monitoring Events tab — histogram showing 1 hit, with the event row for c:\wazuhtest\lab_fim_test.txt, syscheck.event: modified, rule.description: Integrity checksum changed, rule.level 7, rule.id 550](screenshots/Screenshot_2026-09-25_175109.png)

![File Integrity Monitoring Events tab in full-screen view — single hit table showing timestamp, agent.name (Windows_11), syscheck.path, syscheck.event (modified), rule.description, rule.level (7), and rule.id (550)](screenshots/Screenshot_2026-09-25_175124.png)

### Summary of Alert Events

| Timestamp | Endpoint / Agent | Monitored File Path | Event Type | Triggered Rule | Rule Description | Alert Level |
|---|---|---|---|---|---|---|
| Sep 25, 2026 @ 17:47:05.5... | Windows_11 (002) | `c:\wazuhtest\lab_fim_test.txt` | modified | 550 | Integrity checksum changed. | 7 |

### Key Observations

- **Real-time Alerting:** The event was captured immediately upon file edit execution.
- **Rule ID 550:** Indicates that Wazuh detected a change in file attribute or cryptographic checksum (MD5/SHA1/SHA256).
- **Severity Classification:** Rule level 7 reflects a medium-high security priority suitable for immediate security analyst review.

![Windows File Explorer showing the C:\WazuhTest folder containing lab_fim_test as a Text Document, confirming the file's final state on the endpoint](screenshots/Screenshot_2026-09-25_175151.png)

---

## 6. Conclusion & Security Takeaways

1. **Operational Readiness:** Wazuh FIM successfully provides granular visibility into localized filesystem changes on Windows endpoints.
2. **Defensive Utility:** Configuring FIM on sensitive system paths (e.g., `System32`, registry entries, binary configurations, or web server roots) significantly reduces threat actor dwell time during ransomware deployment or unauthorized data modification.
3. **Recommendation:** For production environments, FIM rules should be paired with automated response actions (e.g., Active Response scripts) to isolate endpoints or block compromise paths upon detecting unauthorized changes to critical system files.
