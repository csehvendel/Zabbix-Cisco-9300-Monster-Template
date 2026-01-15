Zabbix Cisco 9300 Monster Template
A professional-grade, high-density Zabbix template specifically engineered for Cisco Catalyst 9300 Series switches. This template provides deep visibility into hardware health, network security, configuration compliance, and real-time SLA performance.

🚀 Key Features
📍 Automated Geolocation & Inventory
The template automatically populates Zabbix Inventory (City, Building, Room) and Geomap (Latitude/Longitude) by parsing the switch's location string. To enable this, configure your switch location using the following syntax:

snmp-server location C:City B:Building R:Room LAT:X.Y LONG:X.Y


📉 Advanced IP SLA Monitoring
Comprehensive monitoring for network quality and latency. The template discovers and tracks:

ICMP & UDP Jitter: Track jitter, latency, and packet loss.

TCP & UDP Echo: Monitor service availability.

DNS & HTTP Checks: Real-time application response time tracking.

🛡️ Security & Conflict Tracking
ARP/NDP Conflict Detection: Identifies IP-MAC address collisions using JavaScript preprocessing.

Security Compliance: Monitors the status of DHCP Snooping and IP Device Tracking.

⚙️ Configuration & Hardware Health
Unsaved Changes Alert: Triggers a HIGH priority alert if the running-config differs from the startup-config (reminds you to write memory).

Full Stack Discovery: Monitors stack ring redundancy, member counts, and serial numbers.

Smart PSU Monitoring: Automatically identifies PSU models and extracts their rated wattage.

Environmentals: Real-time status for all fans and transceiver (SFP) optical levels (dBm).

🛠️ Configuration (Cisco CLI)
To get the most out of this template, apply the following configuration to your Cisco 9300 switches:

1. SLA Operations

ip sla 10
 udp-jitter ip.ip.ip.ip 5000 source-port 4000
 tag udp-jitter
 threshold 200
 frequency 45
ip sla schedule 10 life forever start-time now
ip sla 20
 icmp-echo ip.ip.ip.ip
 tag icmp-echo
 threshold 1
 frequency 30
ip sla schedule 20 life forever start-time now
ip sla 30
 tcp-connect ip.ip.ip.ip 5000 source-port 4000
 tag tcp-connect
 frequency 120
 timeout 60
 threshold 60
ip sla schedule 30 life forever start-time now
ip sla 40
 udp-echo ip.ip.ip.ip 5001
 tag udp-echo
 threshold 200
 frequency 45
ip sla schedule 40 life forever start-time now
ip sla 50
 icmp-jitter ip.ip.ip.ip
 tag icmp-jitter
 frequency 30
ip sla schedule 50 life forever start-time now
ip sla 60
 dns example.com name-server ip.ip.ip.ip
 tag dns-check
 threshold 30
ip sla schedule 60 life forever start-time now
ip sla 70
 http get http://example.com
 tag http-check
 threshold 30
ip sla schedule 70 life forever start-time now
ip sla enable reaction-alerts

2. Global Settings

# Set location for Inventory and Geomap
snmp-server location C:[City] B:[Building] R:[Room] LAT:[Latitude] LONG:[Longitude]

# Enable CDP/LLDP for neighbor discovery
cdp run
lldp run


📥 Installation
Import zbx_export_templates.yaml into Zabbix (Data collection > Templates).

Link the "Cisco Catalyst 9300 Switch" template to your host.

Ensure SNMP v2c/v3 is configured on the switch and the community string matches your Zabbix macro.

Set the Host Inventory mode to "Automatic" in Zabbix.

☕ Support
If you find this template useful, consider supporting the development:

PayPal: csehvendel@gmail.com

Developed by csehvendel - Requires Zabbix 7.4+
