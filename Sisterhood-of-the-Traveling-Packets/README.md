# Sisterhood of the Travelling Packets
**Ransomware Infrastructure Investigation | CTF Write-Up**

> **Authorization:** This write-up documents activities performed within an authorized Capture the Flag (CTF) environment. All systems, accounts, credentials, and infrastructure referenced were part of the challenge. 

## Overview

The Sisterhood of the Traveling Packets CTF centered on investigating the Tor-hosted infrastructure of a fictional ransomware group and identifying operational security failures that could be used to gain access to protected portions of their environment. 

I approached this challenge (as most others) as an investigation, rather than simply documenting the shortest path to the flag. This write-up includes the evidence I followed, hypotheses I tested, a few dead ends, and the reasoning behind each pivot. 

The investigation ultimately involved analyzing exposed victim data, discovering an accidentally leaked exfiltration script, enumerating web resources and API endpoints, accessing improperly protected internal communications, and recovering credentials for a known user account. 

# Investigation Environment 

* Operating System: Kali Linux
* Network Access: Tor
* Techniques: Source code inspection, artifact analysis, web enumeration, API enumeration, account enumeration, Base64 decoding, shell-script analysis, and cross-artifact correlation.

# Investigation Path

Tor Site → Page Source Inspection → Exposed Victim Downloads → Victim Data Analysis → Hidden .exfil.sh Artifact → Web Enumeration → robots.txt → /admin.php + /api.php → Account & API Enumeration → Internal Message Disclosure → Encoded Credential Discovery → Credential Decoding → Administrative Access → Flag

## 1. Initial Reconnaissance

I began by manually exploring the ransomware group's Tor hosted website and reviewing the page source for information that was not visible through the normal interface. One of the first things that stood out was a suspicious Base64-encoded string. I decoded the value to determine whether it was the flag or contained hidden information, but it was a false flag that encouraged you to keep looking. 

Continuing through the source, however, revealed something much more interesting. Direct references to downloadable data belonging to two victim organizations. 

/downloads/quantumcore/quantumcore_leak.zip
/downloads/aetherflow/aetherflow_leak.zip

The exposed paths provided access to leaked data belonging to QuantumCore System and AetherFlow Enterprises, giving me the next place to investigate. 

