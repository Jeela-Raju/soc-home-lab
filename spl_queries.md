# SPL Queries — SOC Home Lab

All searches run against Splunk index `main`, populated via Splunk Universal Forwarder from a Windows 10 VM (Sysmon + Windows Event Logs). Attacker machine: Kali Linux (192.168.24.129) on an isolated internal VMware network.

---

## Dashboard Panels

**Total Events (Security-relevant only)**
```spl
index=main sourcetype=WinEventLog* OR source=*Sysmon* earliest=-1h
```

**Successful Logins**
```spl
index=main EventCode=4624 | stats count
```

**Failed Logins**
```spl
index=main EventCode=4625 | stats count
```

**Brute Force Detection (failed logins by source IP)**
```spl
index=main EventCode=4625 LogName=Security | stats count by Source_Network_Address
```

**Network Connections Timeline**
```spl
index=main source="*Sysmon*" EventCode=3 | timechart count
```

**Reconnaissance Activity (Nmap scan detection via Sysmon)**
```spl
index=main source="*Sysmon*" EventCode=3 "192.168.24.129"
```

**Suspicious PowerShell Execution (Base64-encoded command)**
```spl
index=main source="*Sysmon*" EventCode=1 CommandLine="*-enc*"
```

**Combined Attack Timeline**
```spl
index=main (EventCode=4625 LogName=Security) OR (source="*Sysmon*" EventCode=3 "192.168.24.129") OR (source="*Sysmon*" EventCode=1 CommandLine="*-enc*") | timechart count by EventCode
```

---

## Alert

**Brute Force Login Attempt Detected**
- Search: `index=main EventCode=4625 LogName=Security | stats count by Source_Network_Address | where count >= 3`
- Schedule: Cron `*/5 * * * *` (every 5 minutes)
- Time Range: Rolling, last 1 hour
- Trigger: Number of results > 0
- Action: Add to Triggered Alerts

---

## Notes

- `EventCode=4625` searches must be filtered with `LogName=Security` — without this filter, unrelated Application log events sharing the same numeric code can produce false matches.
- Field name for source IP in Windows Security failed-logon events is `Source_Network_Address`, not `IpAddress`.
