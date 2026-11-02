
## Architecture logique

```mermaid
flowchart TB
    Kali["Kali Linux<br/>192.168.100.10"]

    subgraph WAN["ZONE WAN — 192.168.100.0/24"]
        K["Kali Linux<br/>192.168.100.10"]
        F1["FortiGate port1<br/>192.168.100.254"]
    end

    subgraph DMZ["ZONE DMZ — 10.10.10.0/24"]
        F2["FortiGate port2<br/>10.10.10.1"]
        WAF["Ubuntu Server<br/>Nginx + ModSecurity<br/>10.10.10.10"]
    end

    subgraph APP["ZONE APP — 10.10.20.0/24"]
        F3["FortiGate port3<br/>10.10.20.1"]
        DVWA["Ubuntu Server / Docker<br/>DVWA + MariaDB<br/>10.10.20.10"]
    end

    subgraph MGMT["ZONE MGMT — 10.10.30.0/24"]
        F4["FortiGate port4<br/>10.10.30.1"]
        WAZUH["Wazuh<br/>10.10.30.10"]
        ADMIN["Poste d'administration<br/>10.10.30.100"]
    end

    K --- F1
    F1 --- F2
    F2 --- WAF
    F1 --- F3
    F3 --- DVWA
    F1 --- F4
    F4 --- WAZUH
    F4 --- ADMIN

    WAF -->|"Requêtes applicatives"| DVWA
    F1 -.->|"Logs"| WAZUH
    WAF -.->|"Logs"| WAZUH
```

## Plan d’adressage

| Zone | Réseau | Équipement | Interface | Adresse IP |
|---|---|---|---|---|
| WAN | 192.168.100.0/24 | Kali Linux | NIC 1 | 192.168.100.10 |
| WAN | 192.168.100.0/24 | FortiGate-VM | port1 | 192.168.100.254 |
| DMZ | 10.10.10.0/24 | FortiGate-VM | port2 | 10.10.10.1 |
| DMZ | 10.10.10.0/24 | Serveur WAF | NIC 1 | 10.10.10.10 |
| APP | 10.10.20.0/24 | FortiGate-VM | port3 | 10.10.20.1 |
| APP | 10.10.20.0/24 | Serveur DVWA + MariaDB | NIC 1 | 10.10.20.10 |
| MGMT | 10.10.30.0/24 | FortiGate-VM | port4 | 10.10.30.1 |
| MGMT | 10.10.30.0/24 | Wazuh | NIC 1 | 10.10.30.10 |
| MGMT | 10.10.30.0/24 | Poste d'administration | NIC 1 | 10.10.30.100 |

## Principes de segmentation

- Chaque zone utilise un réseau virtuel isolé distinct.
- Les quatre interfaces du FortiGate relient les zones et assurent le routage inter-zone.
- Le serveur DVWA est placé dans la zone APP et ne doit pas être directement exposé au WAN.
- Le serveur WAF est placé dans la DMZ et sert de frontal à DVWA.
- Wazuh et le poste d'administration sont placés dans la zone MGMT.
- Les sous-réseaux ne se chevauchent pas.

## Points à confirmer avant validation

- Le nom exact des réseaux virtuels dans VMware ou VirtualBox.
- Le mode d'adressage statique des interfaces des VM.
- La passerelle par défaut de Kali et des serveurs internes : l'adresse FortiGate de leur zone est proposée comme passerelle.

