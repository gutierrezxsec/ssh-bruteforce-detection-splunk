# Step 1 – Homelab Architecture

## Objective

Build a SOC-focused homelab that simulates attacker activity against a Linux endpoint while forwarding endpoint logs to a centralized Splunk SIEM for monitoring, detection, and investigation. The goal of this lab is to create a realistic but controlled environment where security events can be generated, collected, and analyzed in a way that mirrors a small enterprise security workflow.

## Lab Architecture

| Component                      | Role                        | Purpose in the Lab                                                                                                                                                                                                                                                                                     |
| ------------------------------ | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Main Laptop**                | Primary host machine        | Runs the main security tools for the lab. This device hosts Splunk Enterprise for log ingestion and analysis, and it also runs the Kali Linux virtual machine used to simulate attacker activity. It acts as the central workstation for monitoring, testing, and reviewing security events.           |
| **Splunk Enterprise**          | SIEM platform               | Serves as the central logging and analysis platform. It receives logs from the Ubuntu victim machine, indexes the data, and allows searches, dashboards, alerts, and investigations to be performed on the collected events.                                                                           |
| **Kali Linux VM**              | Attacker simulation machine | Used to generate realistic offensive activity such as reconnaissance, port scanning, service enumeration, and login attempts. This VM represents the adversary side of the lab and is used to test whether Splunk can detect suspicious behavior.                                                      |
| **Spare Laptop**               | Secondary host machine      | Dedicated to running the victim-side virtual machine. Using a separate physical laptop helps isolate the victim environment from the attacker environment and makes the lab more stable when both systems are running at the same time.                                                                |
| **Ubuntu Server VM**           | Victim endpoint             | Acts as the target system in the lab. This machine generates authentication logs, system logs, and other endpoint activity that can be monitored by Splunk. It is the system being observed during simulated attacks.                                                                                  |
| **Splunk Universal Forwarder** | Log forwarding agent        | Installed inside the Ubuntu Server VM to collect local logs and securely forward them to Splunk Enterprise. It is responsible for sending endpoint data from the victim machine to the SIEM for centralized analysis.                                                                                  |
| **Mobile Hotspot Phone**       | Network provider            | Provides the wireless network connection that links both laptops and their virtual machines. The phone hotspot acts as the temporary local network for the lab, allowing the attacker machine, victim machine, and Splunk server to communicate without requiring a home router or enterprise network. |

## Device Roles in Detail

### Main Laptop

The main laptop is the control center of the lab. It is used to run the security monitoring tools and the attacker simulation environment. Because it hosts Splunk Enterprise, it is responsible for receiving and analyzing logs from the victim machine. It also runs the Kali Linux VM, which is used to generate attack traffic and test detection capabilities. This device is the primary workstation for interacting with the lab.

### Spare Laptop

The spare laptop is dedicated to the victim environment. It runs the Ubuntu Server VM, which acts as the endpoint being monitored. Separating the victim machine onto a second physical laptop helps reduce performance issues and keeps the lab organized. It also makes it easier to simulate a more realistic environment where the attacker and victim are on different systems.

### Ubuntu Server VM

The Ubuntu Server VM is the target endpoint in the lab. It generates logs related to user authentication, system activity, service behavior, and other security-relevant events. These logs are important because they provide the raw data that Splunk will analyze. The Ubuntu VM is the machine that will be scanned, probed, and monitored during the lab exercises.

### Kali Linux VM

The Kali Linux VM is the attacker simulation platform. It is used to perform actions such as Nmap scans, service discovery, and other reconnaissance techniques. The purpose of this VM is not to cause damage, but to create realistic security events that can be detected and investigated in Splunk. It represents the adversary side of the lab.

### Splunk Enterprise

Splunk Enterprise is the SIEM platform at the center of the lab. It collects logs from the Ubuntu Server VM through the Splunk Universal Forwarder, indexes the data, and makes it searchable. This allows the lab to move beyond simple attack simulation and into actual security monitoring and analysis. Splunk is where detections, dashboards, and investigations will be performed.

### Splunk Universal Forwarder

The Splunk Universal Forwarder is a lightweight agent installed on the Ubuntu Server VM. Its job is to collect logs from the local system and send them to Splunk Enterprise. It does not analyze the logs itself; instead, it acts as the delivery mechanism that moves endpoint data from the victim machine to the SIEM.

### Mobile Hotspot Phone

The mobile hotspot phone provides the network connection for the entire lab. Since both laptops are connected through the hotspot, it acts as the temporary local network that allows communication between the attacker machine, the victim machine, and the Splunk server. This is especially useful when a wired router or stable home network is not available. Because hotspot IP addresses can change, the lab may require occasional network reconfiguration.

## Expected Workflow

