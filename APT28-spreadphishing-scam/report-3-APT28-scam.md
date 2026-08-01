# Phishing Analysis Report — 3
## APT28 State-Sponsored Spearphishing Against Ukrainian Government (CERT-UA#6562)

**Date Analyzed:** July 2026
**Attack Type:** State-Sponsored Spearphishing — PowerShell Execution and Data Exfiltration
**Target:** Ukrainian government bodies — multiple ministries and departments
**MITRE ATT&CK:** T1566.002 (Initial Access) + T1059.001 (Execution) + T1567 (Exfiltration)
**Threat Group:** APT28 / UAC-0028 / Fancy Bear — Russian GRU Unit 26165
**CERT-UA Advisory:** CERT-UA#6562 — April 2023
**Reference URL:** https://cert.gov.ua/article/4492467

---

## Executive Summary

In April 2023, CERT-UA documented a large-scale spearphishing campaign by the Russian
state-sponsored hacking group APT28 (tracked as UAC-0028) targeting multiple Ukrainian
government departments simultaneously. The attackers sent emails with the subject
"Windows Update" in Ukrainian, impersonating system administrators of the targeted
departments. The sender addresses were created on Microsoft Outlook using real employee
surnames obtained through prior reconnaissance, making the emails visually convincing
to recipients who recognized the names. The email body contained instructions to run
a PowerShell command described as a Windows security update. Executing this command
collected system information from the compromised machine and exfiltrated it to
mocky.io, a legitimate free API service abused as a command-and-control endpoint.
CERT-UA assessed that this campaign represents a systematic intelligence-gathering
operation by Russian military intelligence against Ukrainian infrastructure
during the active war.

This report analyzes the documented indicators and attack chain from CERT-UA#6562.
This campaign is the initial access vector that precedes credential dumping operations
of the type documented in my APT Detection Lab project.

---

## Email Details

| Field | Value |
|---|---|
| From | [surname.initials]@outlook.com |
| Example addresses | ivanov.mv@outlook.com, petrenko.oi@outlook.com |
| Subject | Windows Update (Ukrainian: "Оновлення Windows") |
| Language | Ukrainian |
| Target | Ukrainian government ministries and departments |
| Sender claims | System administrator of the victim's own department |
| Delivery method | PowerShell command instruction (T1566.002 → T1059.001) |
| Infrastructure used | Microsoft Outlook free accounts + Mocky.io API service |

---

## 1. Header Analysis

### Note on Header Availability

Full email headers for this campaign are not publicly disclosed by CERT-UA to protect
victim organizations and ongoing investigations.

### What is Known from Published Intelligence

**Authentication results (from CERT-UA technical analysis):**

| Check | Result | Explanation |
|---|---|---|
| SPF | PASS | Microsoft authorizes outlook.com to send for its accounts |
| DKIM | PASS | Microsoft signs all Outlook.com mail |
| DMARC | PASS | Microsoft's DMARC policy passes for outlook.com |

**The Key Point of the Attack:** By using legitimate Microsoft Outlook free accounts
as the sender, APT28 ensured all authentication checks pass. There is no malicious
infrastructure to detect at the email authentication level. Standard email security
controls are blind to this attack because the email is technically legitimate as far
as authentication is concerned.

---

## 2. Domain Investigation

### Sender Infrastructure: outlook.com

**Key finding:** outlook.com is a legitimate Microsoft-owned domain registered in 1994.
It passes all security checks because it is a legitimate domain. 

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/d06de78742548d95aca52f4fbf08f7554aef4857/APT28-spreadphishing-scam/screenshots-APT28/whois_domain_check.png

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/d06de78742548d95aca52f4fbf08f7554aef4857/APT28-spreadphishing-scam/screenshots-APT28/icann_lookup.png

### Exfiltration Infrastructure: mocky.io

**Key insight:** Mocky.io is a legitimate service for API testing. It allows anyone to
create a free endpoint that accepts and stores HTTP POST data. APT28 created a Mocky.io
endpoint and had the PowerShell payload send system data to it. The traffic from victim
machines to mocky.io looks identical to legitimate API testing traffic and is extremely effective at evading
detection.

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/d06de78742548d95aca52f4fbf08f7554aef4857/APT28-spreadphishing-scam/screenshots-APT28/whois_check_mocky.png

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/d06de78742548d95aca52f4fbf08f7554aef4857/APT28-spreadphishing-scam/screenshots-APT28/blacklist_mocky.png

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/d06de78742548d95aca52f4fbf08f7554aef4857/APT28-spreadphishing-scam/screenshots-APT28/virustotal_mocky.png

---

## 3. Payload Analysis

### The Phishing Link

This campaign used a different delivery mechanism than Reports 1 and 2. Instead of
a malicious URL link, the email instructed recipients to open PowerShell and paste a
command. 

**Full execution of the attack:**

