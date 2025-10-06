```
  ____                   _      ____ _           _ _                       
 / ___| _ __   ___  _ __| |_   / ___| |__   __ _| | | ___ _ __   __ _  ___ 
 \___ \| '_ \ / _ \| '__| __| | |   | '_ \ / _` | | |/ _ \ '_ \ / _` |/ _ \
  ___) | | | | (_) | |  | |_  | |___| | | | (_| | | |  __/ | | | (_| |  __/
 |____/|_| |_|\___/|_|   \__|  \____|_| |_|\__,_|_|_|\___|_| |_|\__, |\___|
                                                               |___/      
    ⚡ Live Attack Defense: Real-Time Intrusion Prevention
```

---

# ⚔️ Snort Challenge - Live Attacks: Defending Against Real Threats

> *"Put your snort skills into practice and defend against a live attack"* — TryHackMe

This advanced challenge took my Snort skills to the next level by defending **J&Y Enterprise** against real-time cyber attacks. Unlike analyzing static packet captures, I had to detect and **actively block** live malicious traffic using Snort's Intrusion Prevention System (IPS) capabilities.

---

## 🎯 Mission Overview

**The Setting:** J&Y Enterprise - a world-renowned tech-coffee shop chain known for unique IT-themed recipes like "WannaWhite," "ZeroSleep," and their super-secret "Shot4J" recipe.

**The Threat:** Attackers are targeting their digital assets, and I was called in to implement real-time protection using Snort IPS.

**My AI Assistant:** J.A.V.A. (Just Another Virtual Assistant) helped identify anomalies and coordinate the defense.

---

## 🚨 Scenario 1: SSH Brute-Force Attack

### **The Alert**
> **J.A.V.A.:** *"We have a brute-force attack, sir. Somebody is knocking on the door!"*

### **Investigation Phase**

#### Step 1: Traffic Capture & Analysis
```bash
# Start Snort in sniffer mode to capture live traffic
sudo snort -v -l .
```

**Duration:** Ran for 10-15 seconds to capture attack traffic
**Result:** Generated `snort.log.1672414629` with suspicious activity

#### Step 2: Log Analysis
```bash
# Read captured packets in detail
sudo snort -r snort.log.1672414629 -X

# Search for specific port patterns
sudo snort -r snort.log.1672414629 -X | grep :22

# Look for SSH-specific indicators  
sudo snort -r snort.log.1672414629 -X | grep "ssh"
```

**Key Discovery:** Extensive traffic targeting **port 22** (SSH service)

#### Step 3: Packet Inspection
```bash
# Examine first 30 packets for detailed analysis
sudo snort -r snort.log.1672414629 -X -n 30
```

**Attack Pattern Identified:**
- **Service:** SSH (Secure Shell)
- **Protocol:** TCP
- **Port:** 22
- **Attack Type:** Brute-force login attempts

### **Defense Implementation**

#### Step 4: IPS Rule Creation
```bash
# Open Snort rules configuration
sudo gedit /etc/snort/rules/local.rules
```

**Critical Rule Written:**
```bash
drop tcp any 22 <> any any (msg:"SSH Connection attempted"; sid:100001; rev:1;)
```

**Rule Breakdown:**
- `drop` - **Block** traffic instead of just alerting
- `tcp any 22 <> any any` - Bidirectional TCP traffic on port 22
- `msg` - Descriptive alert message
- `sid:100001` - Unique rule identifier

#### Step 5: Live Attack Blocking
```bash
# Deploy IPS in full protection mode
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A full
```

**Result:** ✅ **Attack Successfully Blocked!**
- **Flag Retrieved:** `THM{81b7fef657f8aaa6e4e200d616738254}`
- **Attack Duration:** Blocked within 1 minute of rule deployment

---

## 🔄 Scenario 2: Reverse Shell Detection

### **The Discovery**
> **J.A.V.A.:** *"Persistent outbound traffic detected. Possibly a reverse shell..."*

### **Investigation Phase**

#### Step 1: Outbound Traffic Analysis  
```bash
# Capture suspicious outbound communications
sudo snort -v -l .
```

**Focus:** Investigating **insider threats** and **persistent outbound connections**

#### Step 2: Traffic Pattern Analysis
```bash
# Analyze captured traffic
sudo snort -r snort.log.1672697486 -X

# Search for suspicious port usage
sudo snort -r snort.log.1672697486 -X | grep ":4444"

