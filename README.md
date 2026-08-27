# 🦖 Zabbix Monster Templates

**Very detailed, field-tested Zabbix templates for network devices.**

This collection aims to fill the gaps left by standard "out-of-the-box" templates. Our goal was to create a monitoring solution that sees **everything**: from hidden hardware faults and precise interface speeds to complex IP SLA performance metrics and vendor-specific diagnostics.

> **Status:** Production Ready 🟢  
> **Zabbix Version:** 7.0+ (Recommended: 7.4)

---

## 📦 Supported Devices

The repository currently contains optimized templates for the following hardware:

### 🔵 Ubiquiti UniFi Series (Monster Edition)
*Direct system querying (`mca-dump`) for deep diagnostics with advanced JS parsing.*

| Device Model | Architecture | Key Features Enabled |
| :--- | :--- | :--- |
| **U6 Enterprise** | ARM / Qualcomm | WiFi 6E (6GHz), VAP Fallback, Anomaly Detection |
| **U6 Pro** | ARM / Qualcomm | WiFi 6, VAP Fallback, Client Distribution |
| **U6 In-Wall** | ARM / MediaTek | **Hybrid Tracking** (Wired + Wireless), 4-Port Switch Mon. |
| **IW-HD** | MIPS / MediaTek | **Hybrid Tracking**, PoE Passthrough, Port Stats |
| **UAP-AC-Pro** | MIPS / Qualcomm | Legacy Support, Anomaly Mining, LLDP Fixes |
| **UAP-AC-LR** | MIPS / Qualcomm | Interface-based Discovery, Long Range Radio stats |

### 🟢 Cisco Catalyst Series (Monster Edition)
*Deep SNMP monitoring with IP SLA and Stack support.*
- **Cisco Catalyst 9200 / 9200L**
- **Cisco Catalyst 9300 / 9300L**
- **Cisco Catalyst 9500**

---

## 🔥 Universal "Monster" Features
**Capabilities available across all templates in this repository:**

* **Mirrored Traffic Graphs:** Inbound traffic is visualized in the negative range (inverted) while outbound is positive. This allows for an instant, clean visual overview of traffic flow on graphs.
* **Automatic Map Coordinates:** The templates automatically extract GPS coordinates (`system.location` or SNMP) and populate the Zabbix Host Inventory fields. This enables automatic placement on **Geomaps**.

---

## 🧠 Deep Dive: Ubiquiti UniFi Logic

Standard SNMP templates often fail on modern UniFi firmware. These templates use a **Single-Item-Master** approach (SSH `mca-dump`) with advanced JavaScript preprocessing to extract everything in one go.

### 1. Universal Radio Analytics (VAP Fallback)
* **The Problem:** Newer firmware often reports `0` or `null` for Channel and TX Power in the standard `radio_table`.
* **The Solution:** These templates automatically fallback to scanning the Virtual Access Point (VAP) table to extract real-world EIRP, Channel Utilization, and Operating Channels.

### 2. Hybrid Client Tracking (Wired + Wireless)
* **Target:** In-Wall units (U6-IW, IW-HD).
* **The Feature:** Automatically discovers clients plugged into the physical switch ports **AND** WiFi clients.
* **Result:** A unified `System: MAC Address Table Size` count (Port + WLAN) and unified tracking list.

### 3. Advanced Anomaly Detection
* Mines deep JSON data to find hidden issues:
    * DHCP Timeouts & DNS Latency
    * ARP Timeouts
    * High WiFi Retries & Packet Drops

### 4. Robust Discovery
* **LLDP 2.0:** Handles data type mismatches (string vs int) to reliably find neighbor switch information even on Legacy MIPS devices.
* **Conflict Detection:** Built-in JS logic to identify IP conflicts and MAC loops across the AP's bridge.

---

## 🧠 Deep Dive: Cisco Catalyst Logic

### 1. Stack-Aware Monitoring
The templates automatically detect if the switch is part of a **StackWise** configuration.
* **Stack Ring Speed:** Monitors the backplane bandwidth (e.g., 480Gbps).
* **Member Status:** Alerts if a stack member goes offline or changes priority.
* **Individual CPU/Mem:** Monitors resources for *each* switch in the stack, not just the Master.

### 2. IP SLA (Service Level Agreement)
Automatically discovers configured IP SLA probes on the switch.
* **RTT (Round Trip Time):** Measures latency to target.
* **Jitter:** Monitors voice quality metrics (source-to-dest and dest-to-source).
* **Packet Loss:** Tracks lost packets in real-time.
* *Requirement:* The IP SLA must be configured and active on the switch (IOS-XE).

### 3. SFP & Transceiver Health
Reads the DOM (Digital Optical Monitoring) data from SFP/SFP+ modules.
* **RX/TX Power (dBm):** Alerts on low signal levels (fiber cuts/bends).
* **Temperature & Voltage:** Hardware health monitoring.
* **Serial Number Tracking:** Automatically inventories connected optics.

### 4. PoE (Power over Ethernet)
* **Total Power Budget:** Visualizes available vs. used power.
* **Port-Level PoE:** Tracks power consumption per port (useful for identifying power-hungry APs or cameras).

---

### Installation Steps

1.  Download the `.yaml` or `.json` files.
2.  Import them into Zabbix (**Configuration** -> **Templates** -> **Import**).
3.  Assign the appropriate template to your Host.
4.  Wait for **Discovery** to run (Standard LLDP/VAP/Interface discovery). ☕

## 🛠️ Setup Guide (Step-by-Step)

