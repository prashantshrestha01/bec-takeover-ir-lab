# Attack Timeline — CEO Mailbox Takeover (simulated)

Reconstructed from `../data/sample-signin-logs.csv` plus Exchange audit events referenced in the investigation. All times UTC. (Local time in Toronto was UTC−4.)

## 2026-10-14 — Initial access

| Time (UTC) | Event |
|------------|-------|
| 02:11 – 02:29 | **Password spray:** 12 failed logins (`50126`) from `197.210.29.5` (Lagos, Nigeria, ASN 29465) against 10 different accounts, one attempt every ~1–2 minutes — classic low-and-slow spray. User agent: generic Linux. |
| 02:31 | **Takeover:** successful sign-in to `alex.morgan@contoso.com` (CEO) from the same IP via **legacy IMAP**. MFA shows as "Bypassed — legacy auth". |
| 02:33 – 02:41 | Attacker opens the mailbox via Exchange Online (3 sessions). User agent switches to Chrome-on-Windows — never seen on this account before. |
| 02:44 | **Persistence:** new inbox rule created on the CEO mailbox — forwards mail with finance keywords to an external address and marks it read. (Exchange audit event; not in the sign-in log.) |
| 02:47 – 03:18 | **Collection:** continued mailbox enumeration; one SharePoint Online session (looking for financial documents). |
| 03:26 – 03:44 | Attacker shifts egress to `197.210.55.91` (same ASN/city) — likely VPN endpoint rotation. Sessions continue. |

## 2026-10-14 — Fraud attempt and detection

| Time (UTC) | Event |
|------------|-------|
| 12:58 | CEO logs in normally from Toronto — unaware anything is wrong (the inbox rule is hiding the attacker's tracks). |
| 13:12 | **Fraud:** wire-transfer request email sent *from the CEO's real mailbox* to `priya.nair@contoso.com` (Finance), urging urgent payment to new vendor banking details. |
| 13:15 | Finance analyst opens the email; replies asking for verbal verification per policy — the rule had hidden her reply from the real CEO. |
| 14:02 | Attacker session from Lagos IP still active — checking for a reply. |
| 14:05 | **SOC containment begins:** sessions revoked, account blocked, password reset (see `../runbooks/containment-runbook.md`). |
| 14:31 | Attacker tries to log back in — rejected (`50126`). Locked out. |
| 14:35 | CEO re-onboarded: fresh password, MFA re-registered, sign-in from Toronto succeeds. |

## Dwell time

~12 hours from initial access (02:31) to containment (14:05). The spray was detectable at 02:11 — a 20-minute early-warning window that a tuned detection would have caught.
