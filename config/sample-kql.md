**Purpose**

This file contains some sample KQL queries that can be run after the logs are received.


**Show Failed Login Attempts**

SecurityEvent
| where EventID == 4625
|project TimeGenerated, Account, Computer, EventID, IpAddress

**Show Location and Ip of Attackers**

let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
|project TimeGenerated, Computer, IpAddress, countryname, cityname, latitude, longitude

**Aggregate the Number of Attacks Done by Each Attacker and Show Their Ip**

SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(7d)
| evaluate ipv4_lookup(_GetWatchlist("geoip"), IpAddress, network)
| summarize AttackCount = count() by countryname, cityname, IpAddress
| order by AttackCount desc


