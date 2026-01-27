# 🦖 Zabbix Monster Templates

**Very detailed, field-tested Zabbix templates for network devices.**

This collection aims to fill the gaps left by standard "out-of-the-box" templates. Our goal was to create a monitoring solution that sees **everything**: from hidden hardware faults and precise interface speeds to complex IP SLA performance metrics and vendor-specific diagnostics.

> **Status:** Production Ready 🟢  
> **Zabbix Version:** 7.0+ (Recommended: 7.4)

---

## 📦 Supported Devices

The repository currently contains optimized templates for the following hardware:

### 🔵 Ubiquiti UniFi Series
*Direct system querying (`mca-dump`) for deep diagnostics.*
- **USW-Aggregation** (Switch Aggregation)
- **USW-Pro-Aggregation** (Switch Pro Aggregation)

### 🟢 Cisco Catalyst Series
*Deep SNMP monitoring with IP SLA and Stack support.*
- **Cisco Catalyst 9200 / 9200L**
- **Cisco Catalyst 9300 / 9300L**
- **Cisco Catalyst 9500**

---

## 🔥 Universal "Monster" Features
**Capabilities available across all templates in this repository:**

* **Mirrored Traffic Graphs:** Inbound traffic is visualized in the negative range (inverted) while outbound is positive. This allows for an instant, clean visual overview of traffic flow on graphs.
* **Deep Inventory Collection:** Automatic gathering of Serial Numbers, Firmware versions, and Model specifics.
* **Transceiver (SFP) Diagnostics:** Detailed monitoring of Optical levels (TX/RX), Temperature, Vendor information, and Serial numbers for fiber modules.
* **High-Precision Metrics:** Optimized polling intervals for critical data points to ensure you never miss a spike or a drop.
* **Geolocation Ready:** Parses custom SNMP Location strings (containing `LAT:`/`LONG:`) to automatically place devices on Zabbix Maps.
* **Network Loop & Conflict Detection:** Instant alerts for IP or MAC conflicts detected by the switch.
* **Fan & Temperature Health:** Detailed internal sensor readings not always available via standard SNMP.

---

## 🧠 Vendor-Specific Features

### 🔵 Ubiquiti UniFi Specifics
* **Direct `mca-dump` Parsing:** Unlike standard templates, we parse the switch's internal JSON diagnostics via SSH.
* **Real-time "Satisfaction" Score:** Immediate visibility into the user experience score reported by the switch.

### 🛡️ Cisco Catalyst Specifics
* **Advanced IP SLA Monitoring:** Auto-discovery of complex SLA probes (UDP Jitter, TCP Connect, DNS, HTTP, VoIP scores) to monitor network quality beyond simple pings.
* **StackWise Monitoring:** Full visibility into Stack ring status, member availability, and stack port speeds.
* **Port Security & Err-Disable:** Alerts if a port is shut down due to security violations or loop protection.

---

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
| **Ubiquiti** | `{$SSH_UNIFI_USER}` | `ubnt_admin` | SSH Username set in the UniFi Controller. |
| **Ubiquiti** | `{$SSH_UNIFI_PASSWORD}` | `SecretPass123` | SSH Password set in the UniFi Controller. |
| **Cisco** | `{$SNMP_COMMUNITY}` | `zabbix-read` | The SNMP community string configured on the switch. |

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
These templates are free to use, modify, and distribute. If you find them useful, please star the repo AND/OR support the project via [![Donate](https://img.shields.io/badge/Donate-PayPal-blue.svg)](PayPal.Me/csehvendel
) because a lot of coffee is wasted while grinding.
! ⭐
