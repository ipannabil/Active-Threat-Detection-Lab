# Active Threat Detection & SIEM Home Lab

## 📌 Objective
This project demonstrates the engineering and utilization of a custom Security Information and Event Management (SIEM) home lab. The primary goal was to bridge the gap between offensive cybersecurity tactics (Red Teaming) and defensive network monitoring (Blue Teaming) by generating, ingesting and analyzing real-world attack telemetry.

## 🏗️ Environment & Architecture
* **Attacker Node:** Kali Linux
* **SIEM Server:** Ubuntu (Splunk Enterprise)
* **Target Machine:** Windows 10
* **Sensors & Forwarders:** Splunk Universal Forwarder, Windows Event Logs (Security), Sysmon, Windows Defender Firewall
* **Offensive Tools:** Hydra, NetExec, Atomic Red Team, Nmap

## ⚔️ Attack Scenarios & Log Analysis

### Phase 1: Network Reconnaissance & Firewall Monitoring
* **The Attack:** Conducted targeted Nmap scans from Kali Linux against the Windows target to identify open ports and services. 
* **The Detection:** Configured the Windows Defender Firewall to log dropped packets, ingested the `pfirewall.log` into Splunk and visualized the attack surface to identify the unauthorized scanning activity.

<img width="727" height="697" alt="Screenshot 2026-05-27 194141" src="https://github.com/user-attachments/assets/c0cf3a74-3393-464a-867c-4bfe5d652c42" />
<img width="1859" height="848" alt="Screenshot 2026-05-27 194441" src="https://github.com/user-attachments/assets/37d38ca9-83d9-4a9f-9670-0a4136b9a98c" />


### Phase 2: Active Directory & SMB Brute Forcing
* **The Attack:** Lowering security by turning off Windows Defender Firewall and executing custom dictionary attacks using **Hydra**. After encountering Network Level Authentication (NLA) and modern SMB protections, pivoted to **NetExec** to successfully brute-force the target machine.
* **The Detection:** Queried Windows Security logs for `"eventcode=4625"` (Failed Logon) and `"eventcode=4624"` (Successful Logon). Successfully distinguished between human interactive logins (Logon Type 2) and background system noise (Logon Type 5).

<img width="981" height="713" alt="Screenshot 2026-05-29 000309" src="https://github.com/user-attachments/assets/69b523b3-d474-4d60-aa7b-79a4d171dceb" />
<img width="688" height="255" alt="Screenshot 2026-05-27 194650" src="https://github.com/user-attachments/assets/428e6c98-4f2e-4ce5-bbf7-d6bce23ea732" />
<img width="1854" height="849" alt="Screenshot 2026-05-29 001548" src="https://github.com/user-attachments/assets/1d32a9e3-b4c7-4cba-9dac-413bb5137b44" />

### Phase 3: Advanced PowerShell Malware Emulation
* **The Attack:** Deployed the **Atomic Red Team** framework to simulate an adversary gaining a foothold. Executed technique `T1059.001` (Command and Scripting Interpreter: PowerShell) to emulate fileless execution and security policy bypasses.
* **The Detection:** Engineered a custom Sysmon detection rule to filter out benign process creation. Queried the `CommandLine` field for suspicious execution flags (e.g., `-ExecutionPolicy Bypass`, `Invoke-AtomicTest`), successfully detecting the simulated malware.

<img width="1022" height="769" alt="Screenshot 2026-05-27 195641" src="https://github.com/user-attachments/assets/72589696-8b20-4645-8a30-2c9b308b1642" />
<img width="1023" height="720" alt="Screenshot 2026-05-27 195803" src="https://github.com/user-attachments/assets/4565532a-d79e-4eec-b0b2-b65a8dca47d1" />
<img width="1854" height="715" alt="Screenshot 2026-05-27 200231" src="https://github.com/user-attachments/assets/dcec2399-470a-4528-a17b-dfde06f51461" />



## 📊 Visualizations & Dashboards
To reduce alert fatigue and provide a high level overview of active threats, a real-time SOC Analyst Dashboard was engineered in Splunk.

<img width="1857" height="850" alt="Screenshot 2026-05-27 203243" src="https://github.com/user-attachments/assets/6a0947c8-c87e-41b8-a369-700c9e77b763" />

* **Authentication Status:** A pie chart comparing successful vs failed logins to detect brute-force spikes.
* **Network Timeline:** A timechart tracking dropped firewall packets to visualize network scanning patterns.
* **Sysmon Process Alerts:** A high fidelity single value counter specifically targeting malicious PowerShell execution.

## 🔍 Challenges & Troubleshooting
Throughout the lab, several technical hurdles were encountered and resolved:
* **SPL Query Syntax Errors:** Encountered issues where specific Splunk queries returned zero results despite the data existing in the index. Diagnosed the problem as a combination of strict case sensitivity (e.g., `eventcode` vs `EventCode`) and improper string formatting. Resolved by refining the syntax and utilizing raw text search functions to bypass broken field extractions.
* **UAC Split Token Mechanism:** Investigated "duplicate" successful logon events in Splunk and correctly attributed them to Windows generating dual Type 2 tokens (Standard and Administrator) upon privilege escalation.
* **Log Formatting (XML vs Plain Text):** Diagnosed search syntax failures caused by the `renderXml = true` configuration in the Universal Forwarder. Bypassed the issue by using the `searchmatch()` function in Splunk to accurately parse XML wrapped Event Codes before completely restructuring the inputs.conf file for clean data ingestion.
* **Protocol Hardening:** Adapted offensive strategies on the fly when built-in Windows protections (like NLA) blocked legacy tools like Hydra, successfully pivoting to modern frameworks like NetExec to complete the objective.

## 💡 Lessons Learned
* **Command-Line & Query Proficiency:** Gained hands-on experience executing advanced commands across multiple operating systems (Kali Linux, Windows PowerShell) and writing precise Splunk Processing Language (SPL) queries to filter out system noise and build high fidelity visual dashboards.
* **Advanced Endpoint Telemetry:** Mastered the process of configuring and setting up a monitoring environment utilizing Windows Event Viewer and Sysmon, ensuring the correct logging pipelines were established to capture granular process creation and network connection telemetry.
* **Log Fidelity Over Volume:** I learned firsthand that ingesting every single log creates massive alert fatigue. The true value of a SOC Analyst lies in highly specific, targeted queries that filter out background system noise.
* **Tool Limitations & Evasion:** Realizing that modern Windows protections completely block legacy attack tools taught me that Red Teaming is dynamic. The threat actor must understand *why* a tool failed to know how to pivot to a successful strategy.
* **The SIEM Pipeline:** Building this lab from the ground up solidified my understanding of data routing. Seeing how an action on a target machine travels through a Universal Forwarder, gets parsed by an indexer and is visualized on a dashboard demystified the entire security monitoring architecture.
---
**Author:** Irfan Nabil
