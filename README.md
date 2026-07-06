# SOC Home Lab

This is a home lab I built to practice SOC analyst skills — setting up a small network, simulating some common attacks, and detecting them using Splunk. I did this to go beyond just watching tutorials and actually build something I could talk about in interviews.

## What I built

I set up two VMs in VMware — a Windows 10 machine as the "target" and a Kali Linux machine as the "attacker". I installed Sysmon on the Windows VM for better logging, and used Splunk as my SIEM to collect and search through the logs. Both VMs are on an isolated internal network so nothing leaves the lab.

```
Kali (attacker) <----> Windows 10 (target) --> Sysmon + Event Logs --> Splunk Forwarder --> Splunk (SIEM)
```

## Tools I used

- VMware Workstation
- Windows 10 (target VM)
- Kali Linux (attacker VM)
- Sysmon (SwiftOnSecurity config)
- Splunk Enterprise + Universal Forwarder
- Nmap, xfreerdp, PowerShell

## How I built it

1. Installed VMware and created the two VMs
2. Set up networking between them and made sure they could ping each other
3. Installed Sysmon on Windows to get better logs than the default Windows logging
4. Installed Splunk on my host machine and the Universal Forwarder on the Windows VM to send logs over
5. Spent a good amount of time debugging why logs weren't showing up in Splunk — turned out my `inputs.conf` file wasn't actually saving where it was supposed to because of a Windows Program Files permission issue. Had to fix it through command line directly.
6. Once logs were flowing, I ran a few different attacks from Kali and checked if I could catch them in Splunk
7. Built a dashboard and an alert around what I found
8. Wrote up an incident report on the main attack

## Attacks I simulated

**RDP Brute Force** — tried logging into the Windows VM with a few different usernames and wrong passwords from Kali. Caught this using Windows Event ID 4625 (failed logon) and Sysmon Event ID 3 (network connection).
```
index=main EventCode=4625 LogName=Security | stats count by Source_Network_Address
```
Got 15 failed attempts from the same IP, which looks like brute forcing.

**Nmap Scan** — ran a basic scan from Kali against the target to see open ports. Caught this in Sysmon's network connection logs.
```
index=main source="*Sysmon*" EventCode=3 "192.168.24.129"
```

**Encoded PowerShell** — ran a base64 encoded PowerShell command on the Windows VM (this is a common technique real attackers use to hide what a command is doing). Caught this in Sysmon's process creation logs.
```
index=main source="*Sysmon*" EventCode=1 CommandLine="*-enc*"
```

## MITRE ATT&CK mapping

Mapped each attack to its official MITRE ATT&CK technique, since that's how real SOC teams reference attacker behavior:

- RDP brute force → **T1110 - Brute Force**
- Nmap scan → **T1046 - Network Service Discovery**
- Encoded PowerShell → **T1059.001 - Command and Scripting Interpreter: PowerShell** and **T1027 - Obfuscated Files or Information**

All my SPL queries are in [spl_queries.md](./spl_queries.md).

## Dashboard

I made a Splunk dashboard with panels for total events, successful/failed logins, brute force attempts, network connections over time, and the other two attacks. Screenshots are in this repo.

## Alert

I also set up a Splunk alert that checks every 5 minutes for more than 3 failed logins from the same IP in the last hour. Tested it and confirmed it actually fires when it should.

## Incident report

I wrote up a proper incident report on the brute force attack, covering what happened, how I found it, and what I'd recommend to fix it if this were a real environment. You can read it here: [Incident_Report_Simple.md](./Incident_Report_Simple.md)

## What I learned

This project taught me a lot more than just following steps — especially the debugging part with the forwarder not sending logs. I had to actually understand how Splunk reads its config files instead of just trusting that things worked. I also got more comfortable thinking about detection from an analyst's point of view instead of just the attacker's side.

---
[LinkedIn](https://linkedin.com/in/jeela-raju-415b0132a) • [GitHub](https://github.com/Jeela-Raju)
