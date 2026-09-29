# Cybersecurity-labs
Documentation of my cybersecurity labs, networking exercises, and security assessments.

Network Security & Reconnaissance
Objective
The goal of this lab is to perform network reconnaissance to discover active hosts on a local network and utilize various scanning techniques to identify open ports, service versions, operating systems, and basic vulnerabilities.   

Lab Environment
 Attacker Machine: Kali Linux.   
 Local Target Machine: Metasploitable 2 (IP: 192.168.245.130).   
 Tools Utilized: Netdiscover, Arp-Scan, Nmap.   
 DOCX

Methodology & Commands Used
1. Host Discovery (Layer 2)
 To identify all hosts connected to the specific network routing environment, two different ARP-based tools were used.
   
 Netdiscover: Used to send requests to the network router to list all connected hosts.   
 Command: sudo netdiscover.  

 Arp-Scan: Similar to Netdiscover, used to send network requests to list connected devices.   
 Command: sudo arp-scan -l.

2. Port Scanning & Enumeration (Nmap)
 Nmap was used to scan the discovered Metasploitable host to find open and closed ports, which helps in checking for port     weaknesses and vulnerabilities.   

 Normal Port Scan:
 Command: nmap 192.168.245.130.   
 Specific Port Scan:
 Command: nmap 192.168.245.130 -p21.   
 All Ports Scan:
 Command: nmap 192.168.245.130 -p-.   

 Service Version Detection: Used to identify specific software versions to check for vulnerabilities, such as outdated        systems.   
 Command: nmap 192.168.245.130 -p21 -sV.   

 OS Identification:
 Command: nmap 192.168.245.130 -O.   

 Default Script Scan:
 Command: nmap 192.168.245.130 -p21 -sC.   

 Advanced/Aggressive Scans:
 Fast All-Port Scan: nmap 192.168.245.130 -p- -vv -T4 (Using a higher -T number speeds up the scan). 

 Aggressive Scan: nmap 192.168.245.130 -A (Combines service detection, script scanning, and OS detection).   

3. Nmap Scripting Engine (NSE)
 Scripts located in /usr/share/nmap/scripts were utilized for targeted enumeration.   

 FTP Anonymous Login Check:
 Command: nmap 192.168.245.130 --script=ftp-anon.nse -p21.   

 Domain WHOIS Lookup:
 Command: nmap domain.com --script=whois-domain.nse -p80,443.   

 Actual Results & Findings
 Network Discovery: Both netdiscover and arp-scan successfully identified the VMware host at 192.168.245.130 alongside the    gateway and other VMware interface IPs.   

 Port Discovery: The default Nmap scan revealed multiple open TCP ports on the Metasploitable machine, including 21 (ftp),    22 (ssh), 23 (telnet), 25 (smtp), 53 (domain), and 80 (http).   

 Comprehensive Port Scan: Scanning all 65,535 ports (-p-) revealed 4 additional open ports (40414, 42359, 55686, 60573) that  were missed by the default top-1000 scan.   

 Service Vulnerability: The service scan (-sV) on port 21 identified vsftpd 2.3.4. The Nmap script scan (-sC and ftp-         anon.nse) confirmed that "Anonymous FTP login allowed (FTP code 230)" is enabled on this service.   

 OS Detection: Nmap accurately identified the target as running a Linux 2.6.X kernel (specifically between 2.6.9 and          2.6.33).   
 Domain Reconnaissance: The NSE whois-domain script successfully pulled registration data for Domain.com, revealing a         creation date of 2015-05-11 and an expiry date of 2027-05-11 registered via PKNIC.   

 Security & Remediation Notes
 Anonymous FTP: The presence of Anonymous FTP on vsftpd 2.3.4 poses a significant security risk, as it allows                 unauthenticated users to access the file system. Furthermore, vsftpd 2.3.4 is notoriously known for containing a malicious   backdoor.

Unnecessary Services: The sheer volume of open ports (Telnet, rlogin, shell) indicates a highly vulnerable system configuration. Telnet (port 23) transmits data in plain text and should be disabled in favor of SSH.

Service Hardening: Services should be restricted using firewalls (e.g., iptables), and outdated services like vsftpd 2.3.4 must be patched or replaced with secure, updated alternatives.
