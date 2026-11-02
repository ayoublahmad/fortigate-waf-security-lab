# IP Addressing

## Network plan

| Zone | Network/CIDR | Gateway | Device | IP address | Interface |
|---|---|---|---|---|---|
| WAN | 192.168.100.0/24 | - | Kali Linux | 192.168.100.10 | NIC 1 |
| WAN | 192.168.100.0/24 | - | FortiGate | 192.168.100.254 | port1 |
| DMZ | 10.10.10.0/24 | 10.10.10.1 | FortiGate | 10.10.10.1 | port2 |
| DMZ | 10.10.10.0/24 | 10.10.10.1 | WAF Server | 10.10.10.10 | NIC 1 |
| APP | 10.10.20.0/24 | 10.10.20.1 | FortiGate | 10.10.20.1 | port3 |
| APP | 10.10.20.0/24 | 10.10.20.1 | DVWA Server | 10.10.20.10 | NIC 1 |
| MGMT | 10.10.30.0/24 | 10.10.30.1 | FortiGate | 10.10.30.1 | port4 |
| MGMT | 10.10.30.0/24 | 10.10.30.1 | Wazuh | 10.10.30.10 | NIC 1 |
| MGMT | 10.10.30.0/24 | 10.10.30.1 | Admin workstation | 10.10.30.100 | NIC 1 |

## Rules

- Each zone uses a separate subnet.
- The subnets do not overlap.
- The FortiGate interface is the default gateway for internal devices.
- The DVWA server is not directly accessible from the WAN.
