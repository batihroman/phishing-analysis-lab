# Phishing Analysis Report — 2
## Business Email Compromise — Kforce Inc Impersonation / Job Offer Scam

**Date Analyzed:** July 2026
**Attack Type:** Business Email Compromise — Credential and PII Harvesting
**Target:** Job seekers — individual targeting
**MITRE ATT&CK:** T1566.002 — Phishing: Spearphishing Link
**Source:** Real phishing email received in Gmail, documented by security researcher Secmode
**Reference:** https://github.com/Secmode/Phishing-Email-Analysis-Report-Kforce-Inc-Impersonation

---

## Executive Summary

This phishing email impersonates a hiring manager from Kforce Inc, a legitimate US-based
staffing company, and targets job seekers with a fake job selection message. The attacker
created a free mail.com account and sent the email from @consultant.com, a domain
unrelated to Kforce, to trick the victim into clicking a malicious link disguised as a
background check portal. The originating IP traces to Bangladesh with no reverse DNS, and
the link destination (afflat3d3.com) resolves to a server in Frankfurt, Germany — neither
consistent with a legitimate US staffing company. The email passes all authentication
checks because those checks validate the sending domain (@consultant.com), not the
impersonated company (Kforce). This is a textbook Business Email Compromise using a
legitimate third-party domain for credibility and evasion.

---

## Email Details

| Field | Value |
|---|---|
| From (displayed) | Claire Divas — Claire.Divas@consultant.com |
| Reply-To | Claire.Divas@consultant.com |
| Return-Path | Claire.Divas@consultant.com |
| Subject | Re: Congratulations! You Have Been Selected for Kforce Inc |
| Brand impersonated | Kforce Inc (legitimate US staffing company) |
| Delivery method | Malicious link via redirect chain |

---

## 1. Header Analysis

### Results

| Authentication Check | Result | Meaning |
|---|---|---|
| SPF | PASS | mail.com (74.208.4.201) is authorized for consultant.com |
| DKIM | PASS | Email signed by mail.com |
| DMARC | PASS (p=QUARANTINE) | Policy passes for consultant.com |

**Mail server IP:** 74.208.4.201 (mout.mail.com — external mail service)
**Originating client IP:** 103.76.241.19

**Received chain (two hops):**
1. 103.76.241.19 → web-mail.mail.com (submitted via webmail interface)
2. mout.mail.com (74.208.4.201) → mx.google.com (delivered to victim Gmail)

**Key notes:** The email was composed and submitted through mail.com's free webmail
interface from an IP in Bangladesh. A legitimate Kforce hiring manager in the United
States would send from Kforce's corporate mail servers, not from free webmail hosted
in Bangladesh. Authentication passing for consultant.com says nothing about Kforce.

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/f9b853f34fa0d056a10c86341e91639730433a10/job-offer-scam/screenshots-job-offer/mxtoolbox-report2.png

---

## 2. Domain Investigation

### Sending Domain: consultant.com

**What I found:** consultant.com is a real generic domain. The attacker is using a
legitimate-sounding domain (not a recently registered lookalike) to appear credible.
This makes it harder to block and explains why DMARC passes. Note the registration
date: this is an old domain, not a freshly registered phishing domain.

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/f9b853f34fa0d056a10c86341e91639730433a10/job-offer-scam/screenshots-job-offer/whois-report2.png

**ICANN lookup for consultant.com:**
https://github.com/batihroman/phishing-analysis-lab/blob/f9b853f34fa0d056a10c86341e91639730433a10/job-offer-scam/screenshots-job-offer/lookup-report2.png

### Originating IP: 103.76.241.19

**Blacklist Check:**

**What I found:** This IP has no reverse DNS. Legitimate mail
servers always have reverse DNS configured. An IP with no reverse DNS sending email is
suspicious by itself. It is listed in two blacklist databases: Spamhaus ZEN and RATS NoPtr which is definetelly a red flag.

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/f9b853f34fa0d056a10c86341e91639730433a10/job-offer-scam/screenshots-job-offer/blacklist-report2.png

