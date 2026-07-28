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

**Screenshot:**

---

## 2. Domain Investigation

### Sending Domain: consultant.com

**What I found:** consultant.com is a real generic domain. The attacker is using a
legitimate-sounding domain (not a recently registered lookalike) to appear credible.
This makes it harder to block and explains why DMARC passes. Note the registration
date: this is an old domain, not a freshly registered phishing domain.

**Screenshot:**

**ICANN lookup for consultant.com:**
screenshot

### Originating IP: 103.76.241.19

**Blacklist Check:**

**What I found:** This IP has no reverse DNS. Legitimate mail
servers always have reverse DNS configured. An IP with no reverse DNS sending email is
suspicious by itself. It is listed in two blacklist databases: Spamhaus ZEN and RATS NoPtr which is definetelly a red flag.

**Screenshot:**

**IPinfo lookup:**
- Country: Nowlamary, Khulna Division, Bangladesh
- AS Type: ISP, Business or hosting
- The Anycast says False which means that address is shared by multiple systems

**Screenshot:**

---

## 3. URL Analysis

### Malicious URL: hXXps://afflat3d3[.]com/trk/lnk/...

**How to run VirusTotal:**
Go to: `https://virustotal.com`
Click the URL tab
Paste: `https://afflat3d3.com/trk/lnk/894935A3-834A-4FD6-BB13-EF5DA75BC6EC/?o=28908&c=918277&a=716387&k=DFA1C0722440B87420F8A55EC1EEFC9C&l=32442&s1=priya`
Press Enter

[SCREENSHOT NEEDED: Save as screenshots/report-002/virustotal-afflat3d3.png]
What to capture: Detection count (3/95 vendors flagged), vendor names, threat category

**Documented results:**
- Detection ratio: 3/95 security vendors flagged as suspicious/malicious
- Domain: afflat3d3.com
- Destination IP: 185.158.133.1 — Frankfurt am Main, Germany
- Not consistent with US-based Kforce corporate infrastructure

**How to run URLscan.io:**
Go to: `https://urlscan.io`
Paste the full afflat3d3.com URL
Click Submit

[SCREENSHOT NEEDED: Save as screenshots/report-002/urlscan-afflat3d3.png]
What to capture: Page screenshot, final URL, any forms detected, IP geolocation

**Known URLscan result reference:**
Public scan available at: https://urlscan.io/result/01995550-c451-710f-a78c-fee54f5b37cd/

The URL uses a tracked redirect chain:
Email link → Google SafeRedirect → afflat3d3.com destination
The long URL with tracking parameters (o=, c=, a=, k=) confirms this is a
managed phishing campaign with click tracking per victim.

---

## 4. What the Victim Experiences

The victim receives what appears to be a reply to a job application they submitted
through a legitimate platform. The sender's name (Claire Divas) and the professional
tone of the message lower suspicion. The subject line ("Congratulations!") triggers
excitement before any verification occurs — this is the social engineering hook.

The email asks the victim to click a link to complete a background check. Background
checks are a legitimate part of hiring processes, so this request seems reasonable.
The victim clicks the link, passes through Google's SafeRedirect (which adds a false
sense of security), and lands on afflat3d3.com. There they are asked to enter personal
information — name, email, password, possibly payment details for the "background check
fee." All entered data goes directly to the attacker.

The email also explicitly requests the victim reply with their resume and screenshots
of their email login — a direct attempt to harvest additional credentials and PII.

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
| Mail server IP | 74.208.4[.]201 | mout.mail.com — free mail service |
| Originating IP | 103.76.241[.]19 | Bangladesh — no reverse DNS |
| Destination IP | 185.158.133[.]1 | Frankfurt, Germany — not Kforce infrastructure |
| Malicious domain | afflat3d3[.]com | Credential/PII harvesting site |
| Malicious URL | hXXps://afflat3d3[.]com/trk/lnk/894935A3-... | Tracked redirect link |
| Subject line | Re: Congratulations! You Have Been Selected for Kforce Inc | Social engineering lure |
| Message-ID | trinity-02aa31d3-e4d2-4e15-8bc4-9d54e4dab056-... | Campaign identifier |

---

## 7. Recommendations

**Block at email gateway:**
- Block mail from afflat3d3.com and 103.76.241.19
- Flag emails where display name claims a major company but sending domain does not match
- Alert on emails sent from free webmail services (mail.com, gmx.com) claiming to be
  from major US corporations

**Block at network level:**
- Block afflat3d3.com at DNS and web proxy
- Block 185.158.133.1 if no legitimate traffic expected

**Detection rule:**
- Alert when From display name contains a known company name but sending domain is
  a different unrelated domain — this is the Business Email Compromise signature

**User awareness:**
- Real recruiters from US staffing companies send from @kforce.com, not @consultant.com
- Verify hiring manager identity on LinkedIn before clicking any links
- No background check service requires your email login screenshots
- A job offer email composed from Bangladesh to a US job seeker is a geographic red flag

**If victim clicked the link:**
- Change passwords for all accounts where the same credentials were used
- Enable MFA on email, banking, and employment accounts
- Monitor for identity theft over the following months

---

## 8. What I Learned

Write this section in your own words. Think about:
- Why does passing SPF, DKIM, and DMARC not mean the email is safe?
- What made this more sophisticated than the casino spam in Report 001?
- What is the geographic inconsistency and why does it matter?
- What would you check first if this email arrived in your organization's SOC?
