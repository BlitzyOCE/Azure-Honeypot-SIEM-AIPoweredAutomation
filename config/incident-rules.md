**Purpose**

This file contains some sample rule queries that can be added to the Sentinel's Analytics as detection rules.


**Successful Login After Failed Attempts**

SecurityEvent
| where EventID in (4625, 4624)
| order by TimeGenerated asc
| summarize FailedLogins = countif(EventID == 4625), 
            SuccessfulLogins = countif(EventID == 4624) by IpAddress, bin(TimeGenerated, 1h)
| where FailedLogins > 3 and SuccessfulLogins > 0

**New Attacker IP detected**

let KnownAttackers = SecurityEvent
    | where TimeGenerated < ago(7d)
    | where EventID == 4625
    | distinct IpAddress;
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(1h)
| where IpAddress !in (KnownAttackers)
| summarize FirstSeen = min(TimeGenerated), Attempts = count() by IpAddress

**Rapid Fire Login Attempts**

SecurityEvent
| where EventID == 4625
| summarize Attempts = count() by IpAddress, bin(TimeGenerated, 5m)
| where Attempts > 100

**Brute Force Login Attempt**

SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by IpAddress, bin(TimeGenerated, 1h)
| where FailedAttempts > 3