So easy, even your grandma could do it! 🍪

### 1. Zabbix Prerequisites
- Ensure you are running **Zabbix 7.0** or newer.
- Download the `.yaml` files from this repository.
- In Zabbix: Go to **Data collection** -> **Templates** -> **Import**.
- Select all options (Groups, Templates, Triggers, Graphs, etc.) and click **Import**.

### 2. Device Preparation

#### 🔵 Ubiquiti Devices (Enable SSH)
Since these templates fetch data via SSH, you must enable it in the Controller:
1. Log in to your **UniFi Network Application**.
2. Go to **Settings** -> **System** -> **Advanced**.
3. Enable **Device SSH Authentication**.
4. Set a **Username** and **Password** (you will need these for Zabbix).

#### 🟢 Cisco Devices (Enable SNMP & IP SLA)
Log in to the switch CLI (console/SSH).
The following configuration is **required** for the template to fully function (including Map coordinates and Performance monitoring).

**1. General & Location Configuration**
*Note: The location format (C:City B:Building R:Room LAT/LONG) is strict to allow Zabbix to extract Geolocation coordinates.*

```cisco
conf t

! Read-Only (RO) access for Zabbix
snmp-server community zabbix-read RO

! Location Format: C:City B:Building R:Room LAT:<coord> LONG:<coord>
! Example:
snmp-server location C:City B:Building R:Room LAT:COORD LONG:COORD
snmp-server contact SysAdmin
```

**2. IP SLA Configuration (Monster Level)**
*Copy-paste this block to enable full network performance monitoring (Jitter, DNS, HTTP, TCP). Replace `ip.ip.ip.ip` with your target IP.*

```cisco
! UDP Jitter (VoIP simulation)
ip sla 10
 udp-jitter ip.ip.ip.ip 5000 source-port 4000
 tag udp-jitter
 threshold 200
 frequency 45
ip sla schedule 10 life forever start-time now

! ICMP Echo (Standard Ping)
ip sla 20
 icmp-echo ip.ip.ip.ip
 tag icmp-echo
 threshold 1
 frequency 30
ip sla schedule 20 life forever start-time now

! TCP Connect (Service Availability)
ip sla 30
 tcp-connect ip.ip.ip.ip 5000 source-port 4000
 tag tcp-connect
 frequency 120
 timeout 60
 threshold 60
ip sla schedule 30 life forever start-time now

! UDP Echo
ip sla 40
 udp-echo ip.ip.ip.ip 5001
 tag udp-echo
 threshold 200
 frequency 45
ip sla schedule 40 life forever start-time now

! ICMP Jitter
ip sla 50
 icmp-jitter ip.ip.ip.ip
 tag icmp-jitter
 frequency 30
ip sla schedule 50 life forever start-time now

! DNS Resolution Check
ip sla 60
 dns uni-mate.hu name-server ip.ip.ip.ip
 tag dns-check
 threshold 30
ip sla schedule 60 life forever start-time now

! HTTP Service Check
ip sla 70
 http get http://example.com
 tag http-check
 threshold 30
ip sla schedule 70 life forever start-time now

end
wr
```

### 3. Adding a Host in Zabbix

1. Go to **Data collection** -> **Hosts** -> **Create host**.
2. **Host name:** Enter the device name.
3. **Templates:** Select the appropriate Monster Template.
4. **Interfaces:**
   - **For Cisco:** Add an **SNMP** interface (IP address, Port: 161).
   - **For Ubiquiti:** Add an **Agent** interface (IP address, Port: 10050 - *used only for SSH connectivity targeting*).
5. **Macros Tab (CRUCIAL!):**
   Fill in the credentials here.

| Type | Macro Name | Value (Example) | Description |
| :--- | :--- | :--- | :--- |
| **Generic** | `{$SNMP_COMMUNITY}` | `zabbix-read` | The SNMP community string configured on the switch. |
| **Ubiquiti** | `{$SSH_UNIFI_USER}` | `ubnt_admin` | SSH Username set in the UniFi Controller. |
| **Ubiquiti** | `{$SSH_UNIFI_PASSWORD}` | `SecretPass123` | SSH Password set in the UniFi Controller. |
| **Cisco** | `{$SSH_USER}` | `cisco_ro_user` | SSH Username. |
| **Cisco** | `{$SSH_PASSWORD}` | `SecretPass123` | SSH Password. |


6. Click **Add**.
7. Wait a few minutes for **Discovery** to run. It will automatically find ports, inventory data, and active IP SLAs. ☕

---

## 🧩 Troubleshooting

- **No data appearing for Ubiquiti?**
  - Check if the Zabbix Server/Proxy can reach the switch on port 22 (SSH).
  - Try logging in manually from the terminal: `ssh user@switch-ip`.
  - Double-check the Macros in the Host settings.

- **Cisco "Location" or Map coordinates missing?**
  - Ensure you used the strict `LAT:x.x LONG:y.y` format in `snmp-server location`.
  - Check the *Inventory* tab in the Host view to see if fields are populated.

- **Cisco IP SLA items missing?**
  - Run `show ip sla summary` on the switch to ensure the SLAs are `Active`.
  - The template creates items only for SLAs that are scheduled and active.

---

## 📜 License
These templates are free to use, modify, and distribute. If you find them useful, please star the repo AND/OR support the project via [![Donate via Revolut](https://img.shields.io/badge/Support-Revolut-0075EB?style=for-the-badge&logo=revolut&logoColor=white)](https://revolut.me/vendel_c) because a lot of coffee is wasted while grinding.
! ⭐
