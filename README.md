# Zabbix Cisco Catalyst 9300 "Monster" Template

![Zabbix Version](https://img.shields.io/badge/Zabbix-7.4%2B-red) ![Cisco IOS-XE](https://img.shields.io/badge/Cisco-IOS--XE-blue) ![Method](https://img.shields.io/badge/Method-Hybrid%20(SNMP%20%2B%20SSH)-green)

Advanced monitoring template for **Cisco Catalyst 9300 (IOS-XE)** switches using Zabbix 7.4+.
This template goes beyond standard SNMP monitoring by utilizing **SSH-based data collection** to retrieve deep "Device Tracking" (SISF) information that is often missing or incomplete in standard SNMP MIBs on modern IOS-XE versions.

## 🚀 Key Features

### 1. 🕵️‍♂️ Advanced Device Tracking (SSH-based)
Standard SNMP (`IP-MIB` or `CISCO-IP-DEVICE-TRACKING-MIB`) often fails to report all connected devices on newer IOS-XE versions. This template uses a secure SSH connection to parse the `show device-tracking database` CLI output directly.
* **Complete Visibility:** Sees ALL connected endpoints (DHCP Snooping, ARP, ND, Static).
* **Accuracy:** Successfully retrieves 160+ entries where SNMP typically shows only ~9.

### 2. 🗺️ Port-to-IP Mapping (Neighbor Discovery)
Automatically discovers active ports and lists the connected IP addresses directly in the interface statistics, sorted alphabetically with other interface metrics.
* **Item Name:** `Interface Gi1/0/x: Neighbor IP Address(es)`
* **Value:** `IPv4: 192.168.1.50 | IPv6: 2001:db8::50`
* **Tags:** Automatically tagged with `Monitoring: IP Monitor` and `Interface: <Name>` for easy filtering.
* No need to cross-reference MAC address tables manually!

### 3. 🚨 IP Conflict Monitor (The "Police")
A built-in logic that analyzes the entire Device Tracking database for inconsistencies.
* **Detects:** Duplicate IP addresses associated with different MAC addresses.
* **Smart IPv6 Filtering:** Ignores Link-Local (`fe80::`) addresses to prevent false positives, focusing only on Global Unicast / ULA conflicts.
* **Trigger:** Fires a `HIGH` severity alert immediately upon detecting a conflict (e.g., *"CONFLICT: 192.168.1.10 [MAC1 vs MAC2]"*).

### 4. 📊 Standard Hardware Health (SNMP)
Includes all the essentials:
* CPU & Memory usage.
* Temperature & Fan status.
* Stack Ring Redundancy & Member monitoring.
* Interface traffic & status.

---

## 🛠️ Prerequisites

1.  **Zabbix Server:** Version **7.4** or higher (required for the new import format and UUIDs).
2.  **Configuration:** The Zabbix Server/Proxy must have `Timeout` set to at least **30 seconds** in `zabbix_server.conf`.
    * *Reason:* SSH commands and parsing large tables on the switch can take time.
3.  **Switch Config:**
    * SNMP enabled (RO).
    * SSH enabled.
    * `device-tracking` (formerly IPDT/SISF) feature enabled on the switch ports.

## ⚙️ Installation & Configuration

### 1. Import the Template
Download the `.json` file from this repository and import it into Zabbix (`Data collection` -> `Templates` -> `Import`).

### 2. Set Macros
Link the template to your Cisco 9300 Host and set the following **Inherited Macros**:

| Macro | Description | Type |
| :--- | :--- | :--- |
| `{$SNMP_COMMUNITY}` | SNMP Read Community | Text |
| `{$SSH_USER}` | Username for SSH Login | Text |
| `{$SSH_PASSWORD}` | Password for SSH Login | **Secret Text** |

### 3. Verify Data
Go to `Monitoring` -> `Latest Data` and filter by the tag **Monitoring: IP Monitor** or **Application: Zabbix Raw Data**.
* Check `SSH: Device Tracking Database (CLI)`: Should contain a valid JSON string.
* Check `Net: IP Conflict Monitor`: Should report "OK".
* Check `Interface ... : Neighbor IP Address(es)`: Should list IPs for active ports.

---

## 🧠 How it Works (Under the Hood)

### Hybrid Data Collection
1.  **Master Item (SSH):** The template logs into the switch via SSH and executes:
    ```bash
    show device-tracking database | exclude ^$
    ```
2.  **JavaScript Preprocessing:** Zabbix parses the raw CLI text output into a structured JSON object containing IP, MAC, VLAN, Interface, and State.
3.  **Low-Level Discovery (LLD):**
    * The **Active Ports Discovery** rule iterates through this JSON.
    * It creates dependent items for every active interface (e.g., `Gi1/0/1`).
    * It populates these items with the connected IPv4/IPv6 addresses.
4.  **Conflict Logic:** A separate dependent item scans the JSON for any IP keys that have multiple distinct MAC addresses assigned to them.

---

## 🚧 Roadmap / To-Do
* **QoS Discovery:** Parsing `show policy-map interface` to visualize dropped packets and class-map statistics.
* **PoE Monitoring:** Advanced power budget detailed stats.

## 🤝 Contributing
This template was forged in the fires of debugging IOS-XE SNMP limitations. Pull requests and improvements are welcome!

---
**Created by:** [Vendel Cseh] 

If you like the project, support it with PayPal (csehvendel@gmail.com) because a lot of coffee is being sold while I'm grinding this...
