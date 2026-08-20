# Sisterhood of the Travelling Packets
**Ransomware Infrastructure Investigation | CTF Write-Up**

> **Authorization:** This write-up documents activities performed within an authorized Capture the Flag (CTF) environment. All systems, accounts, credentials, and infrastructure referenced were part of the challenge. 

## Overview

The Sisterhood of the Traveling Packets CTF centered on investigating the Tor-hosted infrastructure of a fictional ransomware group and identifying operational security failures that could be used to gain access to protected portions of their environment. 

I approached this challenge (as most others) as an investigation, rather than simply documenting the shortest path to the flag. This write-up includes the evidence I followed, hypotheses I tested, a few dead ends, and the reasoning behind each pivot. 

The investigation ultimately involved analyzing exposed victim data, discovering an accidentally leaked exfiltration script, enumerating web resources and API endpoints, accessing improperly protected internal communications, and recovering credentials for a known user account. 

# Investigation Environment 

* **Operating System:** Kali Linux
* **Network Access:** Tor
* **Techniques:** Source code inspection, artifact analysis, web enumeration, API enumeration, account enumeration, Base64 decoding, shell-script analysis, and cross-artifact correlation.

# Investigation Path

Tor Site → Page Source Inspection → Exposed Victim Downloads → Victim Data Analysis → Hidden .exfil.sh Artifact → Web Enumeration → robots.txt → /admin.php + /api.php → Account & API Enumeration → Internal Message Disclosure → Encoded Credential Discovery → Credential Decoding → Administrative Access → Flag

## 1. Initial Reconnaissance

I began by manually exploring the ransomware group's Tor-hosted website and reviewing the page source for information that was not visible through the normal interface. One of the first things that stood out was a suspicious Base64-encoded string. I decoded the value to determine whether it was the flag or contained hidden information, but it was a decoy flag that encouraged you to keep looking. 

Continuing through the source, however, revealed something much more interesting. Direct references to downloadable data belonging to two victim organizations.  

``` /downloads/quantumcore ```
``` /downloads/quantumcore/quantumcore_leak.zip ```  

``` /downloads/aetherflow ``` 
``` /downloads/aetherflow/aetherflow_leak.zip ```

The exposed paths provided access to leaked data belonging to QuantumCore Systems and AetherFlow Enterprises, giving me the next place to investigate. 

## 2. Victim Data Analysis

I downloaded and extracted the QuantumCore and AetherFlow archives and began reviewing their contents for anything that might provide more information about the victims, the ransomware group, or how the compromises occurred.   

While comparing the data from both organizations, one name stood out: ``` i.mccarthy ```. The same user appeared in data belonging to both Quantumcore and Aetherflow. Since the datasets came from two separate victim organizations, the overlap seemed quite suspicious. I investigated the accounts further to determine wehter it could represent a connection between the victims or the threat actors.  

It turned out to be a dead end.  

Still, it was a reasonable lead based on the evidence available at the time. With nothing else to connect ``` i.mccarthy ``` to the investigation, I returned to examining the downloaded files for anything I may have missed. 

## 3. Hidden Exfiltration Script

While examining the extracted AetherFlow files from the command line, I ran ``` ls -la ```. Using ``` -a ``` revealed a hidden file that wasn't initially visible in the directory listing: ``` .exfil.sh ```  

I opened the script and immediately found something interesting:  

> `# TODO: delete this before zipping`  

Oops. Someone apparently forgot that there.  

The script appeared to be part of the group's staging and exfiltration process and exposed several pieces of their internal infrastructure, including a Tor-hosted panel address, an API upload endpoint, an ``` X-Panel-Key ``` header and authentication key, and the files selected for exfiltration.   

The script also showed that the targeted files were Base64 encoded before being uploaded to the panel through ``` /api.php?action=upload ```. This provided the first direct look at how the group was staging and transmitting stolen data -- and an operational artifact they clearly hadn't intended to include in the victim archive. 

SCREENSHOT OF EXFIL.SH HERE

