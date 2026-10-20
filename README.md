# BEC Account Takeover — Simulated Investigation Lab

![Incident Response](https://img.shields.io/badge/Incident_Response-C41E3A?style=flat-square) ![DFIR](https://img.shields.io/badge/DFIR-2D2D2D?style=flat-square) ![KQL](https://img.shields.io/badge/KQL-68217A?style=flat-square)


A case-study style investigation of a simulated CEO-fraud / business email compromise (BEC) account takeover, reconstructed entirely from synthetic sign-in logs. Built as a home-lab exercise in timeline analysis, signal correlation, and containment.

> **All data is fictional.** `contoso.com` is Microsoft's standard demo domain; every person, IP, and event here is invented for training.

## Scenario

An attacker runs a password spray against a small company's Microsoft 365 tenant, compromises the CEO's mailbox via legacy IMAP authentication (which bypasses MFA), creates a hidden inbox rule to cover their tracks, and sends a wire-fraud email to Finance impersonating the CEO.

## Attack chain (summary)

1. **Recon / initial access** — password spray from `197.210.29.5` (Lagos, Nigeria) against 10 accounts; CEO credential guessed.
2. **MFA bypass** — successful login via legacy **IMAP** protocol; MFA never challenged.
3. **Persistence** — malicious inbox rule created: forwards finance-related mail to an external address and marks it read.
4. **Collection** — mailbox and SharePoint enumeration over ~1 hour.
5. **Fraud** — wire-transfer request email sent from the CEO's real mailbox to Finance.
6. **Containment** — SOC revokes sessions, resets password, removes the rule; attacker locked out.

## What's in this lab

| Path | Contents |
|------|----------|
| `data/sample-signin-logs.csv` | 58-row synthetic Azure AD-style sign-in log with the full attack trail |
| `investigation/timeline.md` | Minute-by-minute attack timeline reconstructed from the logs |
| `investigation/detection-notes.md` | Which signals gave it away + the KQL queries that surface them |
| `runbooks/containment-runbook.md` | Step-by-step containment and eradication for a mailbox takeover |
| `report/executive-summary-template.md` | One-page executive summary template |

## Key findings (from the exercise)

- The spray was visible **20 minutes before** the successful takeover — a tuned spray rule would have fired first.
- Legacy authentication was the single point of failure; blocking it via Conditional Access would have stopped the takeover cold.
- The attacker's user-agent and ASN never matched the victim's baseline — basic anomaly correlation catches this without ML.

## Skills demonstrated

Log analysis, attack timeline reconstruction, BEC kill-chain understanding (initial access → persistence → collection → fraud), KQL detection authoring, containment runbook execution, and executive communication.

---

*Built by [Prashant Shrestha](https://github.com/prashantshrestha01) · Mississauga, ON 🇨🇦 · [LinkedIn](https://www.linkedin.com/in/cto-prashant/)*