**IPinfo lookup:**
- Country: Nowlamary, Khulna Division, Bangladesh
- AS Type: ISP, Business or hosting
- Anycast: False. This confirms this is a single dedicated server, not a distributed infrastructure. The IP maps to one physical location in Bangladesh

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/f9b853f34fa0d056a10c86341e91639730433a10/job-offer-scam/screenshots-job-offer/ipinfo-report2.png

---

## 3. URL Analysis

### Malicious URL: hXXps://afflat3d3[.]com/trk/lnk/...

**VirusTotal check:**

**Documented results:**
- Detected: 1/95 security vendors flagged as malicious
- Domain: afflat3d3.com
- Destination IP: 185.158.133.1 — Frankfurt am Main, Germany

**Screenshot:** https://github.com/batihroman/phishing-analysis-lab/blob/f9b853f34fa0d056a10c86341e91639730433a10/job-offer-scam/screenshots-job-offer/virustotal-check-report2.png

---

## 4. What the Victim Experiences

The victim receives what appears to be a reply to a job application they submitted
through a legitimate platform. The sender's name (Claire Divas) and the professional
tone of the message lower suspicion. The subject line ("Congratulations!") triggers
the recipient.

The email asks the victim to click a link to complete a background check. Background
checks are a legitimate part of hiring processes, so this request seems reasonable.
The victim clicks the link, passes through Google's SafeRedirect (which adds a false
sense of security), and lands on afflat3d3.com. There they are asked to enter personal
information: name, email, password, possibly payment details for the "background check
fee." All entered data goes directly to the attacker.

The email also explicitly requests the victim reply with their resume and screenshots
of their email login: a direct attempt to harvest additional credentials and PII.

---

## 5. MITRE ATT&CK Reference

| Field | Value |
|---|---|
| Primary technique | T1566.002 — Phishing: Spearphishing Link |
| Secondary | T1585.001 — Establish Accounts: Email Accounts |
| Tertiary | T1204.002 — User Execution: Malicious Link |
| Tactic | Initial Access |
| Reference | https://attack.mitre.org/techniques/T1566/002/ |

---

## 6. Indicators of Compromise

| IOC Type | Value (Defanged) | Description |
|---|---|---|
| Sender email | Claire.Divas@consultant[.]com | Impersonates Kforce hiring manager |
| Mail server IP | 74.208.4[.]201 | mout.mail.com: free mail service |
| Originating IP | 103.76.241[.]19 | Bangladesh, no reverse DNS |
| Destination IP | 185.158.133[.]1 | Frankfurt, Germany (not Kforce infrastructure) |
| Malicious domain | afflat3d3[.]com | Credential/PII harvesting site |
| Malicious URL | hXXps://afflat3d3[.]com/trk/lnk/894935A3-... | Tracked redirect link |
| Subject line | Re: Congratulations! You Have Been Selected for Kforce Inc | Social engineering lure |
| Message-ID | trinity-02aa31d3-e4d2-4e15-8bc4-9d54e4dab056-... | Campaign identifier |

---

## 7. Recommendations

**To block the email gateway:**
- Block mail from afflat3d3.com and 103.76.241.19
- Flag emails where display name claims a major company but sending domain does not match
- Alert on emails sent from free webmail services (mail.com, gmx.com) claiming to be
  from major US corporations

**To block the network level:**
- Block afflat3d3.com at DNS and web proxy
- Block 185.158.133.1 if no legitimate traffic expected

**User awareness:**
- Real recruiters from US staffing companies send from @kforce.com, not @consultant.com
- Verify hiring manager identity on LinkedIn before clicking any links
- No background check service requires your email login screenshots
- A job offer email composed from Bangladesh to a US job seeker is a geographic red flag

**If victim clicked the link:**
- Change passwords for all accounts where the same credentials were used
- Enable MFA on email, banking, and employment accounts
- Monitor for identity theft over the following months
