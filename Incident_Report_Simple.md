# Incident Report – RDP Brute Force Attempt

**Analyst:** Raju Jeela
**Date:** July 5, 2026
**Lab:** SOC Home Lab (VMware)

---

## What happened

While testing my home SOC lab, I simulated an RDP brute force attack from my Kali VM (192.168.24.129) against my Windows 10 VM (192.168.24.128). I tried logging in with a few different usernames (test, admin, administrator, root) and wrong passwords to see if my setup would catch it.

## How I detected it

I had Sysmon and Windows Event Logs forwarding to Splunk using the Universal Forwarder. After running the failed login attempts, I checked Splunk and found:

- **Sysmon Event ID 3** showed a network connection coming in from the Kali VM's IP.
- **Windows Security Event ID 4625** showed the failed login attempts, with the source IP (192.168.24.129) and workstation name (kali) clearly logged.

I used this search in Splunk to pull the failed logins:
```
index=main EventCode=4625 LogName=Security | stats count by Source_Network_Address
```
This showed 15 failed attempts from the same IP, which is a good sign of brute forcing.

## What I set up to catch this automatically

I created a Splunk alert called "Brute Force Login Attempt Detected" that runs every 5 minutes and checks if there are more than 3 failed logins from the same IP in the last hour. I tested it by running one more failed login attempt and confirmed the alert actually fired in Splunk's Triggered Alerts page.

## Why this matters

If this were a real attack, someone could eventually guess a working password or lock out an account. Since I run RDP on my target VM, this kind of brute forcing is a realistic thing to expect if RDP were ever exposed to the internet.

## What I'd recommend (if this were a real environment)

- Turn on account lockout after a few failed attempts
- Don't expose RDP directly to the internet, use a VPN instead
- Use MFA for remote login where possible
- Keep monitoring failed login attempts with alerts like the one I built

## Other things I caught during testing

While testing, I also ran an Nmap scan and a suspicious PowerShell command (base64 encoded), and both showed up in Splunk too — network connection logs for the scan, and process creation logs for the PowerShell command. So the same pipeline catches more than just brute force attempts.

## Conclusion

This was a good test of my lab — I could actually see an attack happen and trace it through Sysmon, Windows Event Logs, and Splunk, then get alerted automatically. It felt close to what a real SOC analyst would be doing when investigating failed login alerts.
