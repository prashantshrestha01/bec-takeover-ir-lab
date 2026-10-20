# Containment Runbook — Mailbox Account Takeover (BEC)

Use when a mailbox is confirmed (or strongly suspected) compromised. Work fast; preserve evidence as you go.

## 1. Immediate containment (minutes)

- [ ] **Revoke all sessions:** Entra admin center → user → *Revoke sessions* (kills refresh tokens; attacker is cut off even with a valid token).
- [ ] **Block sign-in** for the account (do this *before* the password reset so the attacker can't race you).
- [ ] **Reset the password** to a strong random value; require MFA re-registration at next sign-in.
- [ ] Record the time of each action for the timeline.

Exchange Online (PowerShell):
```powershell
# Find and remove malicious inbox rules
Get-InboxRule -Mailbox alex.morgan@contoso.com | Format-Table Name, ForwardTo, RedirectTo, DeleteMessage
Remove-InboxRule -Mailbox alex.morgan@contoso.com -Identity "<malicious rule name>" -Confirm:$false
# Check for forwarding on the mailbox itself
Get-Mailbox alex.morgan@contoso.com | Select ForwardingAddress, ForwardingSmtpAddress, DeliverToMailboxAndForward
```

## 2. Scope the blast radius (first hour)

- [ ] **Message trace:** did the attacker send mail? Identify every recipient; warn them the message was fraudulent.
- [ ] **Hunt laterally:** search sign-in logs for the attacker IPs/user agents across *all* users; check whether the spray hit anyone else successfully.
- [ ] **OAuth grants:** Entra → Enterprise applications → review app consents granted by the victim around the compromise window; revoke anything suspicious.
- [ ] **Other mailboxes:** sweep all mailboxes for forwarding/redirect rules created in the last 7 days.
- [ ] **Finance check:** confirm with Finance whether any wire was initiated — if yes, engage the bank's fraud team immediately.

## 3. Eradication

- [ ] Confirm all malicious rules, forwarding addresses, and OAuth grants are removed.
- [ ] Block attacker IPs/ASNs at the firewall and in Defender.
- [ ] Close the root-cause gap: **block legacy authentication via Conditional Access**; enforce MFA for all users; consider risk-based policies for executives.

## 4. Recovery

- [ ] Re-enable the account with fresh credentials + MFA; verify the legitimate user can sign in.
- [ ] Place the account on heightened monitoring for 14 days.
- [ ] Brief the victim: what happened, what to watch for (unexpected MFA prompts, missing mail).

## 5. Notify

- [ ] Finance leadership — confirm no funds moved.
- [ ] Legal/HR per company policy (BEC may trigger breach-notification obligations depending on data accessed).
- [ ] File the executive summary (`../report/executive-summary-template.md`).

## Evidence to preserve

Exported sign-in logs, Exchange message trace, the malicious rule definition, and timestamps of every containment action. In a real org, coordinate with legal before deleting anything.
