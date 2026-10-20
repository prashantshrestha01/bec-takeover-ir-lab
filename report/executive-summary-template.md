# Executive Summary — Email Account Compromise

> Template. Replace bracketed fields per incident. One page maximum.

| Field | Detail |
|-------|--------|
| **Incident** | Business email compromise — executive mailbox takeover |
| **Date detected** | [DATE, TIME, TIMEZONE] |
| **Account affected** | [NAME, TITLE] |
| **Detection method** | [e.g. impossible-travel alert / finance verification call] |
| **Dwell time** | [hours between initial access and containment] |

## What happened

On [DATE], an external attacker compromised [NAME]'s mailbox after [a password-spray / phishing / MFA-bypass via legacy auth]. The attacker [created a hidden inbox rule / sent a fraudulent wire request / accessed sensitive mail — delete as applicable].

## Business impact

- **Funds:** [no funds transferred / amount at risk / amount transferred]
- **Data:** [mailbox contents potentially exposed — describe scope]
- **Operations:** [account locked for X hours; no wider outage]

## What we did

1. Revoked the attacker's sessions, blocked and re-secured the account.
2. Removed malicious inbox rules and forwarding.
3. Confirmed no other accounts were compromised; swept for similar rules tenant-wide.
4. [Notified the bank / engaged legal — as applicable.]

## What changes next

- Block legacy authentication tenant-wide via Conditional Access.
- Enforce phishing-resistant MFA for executives and finance staff.
- Add the attack's indicators to monitoring; 14-day heightened watch on the affected account.

**Status:** [Contained / Monitoring] — **Owner:** [NAME]
