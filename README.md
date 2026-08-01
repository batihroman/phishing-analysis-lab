# Phishing Analysis Lab

Three phishing email analyses covering different attack types: 
a casino spam, a job offer scam, and an APT28 spearphishing campaign 
documented by CERT-UA against Ukrainian government organizations.

---

## Why I Built This

This project is a starting point of my biggest project so far: APT Detection Lab (https://github.com/batihroman/apt-detection-lab).
Phishing Analysis Lab was created to show how a Russian hacker group APT 28 gained access to important Ukrainian infrastructure 
through email phishing. This project showcasing how different types of phishing attacks work, what happens when a victim
clicks a malicious link and how attackers gain access to networks. In summary, project shows what happens before the attacker 
gets inside the network.

APT28 targeted Ukrainian government ministries, energy companies, 
and military communications using these techniques. Building a lab to 
analyze them was the most direct way I could think of to actually understand 
how it worked.

---

## How This Connects to My Other Projects

These three projects together cover a full attack chain:

| Project | What it covers | Stage |
|---|---|---|
| [Active Directory Lab](https://github.com/batihroman/Home-Lab-Active-Directory-Project) | Built the Windows domain environment — 1,056 users, OUs, group policies | Infrastructure |
| [Network Scanning Lab](https://github.com/batihroman/Network-foundations-and-lab) | Mapped what was exposed on that network using Nmap — open ports, services, attack surface | Reconnaissance |
| [Phishing Analysis Lab](https://github.com/batihroman/phishing-analysis-lab) | Analyzed the emails used to gain initial access to machines like the ones I built | Initial Access |
| [APT Detection Lab](https://github.com/batihroman/apt-detection-lab) | Detected credential dumping after the attacker was already inside the domain | Credential Access |

---

## The Three Reports

**[Report 1 — Casino Spam / Credential Harvesting](casino-phishing-scam)**
Mass-distribution phishing campaign disguised as a casino promotion. Uses 
URL shorteners to hide the real destination and passes all email authentication 
checks because it authenticates correctly for its own domain. Shows how authentication passing does not mean an email is safe.

**[Report 2 — Job Offer Scam / Business Email Compromise](job-offer-scam)**
Impersonates a hiring manager from Kforce Inc, a real US staffing company. 
Sent from Bangladesh through a free mail.com account, routes to a credential 
harvesting site in Germany. More targeted and more convincing than Report 1,
harder to catch because it uses a legitimate existing domain instead of a 
freshly registered one.

**[Report 3 — APT28 Spearphishing Against Ukrainian Government (CERT-UA#6562)](APT28-spreadphishing-scam)**
April 2023 campaign by Russian military intelligence targeting Ukrainian 
government departments. Attacker created Outlook accounts using real employee 
surnames, sent fake Windows Update instructions in Ukrainian, and used 
PowerShell to exfiltrate system data to mocky.io. Everything runs through legitimate Microsoft and API services,
and standard email security controls couldn't detect this. This is the initial 
access technique that precedes credential dumping operations documented in 
the APT Detection Lab.

---

## Analysis Methodology

Same five-step process for every email:

1. Collect raw email artifacts: headers, body, links, attachments
2. Header analysis: MXToolbox for SPF, DKIM, DMARC, sending IP, relay path
3. Domain investigation: Whois registration date, ICANN lookup, blacklist check
4. URL analysis: VirusTotal detection ratio, URLscan.io safe detonation
5. MITRE ATT&CK mapping: technique identification, threat group attribution

---

## Tools Used

| Tool | Purpose |
|---|---|
| MXToolbox Email Header Analyzer | SPF/DKIM/DMARC verification, sending IP, relay path |
| VirusTotal | URL and domain scanning against 90+ vendors |
| URLscan.io | Safe URL detonation and page capture |
| Whois / ICANN | Domain registration investigation |
| MXToolbox Blacklist Check | Sending IP reputation |
| IPinfo | Geographic and ASN data for originating IPs |
| MITRE ATT&CK Navigator | Technique mapping and threat group attribution |

---

## MITRE ATT&CK Coverage

| Technique | ID | Reports |
|---|---|---|
| Phishing: Spearphishing Link | T1566.002 | 1, 2, 3 |
| Establish Accounts: Email Accounts | T1585.001 | 2, 3 |
| User Execution: Malicious Link | T1204.002 | 1, 2 |
| Command and Scripting: PowerShell | T1059.001 | 3 |
| System Information Discovery | T1082 | 3 |
| Exfiltration Over Web Service | T1567 | 3 |
| Masquerading | T1036 | 2, 3 |

---

## References

- CERT-UA#6562: https://cert.gov.ua/article/4492467
- APT28 MITRE page: https://attack.mitre.org/groups/G0007/
- T1566.002: https://attack.mitre.org/techniques/T1566/002/
