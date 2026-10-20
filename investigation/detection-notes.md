# Detection Notes — What Gave the Attack Away

## Signals (strongest first)

1. **Impossible travel:** CEO signs in from Toronto (12:58 Oct 13) then Lagos (02:31 Oct 14) — 8,400+ km apart. Also same-day Toronto/Lagos overlap on Oct 14.
2. **Password spray precursor:** 12 failures across 10 users from one IP in 18 minutes, immediately before the success. Spray-then-success on the same IP is a high-confidence pattern.
3. **Legacy auth MFA bypass:** the successful takeover used IMAP; `MFA_Result` = "Bypassed — legacy auth". Any successful legacy-protocol sign-in for a mailbox user is worth an alert on its own.
4. **First-seen ASN/country:** ASN 29465 / Nigeria had never appeared for any user in the baseline.
5. **User-agent anomaly:** victim always uses Edge on Windows; attacker used a generic Linux UA for the spray and Chrome-on-Windows after — neither matches the baseline.
6. **Off-hours access:** 02:31 UTC = 22:31 Toronto time — outside the victim's normal 9:00–18:00 pattern.
7. **Malicious inbox rule:** `New-InboxRule` with forwarding to an external domain (Exchange audit log).

## Tuning note — the benign lookalike

`j.chen@contoso.com` signed in from Montreal on Oct 13 (business trip). A naive impossible-travel rule could flag Toronto→Montreal. Mitigations: domestic-distance threshold, travel allowlist, and correlating with HR travel records. I kept this row in the dataset deliberately.

## KQL — run these against the CSV's real-table equivalents

**Spray then success from the same IP:**
```kql
let failures = SigninLogs
    | where ResultType in (50126, 50053, 50055, 50056)
    | summarize FailedUsers = dcount(UserPrincipalName) by IPAddress, bin(TimeGenerated, 30m)
    | where FailedUsers >= 5;
let successes = SigninLogs
    | where ResultType == 0
    | project TimeGenerated, UserPrincipalName, IPAddress;
failures
| join kind=inner successes on IPAddress
| where TimeGenerated1 between ((TimeGenerated - 1h) .. (TimeGenerated + 2h))
```

**Successful legacy-protocol sign-in (MFA bypass vector):**
```kql
SigninLogs
| where ResultType == 0
| where AppDisplayName in ("IMAP", "POP", "SMTP", "MAPI")
| where UserPrincipalName !in (allowlisted_service_accounts)
```

**First-seen country/ASN per user (14-day baseline):**
```kql
let baseline = SigninLogs
    | where TimeGenerated between (ago(21d) .. ago(7d))
    | where ResultType == 0
    | summarize by UserPrincipalName, Country = tostring(LocationDetails.countryOrRegion);
SigninLogs
| where TimeGenerated > ago(7d) and ResultType == 0
| extend Country = tostring(LocationDetails.countryOrRegion)
| where (UserPrincipalName, Country) not in (baseline)
```

## CSV quick-analysis (no SIEM needed)

The same checks can be run on `sample-signin-logs.csv` with basic filtering, e.g.:
`awk -F, '$6=="Nigeria"' sample-signin-logs.csv` isolates the attack trail; the `soc-analyst-toolkit` log parser in my portfolio demonstrates programmatic burst detection on this pattern.
