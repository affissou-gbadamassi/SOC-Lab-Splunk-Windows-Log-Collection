# SOC triage report — Repeated failed-logon alert validation

**Case ID:** LAB-003  
**Date:** November 26, 2026  
**Host:** SOC-LAB-PC01  
**Domain:** soclab.test  
**Source:** Windows Security Event ID 4625, indexed in Splunk Enterprise (`index=main`)  
**Detection:** SOC Alert - Multiple Failed Logins Within 5 Minutes  
**Rule severity:** Medium  
**Case priority:** Low for confirmed authorized simulation  
**Disposition:** Controlled SOC-Test activity closed; separate Administrator failures not attributed

## Executive summary

An enabled scheduled Splunk alert detected groups of three failed logins within a five-minute window. The alert's trigger history displayed a fire time of **2026-11-26 16:50 UTC**. Inspection of the associated saved search results showed **three failures for `SOC-Test@soclab.test`** and **three for `Administrator`**, both on `SOC-LAB-PC01` in the same fixed five-minute bucket.

The three SOC-Test failures were generated intentionally on PC01 with `runas /user:SOC-Test@soclab.test cmd` and incorrect passwords. Each attempt produced Windows error 1326. Therefore, that portion of the detection is an authorized exercise, not an attack. The three Administrator failures were present in the same search results, but the screenshot does not establish their origin.

## Evidence

| Evidence | Observation |
|---|---|
| Test user | `SOC-Test` existed and was enabled in Active Directory |
| Effective fine-grained password policy | No output for `Get-ADUserResultantPasswordPolicy` |
| Default account-lockout threshold | 0 (no automatic lockout under default policy) |
| Test execution | Three incorrect-password `runas` attempts from PC01; each returned error 1326 |
| Search verification | Three `SOC-Test@soclab.test` failures in the 15-minute investigation window |
| Other failures | Three `Administrator` failures also appeared |
| Scheduled alert | Enabled, cron schedule `*/5 * * * *`, last-five-minute time range, results > 0 |
| Alert action | Add to Triggered Alerts; Medium severity |
| Fire time | 2026-11-26 16:50 UTC |
| Triggered results | `SOC-Test@soclab.test`: 3; `Administrator`: 3; bucket start 16:45 UTC |

The saved results show a **fixed five-minute bucket**, not a sliding window. Time displays in the environment were previously inconsistent, so this report cites the alert's UTC trigger time and does not claim all VM clocks were permanently synchronized.

## Analysis

- **Indicator:** Repeated Windows authentication failures, EventCode 4625.
- **Affected system:** SOC-LAB-PC01.
- **Known cause:** Three deliberately incorrect credentials for the non-administrator SOC-Test account.
- **Additional activity:** The Administrator account also met the threshold; cause unconfirmed from the displayed summary.
- **Outcome:** Splunk ingested the controlled events and recorded a triggered scheduled alert.
- **Security conclusion:** A working detection and alerting pipeline was demonstrated; no successful unauthorized login or compromise was shown by this test.

## Response and disposition

I verified the source account and test context, checked the Splunk results, reviewed the scheduled alert's trigger history, and retained the evidence for reporting. No containment or password reset was required for the authorized SOC-Test activity.

The Administrator failures should be investigated independently if treated as operational signals. The account names and counts alone are insufficient for attribution.

## Limitations and recommended follow-up

The controlled event proves live collection and alert execution, not full historical completeness. A Nov 25 failed-logon event remained present in the Windows log but could not be located in Splunk. Earlier VM time offsets and transport configuration issues also require follow-up before production use.

The detection uses `coalesce(TargetUserName,Account_Name)` and fixed five-minute buckets. Where `Account_Name` contains multiple extracted values, investigate raw event details before identifying the targeted principal. Tune the rule for service/computer accounts and false positives.

**Disposition:** Lab alert validated. SOC-Test activity classified as expected/authorized; unrelated Administrator matches not resolved by this case.
