# Lab 2 — System Enumeration & Security Testing

## Objective

The goal of this lab is to perform deep system enumeration and security testing against a vulnerable target to identify misconfigurations, outdated services, and backdoors. This involves mapping network shares, enumerating users, and safely exploiting known vulnerabilities in a controlled environment to gain system access.

## Lab Environment

* **Attacker Machine:** Kali Linux.


* **Local Target Machine:** Metasploitable 2 (IP: `<TARGET_IP>`).


* **Tools Utilized:** Nmap, Metasploit Framework (`msfconsole`), Searchsploit, Netcat, `enum4linux`, `smbclient`, `smtp-user-enum`, `showmount`.



## Methodology & Commands Used

### 1. Service Enumeration

Various tools were used to gather information about users, directories, and network shares before attempting exploitation.

* **SMTP (Port 25) User Enumeration:**
Used to verify active users on the system using a wordlist.


```bash
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t <TARGET_IP>

```


* **SMB/Samba (Ports 139, 445) Share Enumeration:**
Used `enum4linux` to perform a full check of shares and policies, discovering an accessible `tmp` share.


```bash
enum4linux -a <TARGET_IP>

```


Connected to the target anonymously to list and access the specific `tmp` directory.


```bash
smbclient -L <TARGET_IP>
smbclient -N //<TARGET_IP>/tmp

```


* **NFS (Ports 111, 2049) Enumeration:**
Scanned for NFS volumes using Nmap scripts (`nfs-ls.nse` and `nfs-showmount.nse`).


```bash
nmap <TARGET_IP> -p111,2049 --script=nfs-ls.nse

```


Verified exported directories using `showmount`.


```bash
showmount -e <TARGET_IP>

```



### 2. Exploitation & System Hacking

Several services were tested for known vulnerabilities, backdoors, or severe misconfigurations leading to direct access.

* **FTP (Port 21) — vsftpd 2.3.4 Backdoor:**
Discovered the outdated version and exploited it using two methods: Metasploit and a standalone Python script from Exploit-DB.


```bash
# Standalone script method
searchsploit vsftpd 2.3.4
searchsploit -m unix/remote/49757.py
python3 49757.py <TARGET_IP>

```


* **IRC (Port 6667) — UnrealIRCD 3.2.8.1 Backdoor:**
Exploited a known malicious backdoor in this specific IRC daemon version using Metasploit (`exploit/unix/irc/unreal_ircd_3281_backdoor`).


* **PostgreSQL (Port 5432):**
Utilized the Metasploit `linux/postgres/postgres_payload` module to achieve a reverse TCP meterpreter session.


* **Java RMI (Port 1099):**
Exploited an insecure default configuration using the Metasploit `multi/misc/java_rmi_server` module to gain root access.


* **Direct Shell via Ingreslock (Port 1524):**
Connected directly to the port using Netcat, instantly receiving an unauthenticated root shell.


```bash
nc <TARGET_IP> 1524

```


* **VNC Authentication Scanner (Port 5900):**
Used the Metasploit `scanner/vnc/vnc_login` auxiliary module to sweep for weak credentials, successfully finding a password, and connected using `vncviewer`.


```bash
vncviewer <TARGET_IP>

```



### 3. Exploiting Misconfigured Remote Access Protocols

The target utilized several insecure, legacy remote access protocols that allowed unauthorized root logins or unauthorized directory mounting.

* **NFS Root Mounting:**
Because the target exported the root directory (`/`), it was mounted locally on the attacker machine, providing full file system access.


```bash
sudo mkdir /mnt/system1
sudo mount -t nfs <TARGET_IP>:/ /mnt/system1

```


* **Legacy "R" Services (Ports 512, 513):**
Logged in directly as root without a password using deprecated UNIX services.


```bash
rlogin root@<TARGET_IP>
rsh root@<TARGET_IP>

```



## Actual Results & Findings

* **Information Disclosure:** `smtp-user-enum` successfully identified multiple valid system accounts (e.g., bin, daemon, ftp, root, user). SMB enumeration revealed an open `tmp` share, and NFS enumeration revealed that the entire root filesystem (`/`) was exported without restriction.


* **Backdoored Software:** Both `vsftpd 2.3.4` and `UnrealIRCD 3.2.8.1` contained malicious backdoors that immediately granted root-level command execution.


* **Unauthenticated Access:** Connecting directly to port 1524 (Ingreslock) provided an instant root shell without any authentication prompt.


* **Insecure Legacy Protocols:** The `rlogin` and `rsh` services allowed direct root access without requiring passwords. Furthermore, VNC was protected by a weak password that was easily discovered via sweeping.



## Security & Remediation Notes

* **Remove Backdoored/Outdated Software:** Services like `vsftpd 2.3.4` and `UnrealIRCD 3.2.8.1` must be immediately uninstalled and replaced with secure, updated versions (e.g., ProFTPd, modern vsftpd).
* **Disable Legacy Protocols:** Telnet, `rlogin`, `rsh`, and Ingreslock are highly insecure protocols that transmit data in plain text or allow trivial authentication bypass. They should be disabled immediately and replaced with SSH.
* **Secure File Shares:** NFS exports should never expose the root directory (`/`). Exports must be restricted to specific IP addresses (e.g., in `/etc/exports`) with strict read/write permissions. Anonymous SMB access should be disabled.
* **Enforce Strong Authentication:** VNC and database services (like PostgreSQL) must be configured with strong, complex passwords and should not be exposed to external or untrusted networks.
