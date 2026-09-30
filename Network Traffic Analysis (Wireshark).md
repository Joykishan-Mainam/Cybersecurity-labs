#  — Network Traffic Analysis (Wireshark)

## Objective

The goal of this lab is to use Wireshark to capture and analyze network traffic. This includes observing the transmission of plaintext credentials over unencrypted protocols, identifying malicious network behavior such as ICMP floods, and understanding the core mechanics of the TCP 3-way handshake.

## Lab Environment

* **Attacker/Analysis Machine:** Kali Linux.


* **Target Machines:** Metasploitable 2 (IP: `192.168.245.139`) and external test site (`vbsca.ca`).


* **Tools Utilized:** Wireshark, hping3, standard FTP and Telnet clients.



## Methodology & Actual Findings

### 1. Capturing Plaintext Credentials

Unencrypted protocols transmit data, including usernames and passwords, in clear text. Wireshark was used to capture and reconstruct these login sessions.

* **HTTP Credential Capture:**
* **Action:** Visited a test login page (`[http://vbsca.ca/login/login.asp](http://vbsca.ca/login/login.asp)`) and submitted the credentials "demo" / "demo".


* **Wireshark Analysis:** Applied the `http` filter. Located the `POST /login/login_results.asp` packet, right-clicked, and selected **Follow > TCP Stream**.


* **Result:** The stream clearly displayed the submitted form data: `txtUsername=demo&txtPassword=demo`, exposing the credentials.




* **FTP Credential Capture:**
* **Action:** Connected to the Metasploitable 2 FTP server using the command `ftp 192.168.245.139` and logged in with the credentials "msfadmin" / "msfadmin".


* **Wireshark Analysis:** Applied the `ftp` filter. Located the "Login successful" response, right-clicked, and selected **Follow > TCP Stream**.


* **Result:** The stream revealed the exact sequence: `USER msfadmin` followed by `PASS msfadmin`.




* **Telnet Credential Capture:**
* **Action:** Connected to Metasploitable 2 using `telnet 192.168.245.139` and logged in with "msfadmin" / "msfadmin".


* **Wireshark Analysis:** Applied the `telnet` filter. Selected a Telnet packet, right-clicked, and followed the TCP stream.


* **Result:** The reconstructed stream displayed the server prompt and the attacker's keystrokes, clearly showing the login as `msfadmin` and the password as `msfadmin`.





### 2. Identifying Malicious Traffic (ICMP Flood)

To simulate a Denial of Service (DoS) attack, an ICMP flood was launched against the target using randomized source IPs.

* **Action:** Ran the following `hping3` command to flood the target:
```bash
sudo hping3 --icmp --flood --rand-source 192.168.245.139

```



* **Wireshark Analysis:** The packet capture showed a massive, continuous wave of ICMP Echo (ping) requests. Due to the `--rand-source` flag, the "Source" column in Wireshark displayed entirely random IP addresses (e.g., 251.25.170.186, 56.19.144.183), demonstrating how attackers spoof IP addresses to hide their origin during an attack.



### 3. Analyzing TCP 3-Way Handshakes

To observe normal connection establishment and termination, basic web traffic was generated.

* **Action:** Navigated to the Metasploitable 2 web server (`[http://192.168.245.139](http://192.168.245.139)`) in a browser.


* **Wireshark Analysis:** Applied the `tcp` filter.


* **Result:** The capture clearly displayed the standard TCP connection process prior to the HTTP GET request:
1. **SYN:** Client sends a synchronization packet to the server (port 80).


2. **SYN, ACK:** Server responds, acknowledging the request and sending its own synchronization.


3. **ACK:** Client acknowledges the server's response, establishing the connection.




* The capture also revealed connection termination packets (`FIN, ACK`) and occasional connection anomalies like `[TCP Retransmission]` and `[TCP Dup ACK]`.





## Security & Remediation Notes

* **Deprecate Cleartext Protocols:** HTTP, FTP, and Telnet provide zero encryption, allowing anyone on the network to passively sniff credentials. These must be replaced with HTTPS, SFTP, and SSH, respectively.
* **Network Monitoring:** The ICMP flood highlights the importance of configuring firewalls and Intrusion Detection Systems (IDS) to rate-limit ICMP requests and drop spoofed packets at the network edge.
