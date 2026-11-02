
## Logical Architecture

```mermaid
flowchart TB
    Kali["Kali Linux<br/>192.168.100.10"]

    subgraph WAN["WAN ZONE — 192.168.100.0/24"]
        K["Kali Linux<br/>192.168.100.10"]
        F1["FortiGate port1<br/>192.168.100.254"]
    end

    subgraph DMZ["DMZ ZONE — 10.10.10.0/24"]
        F2["FortiGate port2<br/>10.10.10.1"]
        WAF["Ubuntu Server<br/>Nginx + ModSecurity<br/>10.10.10.10"]
    end

    subgraph APP["APP ZONE — 10.10.20.0/24"]
        F3["FortiGate port3<br/>10.10.20.1"]
        DVWA["Ubuntu Server / Docker<br/>DVWA + MariaDB<br/>10.10.20.10"]
    end

    subgraph MGMT["MGMT ZONE — 10.10.30.0/24"]
        F4["FortiGate port4<br/>10.10.30.1"]
        WAZUH["Wazuh<br/>10.10.30.10"]
        ADMIN["Admin Workstation<br/>10.10.30.100"]
    end

    K --- F1
    F1 --- F2
    F2 --- WAF
    F1 --- F3
    F3 --- DVWA
    F1 --- F4
    F4 --- WAZUH
    F4 --- ADMIN

    WAF -->|"Application Requests"| DVWA
    F1 -.->|"Logs"| WAZUH
    WAF -.->|"Logs"| WAZUH
```

## Addressing Plan

| Zone | Network | Device | Interface | IP Address |
|---|---|---|---|---|
| WAN | 192.168.100.0/24 | Kali Linux | NIC 1 | 192.168.100.10 |
| WAN | 192.168.100.0/24 | FortiGate-VM | port1 | 192.168.100.254 |
| DMZ | 10.10.10.0/24 | FortiGate-VM | port2 | 10.10.10.1 |
| DMZ | 10.10.10.0/24 | WAF Server | NIC 1 | 10.10.10.10 |
| APP | 10.10.20.0/24 | FortiGate-VM | port3 | 10.10.20.1 |
| APP | 10.10.20.0/24 | DVWA + MariaDB Server | NIC 1 | 10.10.20.10 |
| MGMT | 10.10.30.0/24 | FortiGate-VM | port4 | 10.10.30.1 |
| MGMT | 10.10.30.0/24 | Wazuh | NIC 1 | 10.10.30.10 |
| MGMT | 10.10.30.0/24 | Admin Workstation | NIC 1 | 10.10.30.100 |

## Segmentation Principles

- Each zone uses a separate isolated virtual network.
- The four FortiGate interfaces connect the zones and provide inter-zone routing.
- The DVWA server is located in the APP zone and must not be directly exposed to the WAN.
- The WAF server is located in the DMZ and acts as the frontend for DVWA.
- Wazuh and the admin workstation are located in the MGMT zone.
- The subnets do not overlap.

## Items to Confirm Before Validation

- The exact names of the virtual networks in VMware or VirtualBox.
- The static addressing method for the VM interfaces.
- The default gateway for Kali and the internal servers: the FortiGate address for their zone is proposed as the gateway.
