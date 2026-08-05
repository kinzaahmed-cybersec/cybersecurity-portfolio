# Incident Investigation: Suspected Telegram Account Compromise

**Author:** Kinza Arshad  
**Role:** Cybersecurity Enthusiast | SOC Analyst Trainee

---

## Executive Summary

A trusted family member's Telegram account began sending explicit spam messages containing suspicious links to multiple contacts. Based on the unusual behavior, the incident was treated as a suspected account compromise. The objective was to identify the scope of the incident, contain the threat, and secure the affected account.

---

## Initial Indicators

- Multiple recipients received identical spam messages.
- Messages contained explicit clickbait text with embedded links.
- The account owner confirmed they had not intentionally sent the messages.
- Telegram displayed a security notification indicating **three new login sessions**.

---

## Investigation

The investigation followed a structured incident response methodology.

### 1. Identification

Evidence collected included:

- Screenshot of the malicious Telegram notification.
- Telegram security notification reporting unauthorized logins.
- Confirmation from multiple recipients that they received the same spam message.

These indicators strongly suggested an active account compromise rather than isolated user behavior.

---

### 2. Scope Assessment

The following checks were performed:

- Confirmed multiple contacts received the same message.
- Verified the sender's account information.
- Reviewed active Telegram sessions.

---

## Containment

The following actions were immediately performed:

- Terminated all active Telegram sessions except the legitimate device.
- Enabled Telegram Two-Step Verification.
- Warned affected contacts not to open the malicious links.
- Verified that no additional spam messages were sent after containment.

**Result:** The malicious activity ceased after the unauthorized sessions were terminated.

---

## Root Cause Analysis

The exact initial access vector could not be conclusively determined due to limited forensic evidence.

### Possible Attack Vectors

- Session token theft
- QR-code login phishing
- OTP phishing
- Malware on the endpoint
- Unauthorized Telegram Web/Desktop login

No definitive conclusion was reached without supporting forensic evidence.

---

## Lessons Learned

This investigation reinforced several important security practices:

- Enable multi-factor authentication before an incident occurs.
- Regularly review active account sessions.
- Treat unexpected login notifications as high-priority security events.
- Preserve evidence before making configuration changes.
- Notify affected users immediately to reduce further compromise.
- Avoid assuming an attack vector without sufficient evidence.

---

## Outcome

The incident was successfully contained by removing unauthorized sessions and enabling additional account protection. No further malicious activity was observed after containment.

This investigation highlighted the importance of rapid incident response, user awareness, and structured security operations.

---

## Skills Demonstrated

- Incident Identification
- Security Incident Triage
- Account Compromise Investigation
- Evidence Collection
- Threat Assessment
- Incident Containment
- User Awareness & Communication
- Root Cause Analysis
- Technical Documentation
- Incident Response Lifecycle (Identification → Containment → Lessons Learned)

---

## Evidence

> Sensitive information and malicious content have been redacted.

| Evidence | Description |
|----------|-------------|
| **Figure 1** | Malicious Telegram notification received by multiple contacts. |
| **Figure 2** | Telegram security notification indicating three unauthorized login sessions. |
| **Figure 3** | Devices page after containment showing only the legitimate device remained logged in. |
