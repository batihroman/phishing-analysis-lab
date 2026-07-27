# Phishing Analysis Report — 1
## Casino Promotional Spam / Credential Harvesting via URL Shortener

**Date Analyzed:** July 2026
**Attack Type:** Spam-based Phishing — Credential/Malware Redirect
**Target:** General public (non-targeted mass campaign)
**MITRE ATT&CK:** T1566.002 — Phishing: Spearphishing Link
**Source:** Real phishing email documented by security researcher jacobdcook
**Reference:** https://github.com/jacobdcook/Phishing-Analysis-Lab

---

## Executive Summary

This email is a mass-distribution phishing campaign disguised as a promotional message from an online casino. The attacker sends bulk emails from a randomly-named subdomain
hosted on a cheap OVH VPS in France, offering fake casino bonuses to lure recipients into clicking a URL shortener link. The shortener hides the real destination, which
security vendors flag as a phishing page. The email passes SPF and DKIM authentication checks, but only for the attacker's own domain. The use of
the same shortened URL for the main button, the unsubscribe link, and the terms link confirms this is not a legitimate mailing service.

---

## Email Details

| Field | Value |
|---|---|
| From (displayed) | OneCasino |
| From (actual) | 132784@534617.vav.proo55.us.com |
| Return-Path | bounce@vav.proo55.us.com |
| Subject | Welcome gift inside: 50 spins waiting for you 💰.944175#. |
| Target | General public — mass distribution |
| Brand impersonated | Generic online casino |
| Delivery method | Malicious URL via URL shortener |

---

## 1. Header Analysis

### Results: https://github.com/batihroman/phishing-analysis-lab/blob/1af1ec4fb245369c857e45e459d37e67a4d17dcd/casino-phishing-scam/screenshots-casino/header_check_casino.png

| Authentication Check | Result | Meaning |
|---|---|---|
| SPF | PASS | 141.95.0.46 is authorized to send for vav.proo55.us.com — but this is the attacker's domain, not a real casino |
| DKIM | PASS | Email signed by 534617.vav.proo55.us.com and neutral/expired for googlemail.com |
| DMARC | PASS | Policy passes for attacker domain |
| Auto-Submitted | auto-replied | Bot-generated bulk mail |

**Originating IP:** 141.95.0.46
**Hosting provider:** OVH SAS — a major European VPS provider frequently abused for phishing campaigns due to low cost and weak abuse enforcement

---

## 2. Domain Investigation

### Sending Domain: vav.proo55.us.com

**What I found:**
- Domain appears to be attacker-controlled infrastructure
- No affiliation with any legitimate casino business
- Random subdomain naming pattern typical of spam infrastructure

**Blacklist Check:**

OVH IP ranges used for spam campaigns are frequently listed on multiple blacklists. The IP address was blacklisted in two blacklist databases.

---

## 3. URL Analysis

### Malicious URL: hXXps://tinyurl[.]com/mrymsuhv

**Documented results:**
- Detection ratio: 2/94 security vendors flagged as malicious
- Flagged by: Phishing Database, SafeToOpen
- Threat category: Phishing
- Status: HTTP 200 — page active at time of analysis

---

## 4. What the Victim Experiences

The victim receives an email that looks like a promotional offer from an online casino. The email has a HTML design using casino branding. It offers 50 free
spins with no deposit required. The victim clicks "Start Spinning Now" and is redirected through the tinyurl shortener to the actual phishing destination.

The victim has no visual indication they were redirected because URL shorteners do not show the final destination. Depending on the landing page, the victim either sees a fake
casino registration form that harvests their email, password, and payment details, or is served a malware download disguised as a casino app.

The "Unsubscribe" and "Terms" links going to the same tinyurl URL is the most telling technical indicator. Legitimate companies maintain separate endpoints for these. This
single URL for all three links is the fingerprint of bulk phishing infrastructure.

---

## 5. MITRE ATT&CK Reference

| Field | Value |
|---|---|
| Technique | T1566.002 — Phishing: Spearphishing Link |
| Tactic | Initial Access |
| Reference | https://attack.mitre.org/techniques/T1566/002/ |
| Groups using this | Many financially motivated threat actors |
| Detection | Monitor for URL shortener links in email; alert on bulk mail from OVH IP ranges |

---

## 6. Indicators of Compromise

| IOC Type | Value (Defanged) | Description |
|---|---|---|
| Sender address | 132784@534617.vav.proo55[.]us[.]com | Random subdomain, attacker infrastructure |
| Return-Path | bounce@vav.proo55[.]us[.]com | Bounce address on attacker server |
| Sending IP | 141.95.0[.]46 | OVH VPS France — spam infrastructure |
| Domain | vav.proo55[.]us[.]com | Attacker-controlled domain |
| Phishing URL | hXXps://tinyurl[.]com/mrymsuhv | URL shortener hiding real destination |
| Subject line | Welcome gift inside: 50 spins waiting for you 💰.944175#. | Campaign lure + tracking hash |
| Auto-Submitted | auto-replied | Confirms automated bulk sending |

---

## 7. Recommendations

**Block at email gateway:**
- Block all mail from vav.proo55.us.com and 534617.vav.proo55.us.com
- Consider blocking OVH IP range 141.95.0.0/24 if no legitimate traffic expected

**Block at network level:**
- Block tinyurl.com, bit.ly, and other URL shortener services at the web proxy
- Legitimate corporate communications should not require URL shorteners

**Detection rule:**
- Alert on emails where the Unsubscribe and CTA links share the same destination URL
- Alert on emails with Auto-Submitted: auto-replied combined with suspicious sending IP

**User awareness:**
- Legitimate casino promotions come from the casino's official domain, not random subdomains
- URL shorteners in promotional emails are a phishing red flag — never click them
- If you did click: change your email and gaming passwords immediately, check for
  suspicious charges
