# US-02 — Traffic Matrix and Target Security Policy


**Default policy:** All inter-zone traffic that is not explicitly authorized is **denied** and **logged**. Responses to authorized connections are allowed by FortiGate state tracking.


## 2. Authorized Traffic Matrix

| ID | Source | Destination | Service / Port | Action | Justification | Logging |
| :--- | :--- | :--- | :--- | :---: | :--- | :---: |
| **F-01** | Kali `192.168.100.10` | FortiGate VIP (WAF publication) | HTTP / TCP 80 | Allow | Web access to the lab from the attacker network | Yes |
| **F-02** | Kali `192.168.100.10` | FortiGate VIP (WAF publication) | HTTPS / TCP 443 | Allow | Encrypted web access to the lab, if HTTPS is configured | Yes |
| **F-03** | FortiGate (VIP) | WAF `10.10.10.10` | HTTP / TCP 80 | Allow | Forward published requests to the reverse proxy | Yes |
| **F-04** | WAF `10.10.10.10` | DVWA `10.10.20.10` | HTTP / TCP 80 | Allow | The reverse proxy forwards authorized requests to the application | Yes |
| **F-05** | FortiGate `10.10.10.1` | Wazuh `10.10.30.10` | Syslog / UDP 514 | Allow | Centralize firewall logs | Yes |
| **F-06** | WAF `10.10.10.10` | Wazuh `10.10.30.10` | Syslog / UDP 514 | Allow | Centralize WAF logs | Yes |
| **F-07** | Admin workstation `10.10.30.100` | FortiGate `10.10.30.1` | HTTPS / TCP 443 | Allow | Manage FortiGate through its web interface | Yes |
| **F-08** | Admin workstation `10.10.30.100` | WAF `10.10.10.10` | SSH / TCP 22 | Allow | Administer the WAF server | Yes |
| **F-09** | Admin workstation `10.10.30.100` | DVWA `10.10.20.10` | SSH / TCP 22 | Allow | Administer the server hosting DVWA | Yes |
| **F-10** | Admin workstation `10.10.30.100` | Wazuh `10.10.30.10` | HTTPS / TCP 443 | Allow | Access the Wazuh dashboard | Yes |
| **F-11** | Kali `192.168.100.10` | FortiGate `192.168.100.254` | ICMP Echo | Allow | Verify basic lab connectivity | Yes |

### Notes:
* The VIP and its exact configuration will be defined during FortiGate configuration.
* **F-02** should be enabled when frontend HTTPS is configured.
* **F-08** and **F-09** are administrative flows; they should only be allowed from the admin workstation.
* The listed ports are the proposed destination ports. They must be verified during deployment.

---

## 3. Explicitly Denied Traffic

| ID | Source | Destination | Service | Action | Reason |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **D-01** | WAN / Kali | DVWA `10.10.20.10` | Any | Deny | Prevent direct access to the application |
| **D-02** | WAN / Kali | Wazuh `10.10.30.10` | Any | Deny | Protect monitoring systems |
| **D-03** | WAN / Kali | Admin workstation `10.10.30.100` | Any | Deny | Protect administration systems |
| **D-04** | DMZ / WAF | MGMT, except Wazuh `UDP 514` | Any | Deny | Limit WAF communications |
| **D-05** | APP / DVWA | WAN | Any | Deny | Prevent unnecessary outbound connections |
| **D-06** | APP / DVWA | MGMT, except explicitly added flows | Any | Deny | Isolate the application from the management zone |
| **D-07** | Any zone | Any other unauthorized destination | Any | Deny | Apply the default-deny policy |

---

## 4. Target FortiGate Policy

1. Create one policy for each authorized flow, with a precise source, destination, and service.
2. Avoid `ALL` services and `ALL` destinations in authorization rules.
3. Enable logging for relevant authorized sessions.
4. Enable logging for denied traffic in the final deny rule.
5. Place specific rules before the final deny rule.
6. Verify the rules with positive and negative tests.

---

## 5. Validation

- [x] Each authorized flow has a justification and a precise service.
- [x] The default-deny policy is explicit and logged.
- [x] Denied flows have been tested.
- [x] The matrix has been reviewed and validated before phase 3.