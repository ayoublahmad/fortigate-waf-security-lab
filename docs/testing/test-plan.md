# US-03 — Security Test Plan and Payload Set

## 1. Objective

Evaluate the effectiveness of the security controls protecting the DVWA application by executing security tests under three protection levels:

1. **Baseline:** No protection.
2. **FortiGate only:** FortiGate firewall enabled.
3. **FortiGate + WAF:** FortiGate firewall and Web Application Firewall enabled.

The tests must be performed only in the authorized, isolated lab environment.

## 2. Test Scenarios

| ID | Test scenario | Tool | Payload / procedure | Expected result with FortiGate + WAF |
|---|---|---|---|---|
| TA-01 | Port and service discovery | Nmap | `nmap -sS -sV <TARGET_IP>` | Only authorized ports are reachable. Scan activity is logged, and service banners are reduced. |
| TA-02 | Web server vulnerability scan | Nikto | `nikto -h <TARGET_URL>` | Suspicious requests are blocked where applicable, typically with HTTP 403, and logged by the WAF. |
| TA-03 | Manual SQL injection | Burp Suite Repeater | `' OR '1'='1` | The WAF blocks the request, returns HTTP 403, and logs the matching rule. |
| TA-04 | Automated SQL injection | sqlmap | `sqlmap -u "<DVWA_URL>" --cookie="<SESSION_COOKIE>" --batch` | The WAF blocks injection attempts and prevents database extraction. |
| TA-05 | Time-based blind SQL injection | sqlmap / Burp Suite | `1' AND SLEEP(5)-- -` | The WAF blocks the request; the injected delay should not be observed. |
| TA-06 | Reflected XSS | Browser / Burp Suite | `<script>alert(1)</script>` | The WAF blocks the request, typically with HTTP 403. The script does not execute. |
| TA-07 | Stored XSS | Browser / Burp Suite | Submit `<script>alert(1)</script>` in the guestbook | The WAF blocks the submission. The malicious script is not stored or executed. |
| TA-08 | Login brute force | Hydra | Attempt multiple passwords against the DVWA login form | Attempts are blocked or throttled according to the configured rate limit. WAF logs are generated. |
| TA-09 | Login brute force using Intruder | Burp Suite Intruder | Send a password list against the login form | Attempts are blocked or throttled according to the configured rate limit. |
| TA-10 | Direct access bypassing the WAF | Browser / curl | Attempt to access DVWA directly from the WAN instead of through the WAF | FortiGate denies direct WAN-to-APP access and logs the denied traffic. |
| TA-11 | Network segmentation validation | ping / SSH / HTTPS | From WAN, attempt to reach APP and MGMT services | Unauthorized connections are denied and logged by FortiGate. |
| TA-12 | Legitimate traffic validation | Browser / curl | Perform 20 normal requests: navigate, log in, and submit forms | At least 19 of 20 requests succeed. False-positive rate is no greater than 5%. |
| TA-13 | Command injection | DVWA “Command Injection” module | `; id` | The WAF blocks the request before the command executes. |
| TA-14 | File inclusion | DVWA “File Inclusion” module | `../../../../etc/passwd` | The WAF blocks the request and prevents file disclosure. |

## 3. Execution Procedure

For each test scenario:

1. Configure the lab for the required protection level.
2. Record the target IP address or URL, tool, and tool version.
3. Execute the test using the specified payload or procedure.
4. Record the observed result and HTTP response code, where applicable.
5. Check FortiGate and WAF logs for corresponding events.
6. Save screenshots or other evidence.
7. Repeat the same test under the other protection levels.
8. Compare the results across all three levels.

## 4. Test Results Template

| Field | Value |
|---|---|
| Test ID | |
| Date and time | |
| Protection level | Baseline / FortiGate / FortiGate + WAF |
| Target | |
| Tool and version | |
| Payload / procedure | |
| Expected result | |
| Observed result | |
| HTTP response code | |
| Attack blocked? | Yes / No / Partial |
| Related log found? | Yes / No |
| WAF rule ID | |
| Evidence reference | |
| Verdict | Pass / Fail |
| Comments | |

## 5. Acceptance Criteria

- Unauthorized WAN-to-APP and WAN-to-MGMT traffic is denied and logged.
- The WAF blocks the tested SQL injection, XSS, command injection, and file inclusion payloads.
- Brute-force attempts are blocked or throttled according to the configured policy.
- Legitimate traffic achieves at least 95% success, with a false-positive rate no greater than 5%.
- Test outcomes and corresponding security logs are documented for all three protection levels.

**Note:** Expected results are hypotheses to be verified during execution. Record actual observations rather than assuming that a control worked.