# Examine packet details
sudo snort -r snort.log.1672697486 -X -n 10
```

**Attack Pattern Identified:**
- **Port:** 4444 (Common reverse shell port)
- **Protocol:** TCP  
- **Behavior:** Persistent outbound connections
- **Tool Association:** Metasploit framework

### **Defense Implementation**

#### Step 3: Reverse Shell Rule Creation
```bash
# Create blocking rule for reverse shell traffic
drop tcp any 4444 <> any any (msg:"Reverse Shell Detected"; sid:100001; rev:1;)
```

#### Step 4: Active Prevention
```bash
# Deploy IPS to block reverse shell communications
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A full
```

**Result:** ✅ **Reverse Shell Successfully Blocked!**
- **Flag Retrieved:** `THM{0ead8c494861079b1b74ec2380d2cd24}`
- **Communication Severed:** Malicious outbound connection terminated

---

## 🎓 Technical Skills Demonstrated

### **Live Traffic Analysis:**
- **Real-time packet capture** during active attacks
- **Pattern recognition** for identifying attack signatures  
- **Protocol analysis** for TCP/SSH and reverse shell communications
- **Port-based threat detection** (22, 4444)

### **Intrusion Prevention System (IPS):**
- **Rule-based blocking** instead of passive alerting
- **Bidirectional traffic control** using `<>` operators
- **Live deployment** of protection rules
- **Real-time threat mitigation**

### **Advanced Snort Usage:**
```bash
# Sniffer mode with logging
sudo snort -v -l .

# Detailed packet analysis  
sudo snort -r logfile -X -n [count]

# Pattern searching with grep integration
sudo snort -r logfile -X | grep "pattern"

# IPS deployment with DAQ
sudo snort -c config -q -Q --daq afpacket -i interfaces -A full
```

---

## 🔍 Attack Analysis Summary

| Scenario | Attack Type | Target Service | Port | Protocol | Tool/Method |
|----------|-------------|----------------|------|----------|-------------|
| 1 | Brute-Force | SSH Authentication | 22 | TCP | Credential Attacks |
| 2 | Reverse Shell | Outbound Communication | 4444 | TCP | Metasploit |

---

## 🛡️ Defense Strategy Insights

### **Proactive Monitoring:**
- **Inbound threat detection** - SSH brute-force attempts
- **Outbound anomaly detection** - Reverse shell communications  
- **Protocol-specific rules** - Tailored to service characteristics
- **Real-time response** - Immediate threat blocking

### **Rule Design Principles:**
1. **Action-oriented** - Use `drop` instead of `alert` for active defense
2. **Bidirectional awareness** - Monitor both inbound and outbound traffic
3. **Flexible targeting** - Use `any` for IP addresses to catch IP changes
4. **Clear messaging** - Descriptive rule messages for incident response

---

## 🏆 Key Learning Outcomes

### **Real-World Application:**
- **Live attack simulation** provided realistic threat scenarios
- **Time pressure** simulated actual incident response conditions
- **Multi-vector attacks** required comprehensive defense strategies
- **Business context** emphasized protecting valuable digital assets

### **IPS vs IDS Understanding:**
- **IDS (Detection):** Passive monitoring and alerting
- **IPS (Prevention):** Active blocking and threat mitigation  
- **Rule adaptation:** Different syntax for blocking vs. alerting
- **Performance impact:** Real-time processing overhead

### **Incident Response Workflow:**
1. **Detection** → Identify anomalous traffic patterns
2. **Analysis** → Determine attack type and characteristics  
3. **Rule Creation** → Write targeted blocking rules
4. **Deployment** → Activate real-time protection
5. **Verification** → Confirm successful threat mitigation

---

## 🚀 Professional Applications

This experience demonstrates skills directly applicable to:

### **SOC Operations:**
- Real-time threat detection and response
- Custom IPS rule development
- Live attack mitigation
- Multi-scenario threat analysis

### **Incident Response:**
- Rapid threat assessment and classification
- Emergency rule deployment
- Attack pattern recognition  
- Real-time forensic analysis

### **Network Security:**
- Proactive defense implementation
- Traffic anomaly detection
- Protocol-specific monitoring
- Insider threat detection

---

## 🎯 Challenge Reflection  

The **Snort Challenge - Live Attacks** was an intense, hands-on experience that bridged the gap between theoretical knowledge and real-world cybersecurity operations. Defending J&Y Enterprise against live attacks taught me:

**Critical Skills:**
- **Speed and accuracy** under pressure
- **Pattern recognition** in live traffic streams  
- **Strategic rule writing** for maximum protection
- **Real-time decision making** during active incidents

**Business Impact:**
- Protecting valuable intellectual property (Shot4J recipe)
- Preventing unauthorized access to critical systems
- Maintaining business continuity during attacks
- Demonstrating ROI of security investments

---

## 🏅 Mission Accomplished

**Final Results:**
- ✅ SSH Brute-force attack **BLOCKED**
- ✅ Reverse shell communication **TERMINATED**  
- ✅ J&Y Enterprise assets **PROTECTED**
- ✅ Flags captured: `2/2`
- ✅ Zero successful compromises

**J.A.V.A.'s Final Message:** *"Congratulations sir. It is inspiring watching you work."*

---

✍️ *This blog documents my successful defense of J&Y Enterprise against live cyber attacks using advanced Snort IPS capabilities, showcasing real-time threat detection and mitigation skills essential for modern cybersecurity operations.*

---