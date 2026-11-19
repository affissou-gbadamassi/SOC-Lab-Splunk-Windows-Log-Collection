# Triage report: controlled failed logons on PC01

**Case:** LAB-002  
**Analyst:** Affissou Gbadamassi  
**Investigation date:** November 19, 2026  
**Priority:** Low  
**Disposition:** Closed as expected authorized test activity

## Summary

I investigated two failed logon attempts involving `SOC-Triage-Test` on `SOC-LAB-PC01`. I generated these attempts with `runas` as part of an authorized lab exercise, verified the Windows Security events locally, and located both events in Splunk. The search for successful logons mentioning this account returned zero matches.

## Evidence and findings

| Item | Observed result |
|---|---|
| Endpoint | SOC-LAB-PC01, 192.168.100.20 |
| Attempted account | SOC-LAB-PC01\SOC-Triage-Test |
| Data source | WinEventLog:Security in index main |
| Failed logons | Two events with EventCode 4625 |
| Splunk displayed event times | November 19, 2026, 10:17:14.416 and 10:18:13.167 |
| Logon type | 2, interactive |
| Failure reason | Unknown user name or bad password |
| Local event status / substatus | 0xC000006D / 0xC0000064 |
| Initiating account | SOCLAB\Administrator |
| Matching successful logons | Zero EventCode 4624 matches in the searched indexed data |

The local event screenshot shows the attempted account separately from the initiating Administrator account. Splunk's extracted `Account_Name` field contains both names, so I used the raw Windows event to distinguish their roles. Substatus `0xC0000064` is consistent with the nonexistent username used for this test.

## Assessment and action

The two failures match the two deliberate test attempts. I classified this case as expected authorized activity with low priority. I found no matching successful logon in the searched data. No account containment or password reset was required for this test; I retained the evidence and documented the outcome.

This conclusion applies to the controlled activity on PC01. It is not a finding that the environment is free of compromise.

## Investigation limits

Clock discrepancies affected the lab. I searched from `earliest=0` to `latest=+2d` to include events that could otherwise fall beyond the SIEM's current time. Times above are recorded as displayed in Splunk; the screenshots do not establish its display timezone. The local Windows event displays 12:18:13 PM for the second attempt, while Splunk displays 10:18:13.167 AM.

The zero-match result is limited to events available in Splunk and the selected search scope. Earlier Ubuntu CPU soft-lockup warnings and clock instability remain operational follow-up items. I have not demonstrated complete ingestion coverage or production-ready timekeeping.

## Evidence

[Local Windows event details](screenshots/06-windows-event-4625-test-account-details.png) · [Two events found in Splunk](screenshots/07-test-account-events-found-in-splunk.png) · [Failed-logon table](screenshots/08-splunk-failed-logon-triage-table.png) · [Successful-logon search: zero matches](screenshots/09-no-successful-logons-for-test-account.png)
