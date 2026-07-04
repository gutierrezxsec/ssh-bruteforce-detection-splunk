# 🛡️ SSH Authentication Monitoring & Brute Force Detection

A SOC-focused homelab project simulating an SSH brute-force attack and detecting it end-to-end using Splunk Enterprise — from log forwarding, to attack simulation, to dashboarding and alert validation.

---

## 📑 Table of Contents

- [Step 1 – Homelab Architecture](#step-1--homelab-architecture)
- [Step 2 – Environment Preparation](#step-2--environment-preparation)
- [Step 3 – Configuring Centralized Log Collection](#step-3--configuring-centralized-log-collection)
- [Attack Simulation – SSH Brute Force Attack](#attack-simulation--ssh-brute-force-attack)
- [Dashboard Creation](#dashboard-creation)
- [Alert Creation](#alert-creation)
- [Alert Validation](#alert-validation)

---

## Step 1 – Homelab Architecture

### Objective

Build a SOC-focused homelab that simulates attacker activity against a Linux endpoint while forwarding endpoint logs to a centralized Splunk SIEM for monitoring, detection, and investigation. The goal of this lab is to create a realistic but controlled environment where security events can be generated, collected, and analyzed in a way that mirrors a small enterprise security workflow.

### Lab Architecture

| Component | Role | Purpose in the Lab |
|---|---|---|
| **Main Laptop** | Primary host machine | Runs the main security tools for the lab. This device hosts Splunk Enterprise for log ingestion and analysis, and it also runs the Kali Linux virtual machine used to simulate attacker activity. It acts as the central workstation for monitoring, testing, and reviewing security events. |
| **Splunk Enterprise** | SIEM platform | Serves as the central logging and analysis platform. It receives logs from the Ubuntu victim machine, indexes the data, and allows searches, dashboards, alerts, and investigations to be performed on the collected events. |
| **Kali Linux VM** | Attacker simulation machine | Used to generate realistic offensive activity such as reconnaissance, port scanning, service enumeration, and login attempts. This VM represents the adversary side of the lab and is used to test whether Splunk can detect suspicious behavior. |
| **Spare Laptop** | Secondary host machine | Dedicated to running the victim-side virtual machine. Using a separate physical laptop helps isolate the victim environment from the attacker environment and makes the lab more stable when both systems are running at the same time. |
| **Ubuntu Server VM** | Victim endpoint | Acts as the target system in the lab. This machine generates authentication logs, system logs, and other endpoint activity that can be monitored by Splunk. It is the system being observed during simulated attacks. |
| **Splunk Universal Forwarder** | Log forwarding agent | Installed inside the Ubuntu Server VM to collect local logs and securely forward them to Splunk Enterprise. It is responsible for sending endpoint data from the victim machine to the SIEM for centralized analysis. |
| **Mobile Hotspot Phone** | Network provider | Provides the wireless network connection that links both laptops and their virtual machines. The phone hotspot acts as the temporary local network for the lab, allowing the attacker machine, victim machine, and Splunk server to communicate without requiring a home router or enterprise network. |

### Device Roles in Detail

**Main Laptop**
The main laptop is the control center of the lab. It is used to run the security monitoring tools and the attacker simulation environment. Because it hosts Splunk Enterprise, it is responsible for receiving and analyzing logs from the victim machine. It also runs the Kali Linux VM, which is used to generate attack traffic and test detection capabilities. This device is the primary workstation for interacting with the lab.

**Spare Laptop**
The spare laptop is dedicated to the victim environment. It runs the Ubuntu Server VM, which acts as the endpoint being monitored. Separating the victim machine onto a second physical laptop helps reduce performance issues and keeps the lab organized. It also makes it easier to simulate a more realistic environment where the attacker and victim are on different systems.

**Ubuntu Server VM**
The Ubuntu Server VM is the target endpoint in the lab. It generates logs related to user authentication, system activity, service behavior, and other security-relevant events. These logs are important because they provide the raw data that Splunk will analyze. The Ubuntu VM is the machine that will be scanned, probed, and monitored during the lab exercises.

**Kali Linux VM**
The Kali Linux VM is the attacker simulation platform. It is used to perform actions such as Nmap scans, service discovery, and other reconnaissance techniques. The purpose of this VM is not to cause damage, but to create realistic security events that can be detected and investigated in Splunk. It represents the adversary side of the lab.

**Splunk Enterprise**
Splunk Enterprise is the SIEM platform at the center of the lab. It collects logs from the Ubuntu Server VM through the Splunk Universal Forwarder, indexes the data, and makes it searchable. This allows the lab to move beyond simple attack simulation and into actual security monitoring and analysis. Splunk is where detections, dashboards, and investigations will be performed.

**Splunk Universal Forwarder**
The Splunk Universal Forwarder is a lightweight agent installed on the Ubuntu Server VM. Its job is to collect logs from the local system and send them to Splunk Enterprise. It does not analyze the logs itself; instead, it acts as the delivery mechanism that moves endpoint data from the victim machine to the SIEM.

**Mobile Hotspot Phone**
The mobile hotspot phone provides the network connection for the entire lab. Since both laptops are connected through the hotspot, it acts as the temporary local network that allows communication between the attacker machine, the victim machine, and the Splunk server. This is especially useful when a wired router or stable home network is not available. Because hotspot IP addresses can change, the lab may require occasional network reconfiguration.

### Expected Workflow

1. The mobile hotspot phone provides network connectivity for both laptops and their virtual machines.
2. The main laptop runs Splunk Enterprise and the Kali Linux VM.
3. The spare laptop runs the Ubuntu Server VM, which acts as the victim endpoint.
4. The Splunk Universal Forwarder inside the Ubuntu VM collects local logs.
5. The forwarder sends those logs to Splunk Enterprise on the main laptop.
6. Kali Linux performs simulated attacker activity against the Ubuntu Server.
7. The Ubuntu Server records the resulting security-related events.
8. Splunk indexes the logs and makes them available for searching and investigation.
9. The activity is analyzed to determine whether the simulated attack can be detected and understood through SIEM monitoring.

---

## Step 2 – Environment Preparation

### Software Used

| Software | Purpose |
|---|---|
| Windows 11 | Host operating system |
| VirtualBox | Virtualization platform |
| Kali Linux | Attack simulation |
| Ubuntu Server 24.04 LTS | Victim endpoint |
| Splunk Enterprise | Security Information and Event Management (SIEM) |
| Splunk Universal Forwarder | Log forwarding agent installed on the Ubuntu Server |

### Network Configuration

- Both physical laptops are connected to the same local hotspot network.
- Virtual machines use a **Bridged Adapter** to obtain their own IP addresses.
- The Ubuntu Server is reachable from the Kali VM for attack simulation.
- Splunk Enterprise receives logs over the local network.

### Current Progress

- ✅ VirtualBox installed on both laptops.
- ✅ Kali Linux configured and operational.
- ✅ Ubuntu Server installed on the spare laptop.
- ✅ Splunk Enterprise configured on the main laptop.
- ✅ Splunk Universal Forwarder downloaded and ready for installation.
- ✅ Network connectivity between both laptops verified.

---

## Step 3 – Configuring Centralized Log Collection

### Objective

Configure the Ubuntu Server VM to securely forward system logs to Splunk Enterprise using the Splunk Universal Forwarder. This establishes the centralized logging pipeline that will be used throughout the SOC homelab.

### Installation Process

The Splunk Universal Forwarder for Linux (64-bit Debian package) was downloaded from the official Splunk website and installed on the Ubuntu Server VM.

After installation, the forwarder service was started and configured to automatically start during system boot.

The following log sources were configured for monitoring:

- `/var/log/syslog`
- `/var/log/auth.log`
- `/var/log/kern.log`

These logs provide visibility into system events, authentication activity, and kernel-level events that are commonly investigated by SOC analysts.

### Forwarding Configuration

The Universal Forwarder was configured to send logs to the Splunk Enterprise instance running on the main laptop over TCP port **9997**.

Once configured, the monitored logs were successfully forwarded and indexed by Splunk Enterprise.

### 🔧 Troubleshooting

**Issue 1 — Forward Server Inactive**

The Splunk Universal Forwarder successfully monitored the configured log files, but the configured forward server appeared as **Inactive**.

*Investigation:*
- Verified that the monitored log sources were correctly configured.
- Confirmed that the forward server IP address and TCP port were correctly specified.
- Re-added the Splunk server as the forwarding destination.
- Verified that the receiving port (9997) was enabled within Splunk Enterprise.

**Issue 2 — Network Connectivity Test**

Connectivity testing showed that the Ubuntu VM could successfully reach the Splunk server using ICMP (`ping`). However, testing TCP connectivity to port **9997** resulted in a timeout. This indicated that the network path existed, but TCP traffic was being blocked before reaching Splunk Enterprise.

**Issue 3 — Windows Defender Firewall**

Further investigation identified Windows Defender Firewall on the main laptop as the root cause. Although Splunk Enterprise was actively listening on TCP port **9997**, inbound connections from the Ubuntu VM were blocked. After temporarily disabling the firewall, the Universal Forwarder immediately established an active connection and log forwarding began successfully.

### ✅ Result

**Figure 1.** Successful Log Ingestion from Ubuntu Server to Splunk Enterprise — Splunk Enterprise successfully receives and indexes system logs forwarded by the Splunk Universal Forwarder running on the Ubuntu Server VM.

<img width="1021" height="728" alt="Splunk log ingestion confirmation" src="https://github.com/user-attachments/assets/f089f632-cfe3-4c3a-b25e-7f90dbc86720" />

<img width="3120" height="4160" alt="Forwarder to Splunk network setup" src="https://github.com/user-attachments/assets/2007cfb2-4081-4563-a186-7157ee1d7b56" />

The centralized logging pipeline is now fully operational. Ubuntu Server continuously forwards endpoint logs to Splunk Enterprise, providing live visibility into authentication events, system activity, and future attack simulations. This configuration serves as the foundation for the remaining SOC detection and investigation scenarios in this homelab.

---

## Attack Simulation – SSH Brute Force Attack

### Objective

Simulate an SSH brute-force attack against an Ubuntu Server to generate authentication events that can be collected, monitored, and investigated using Splunk Enterprise.

### Attack Overview

A controlled SSH brute-force attack was launched from the Kali Linux attacker machine against the Ubuntu Server. The objective was not to compromise the system but to generate realistic authentication logs for SOC monitoring and detection.

The attack repeatedly attempted to authenticate using invalid credentials against the SSH service running on the Ubuntu Server.

### Attack Environment

| Machine | Role |
|---|---|
| Kali Linux | Attacker |
| Ubuntu Server 24.04 LTS | Target/Victim |
| Splunk Universal Forwarder | Log Forwarder |
| Splunk Enterprise | SIEM Platform |

### Attack Execution

A dedicated test account was created on the Ubuntu Server for the lab.

From the Kali Linux machine, the Hydra password auditing tool was used with a custom password list to perform multiple SSH authentication attempts against the target system.

```bash
hydra -l soclab -P passwords.txt ssh://<Ubuntu-IP> -t 4 -V
```

<img width="495" height="362" alt="Hydra SSH brute force execution" src="https://github.com/user-attachments/assets/255f2cb8-66be-4420-bbc2-a8ace70dc44e" />

**Command Parameters**

| Parameter | Description |
|---|---|
| `-l soclab` | Target username |
| `-P passwords.txt` | Password wordlist |
| `ssh://<Ubuntu-IP>` | Target SSH service |
| `-t 4` | Number of concurrent connections |
| `-V` | Display each login attempt |

### Expected Behavior

Each unsuccessful authentication attempt generated SSH authentication events within the Ubuntu authentication log. These events were automatically forwarded by the Splunk Universal Forwarder to Splunk Enterprise for analysis.

Typical events included:

- Failed password
- Invalid user
- Authentication failure
- Connection closed

### Log Source

```text
/var/log/auth.log
```

### Verification

The generated events were successfully received by Splunk Enterprise and verified using the following search:

```spl
index=* source="/var/log/auth.log" sshd
```

The search confirmed that the SSH daemon recorded each authentication attempt, including the timestamp, source IP address, targeted username, authentication result, and additional SSH session details.

<img width="1427" height="668" alt="SSH auth log search results in Splunk" src="https://github.com/user-attachments/assets/624eadfe-8fb9-47a0-b82a-6d553e377131" />

### ✅ Result

The simulated brute-force attack successfully generated authentication telemetry that could be analyzed in Splunk Enterprise. The collected logs provided sufficient information to identify repeated failed login attempts, determine the attack source, and build detection dashboards and alert rules based on SSH authentication activity.

---

## Dashboard Creation

To improve visibility of SSH authentication activity, a Splunk dashboard was created to monitor and investigate brute-force attacks in real time. The dashboard summarizes key authentication metrics and provides quick access to relevant security events generated during the attack simulation.

<img width="2244" height="2173" alt="SSH Brute Force Detection Dashboard" src="https://github.com/user-attachments/assets/dfdaffcd-c2db-40b6-a9b4-f6a714e449d7" />

### Dashboard Panels

**1. Failed Login Attempts Over Time**
Displays the number of failed SSH login attempts over time to identify spikes in authentication activity.

```spl
index=* source="/var/log/auth.log" "Failed password"
| timechart span=1m count AS "Failed Logins"
```

**2. Total Failed Login Attempts**
Shows the total number of failed SSH authentication attempts recorded during the selected time range.

```spl
index=* source="/var/log/auth.log" "Failed password"
| stats count AS "Failed Login Attempts"
```

**3. Top Attacking IP Addresses**
Identifies the source IP addresses responsible for the highest number of failed login attempts.

```spl
index=* source="/var/log/auth.log" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort -count
```

**4. Most Targeted Usernames**
Displays the usernames most frequently targeted during the brute-force attack.

```spl
index=* source="/var/log/auth.log" "Failed password"
| rex "for (invalid user )?(?<username>\S+)"
| stats count by username
| sort -count
```

**5. Authentication Success vs. Failure**
Compares successful and failed SSH authentication attempts to provide an overall authentication summary.

```spl
index=* source="/var/log/auth.log" sshd
| eval Result=case(
    like(_raw,"%Failed password%"),"Failed",
    like(_raw,"%Accepted password%"),"Successful"
)
| search Result=*
| stats count by Result
```

**6. Recent SSH Authentication Events**
Lists the latest SSH authentication events for detailed investigation and log review.

```spl
index=* source="/var/log/auth.log" sshd
| table _time host _raw
| sort -_time
```

The completed dashboard provides a centralized view of SSH authentication activity, enabling faster detection and investigation of brute-force attacks.

---

## Alert Creation

An automated Splunk alert was created to detect potential SSH brute-force attacks by monitoring repeated failed authentication attempts from the same source IP address.

The alert evaluates incoming SSH authentication logs and triggers when a source IP exceeds the defined threshold of failed login attempts within the configured time window. This helps identify suspicious activity early and provides immediate notification to security analysts.

```spl
index=* source="/var/log/auth.log" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| where count >= 10
```

This alert complements the dashboard by providing automated detection, while the dashboard supports investigation and analysis of the generated events.

---

## Alert Validation

### Objective

Validate that the configured Splunk alert successfully detects an SSH brute-force attack and triggers automatically when the defined threshold is exceeded.

### Attack Simulation

A second SSH brute-force attack was launched from the Kali Linux attacker machine using Hydra. The attack targeted the Ubuntu Server with a custom password list containing only incorrect passwords to generate multiple failed authentication attempts.

```bash
hydra -l soclab -P passwords.txt ssh://<Ubuntu-IP> -t 4 -V
```

<img width="608" height="303" alt="Second Hydra brute force attempt" src="https://github.com/user-attachments/assets/b36621bc-ad50-42ce-a4fd-29a70a135f0c" />

The attack generated ten failed SSH authentication attempts from the same source IP address within a short period, satisfying the alert conditions.

### Alert Detection

The Splunk alert continuously monitored SSH authentication logs and automatically triggered after detecting repeated failed login attempts from a single source IP.

**Alert Logic:**

```spl
index=* source="/var/log/auth.log" "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| where count >= 10
```

### ✅ Result

The alert successfully triggered during the simulated brute-force attack, confirming that the detection rule correctly identified excessive failed SSH authentication attempts from the attacking host.

This validation demonstrates that the SIEM can automatically detect potential SSH brute-force activity while providing analysts with immediate notification for further investigation.

<img width="1432" height="347" alt="Triggered Splunk alert" src="https://github.com/user-attachments/assets/a88f51f5-7a02-42ed-ae36-b2053e6fba5d" />

<img width="2112" height="2785" alt="Final SSH Brute Force Detection Dashboard" src="https://github.com/user-attachments/assets/3c3033e6-2a68-4d7e-9b17-062cf5e68624" />