| Phase | Technique | Description |
|---|---|---|
| Initial Access | T1566.002 — Spearphishing Link | Email with instructions to execute PowerShell |
| Execution | T1059.001 — PowerShell | Victim runs attacker-supplied PowerShell command |
| Collection | T1082 — System Information Discovery | Script collects hostname, OS, users, network config |
| Exfiltration | T1567 — Exfiltration Over Web Service | Data sent to mocky.io via HTTP POST |
| Defense Evasion | T1036 — Masquerading | Email disguised as legitimate Windows admin message |
| Defense Evasion | T1027 — Obfuscated Files or Information | PowerShell command may be encoded or obfuscated |

### Why PowerShell

PowerShell is a legitimate Windows administration tool installed on every Windows
machine. Antivirus and endpoint security tools cannot simply block PowerShell because
it is required for system administration. By using PowerShell rather than delivering
a malware executable, APT28 avoids triggering file-based detection entirely. The
malicious behavior occurs entirely in memory, leaving minimal artifacts.

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/d06de78742548d95aca52f4fbf08f7554aef4857/APT28-spreadphishing-scam/screenshots-APT28/T1566_attack.png

---

## 4. What the Victim Experiences

A Ukrainian government employee receives an email from what appears to be their
department's system administrator, because the attacker used real employee names from prior reconnaissance. The subject says
"Windows Update" which is a routine IT maintenance. The body contains instructions in Ukrainian telling the employee to open PowerShell (with specific steps
for non-technical users) and paste a command that will "apply the update."

The employee follows the instructions. There is no malware download, no error message, no indication that anything went wrong. In the background,
the PowerShell command ran, collected the computer's hostname, operating system
version, user account list, and network configuration, and posted all of it to a
Mocky.io endpoint controlled by the attacker. The attacker now knows the employee's
machine is accessible, what operating system it runs, and what accounts exist on it.

This reconnaissance enables the next phase: the attacker targets this specific machine
with credentials or exploits appropriate for its configuration. In the CERT-UA#8399
advisory (December 2023), a related campaign showed APT28 moving from initial
phishing to threatening the domain controller within one hour. The information gathered
by campaigns like CERT-UA#6562 makes that speed possible.

---

## 5. Connection to APT Detection Lab

This report directly connects to my APT Detection Lab project. The relationship
between the two projects is a complete attack chain:

**Stage 1 — Initial Access (this report):**
APT28 sends a spearphishing email disguised as a Windows update.
A Ukrainian government employee runs the PowerShell payload.
The attacker gains information about the compromised machine.

**Stage 2 — Credential Access (APT Detection Lab):**
Using the foothold from Stage 1, the attacker deploys Mimikatz.
Mimikatz accesses LSASS memory and dumps NTLM hashes.
Domain credentials are extracted and available for lateral movement.
Wazuh detects the Mimikatz process creation (Event ID 4688, Rule 67027).

My two projects together document the attack chain that CERT-UA has documented
APT28 using against Ukrainian government infrastructure. 

---

## 6. MITRE ATT&CK Reference

| Field | Value |
|---|---|
| Technique | T1566.002 — Phishing: Spearphishing Link |
| Tactic | Initial Access |
| Threat actor | APT28 (G0007) |
| CERT-UA tracking | UAC-0028 |
| Reference | https://attack.mitre.org/techniques/T1566/002/ |
| APT28 group page | https://attack.mitre.org/groups/G0007/ |

---

## 7. Indicators of Compromise

| IOC Type | Value (Defanged) | Description |
|---|---|---|
| Sender pattern | [surname.initials]@outlook[.]com | Free Outlook accounts using real employee names |
| Subject line | Windows Update / Оновлення Windows | Fake system admin message in Ukrainian |
| Exfil endpoint | hXXps://run[.]mocky[.]io/v3/[unique-id] | Mocky.io API endpoint used for data collection |
| Payload type | PowerShell command | System enumeration and HTTP POST exfiltration |
| Threat actor | APT28 / UAC-0028 | Russian GRU military intelligence |
| CERT-UA advisory | CERT-UA#6562 | April 2023 campaign documentation |
| Target sector | Ukrainian government bodies | Multiple ministries and departments |

---

## 8. Recommendations

**Immediate technical controls:**
- Restrict PowerShell execution on government workstations, standard users should
  not be able to run arbitrary PowerShell commands
- Implement PowerShell Constrained Language Mode or application control policies
- Block outbound HTTP POST traffic to mocky.io, mockbin.org, and similar free API services
  from government endpoints
- Alert on PowerShell processes making HTTP connections to external IPs

**Email security:**
- CERT-UA recommends restricting users' ability to run PowerShell scripts, implement
  this as a Group Policy Object
- Train staff that IT administrators will never send system updates via email with
  PowerShell instructions
- Implement email impersonation detection that alerts when display names match known
  internal staff but the sending domain is external (@outlook.com)

**Monitoring:**
- Monitor for PowerShell child processes making network connections
- Alert on outbound connections from PowerShell to external services
- Deploy EDR capable of detecting process hollowing and memory-resident payloads

**Broader context:**
The CERT-UA advisory explicitly states that this phishing is designed to enable
follow-on operations against domain infrastructure. An organization that receives
this email has likely already been profiled. Treat receipt of this email as an
indicator of active targeting.