1. The mobile hotspot phone provides network connectivity for both laptops and their virtual machines.
2. The main laptop runs Splunk Enterprise and the Kali Linux VM.
3. The spare laptop runs the Ubuntu Server VM, which acts as the victim endpoint.
4. The Splunk Universal Forwarder inside the Ubuntu VM collects local logs.
5. The forwarder sends those logs to Splunk Enterprise on the main laptop.
6. Kali Linux performs simulated attacker activity against the Ubuntu Server.
7. The Ubuntu Server records the resulting security-related events.
8. Splunk indexes the logs and makes them available for searching and investigation.
9. The activity is analyzed to determine whether the simulated attack can be detected and understood through SIEM monitoring





# Step 2 – Environment Preparation

## Software Used

| Software                   | Purpose                                             |
| -------------------------- | --------------------------------------------------- |
| Windows 11                 | Host operating system                               |
| VirtualBox                 | Virtualization platform                             |
| Kali Linux                 | Attack simulation                                   |
| Ubuntu Server 24.04 LTS    | Victim endpoint                                     |
| Splunk Enterprise          | Security Information and Event Management (SIEM)    |
| Splunk Universal Forwarder | Log forwarding agent installed on the Ubuntu Server |

## Network Configuration

* Both physical laptops are connected to the same local hotspot network.
* Virtual machines use a **Bridged Adapter** to obtain their own IP addresses.
* The Ubuntu Server is reachable from the Kali VM for attack simulation.
* Splunk Enterprise receives logs over the local network.

## Current Progress

* VirtualBox installed on both laptops.
* Kali Linux configured and operational.
* Ubuntu Server installed on the spare laptop.
* Splunk Enterprise configured on the main laptop.
* Splunk Universal Forwarder downloaded and ready for installation.
* Network connectivity between both laptops verified.


# Step 3 – Configuring Centralized Log Collection

## Objective

Configure the Ubuntu Server VM to securely forward system logs to Splunk Enterprise using the Splunk Universal Forwarder. This establishes the centralized logging pipeline that will be used throughout the SOC homelab.

## Installation Process

The Splunk Universal Forwarder for Linux (64-bit Debian package) was downloaded from the official Splunk website and installed on the Ubuntu Server VM.

After installation, the forwarder service was started and configured to automatically start during system boot.

The following log sources were configured for monitoring:

* `/var/log/syslog`
* `/var/log/auth.log`
* `/var/log/kern.log`

These logs provide visibility into system events, authentication activity, and kernel-level events that are commonly investigated by SOC analysts.

## Forwarding Configuration

The Universal Forwarder was configured to send logs to the Splunk Enterprise instance running on the main laptop over TCP port **9997**.

Once configured, the monitored logs were successfully forwarded and indexed by Splunk Enterprise.

---

## Troubleshooting

Several issues were encountered during the initial configuration process.

### Issue 1 – Forward Server Inactive

The Splunk Universal Forwarder successfully monitored the configured log files, but the configured forward server appeared as **Inactive**.

**Investigation**

* Verified that the monitored log sources were correctly configured.
* Confirmed that the forward server IP address and TCP port were correctly specified.
* Re-added the Splunk server as the forwarding destination.
* Verified that the receiving port (9997) was enabled within Splunk Enterprise.

### Issue 2 – Network Connectivity Test

Connectivity testing showed that the Ubuntu VM could successfully reach the Splunk server using ICMP (`ping`).

However, testing TCP connectivity to port **9997** resulted in a timeout.

This indicated that the network path existed, but TCP traffic was being blocked before reaching Splunk Enterprise.

### Issue 3 – Windows Defender Firewall

Further investigation identified Windows Defender Firewall on the main laptop as the root cause.

Although Splunk Enterprise was actively listening on TCP port **9997**, inbound connections from the Ubuntu VM were blocked.

After temporarily disabling the firewall, the Universal Forwarder immediately established an active connection and log forwarding began successfully.

## Result

Figure 1. Successful Log Ingestion from Ubuntu Server to Splunk Enterprise
Splunk Enterprise successfully receives and indexes system logs forwarded by the Splunk Universal Forwarder running on the Ubuntu Server VM.
<img width="1021" height="728" alt="image" src="https://github.com/user-attachments/assets/f089f632-cfe3-4c3a-b25e-7f90dbc86720" />
<img width="3120" height="4160" alt="75f07492-2e3b-4628-b80d-4bcf570306be" src="https://github.com/user-attachments/assets/2007cfb2-4081-4563-a186-7157ee1d7b56" />

The centralized logging pipeline is now fully operational.

Ubuntu Server continuously forwards endpoint logs to Splunk Enterprise, providing live visibility into authentication events, system activity, and future attack simulations.

This configuration serves as the foundation for the remaining SOC detection and investigation scenarios in this homelab.






