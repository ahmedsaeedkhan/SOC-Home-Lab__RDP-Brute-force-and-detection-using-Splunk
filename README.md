# SOC Home Lab: Simulating RDP Brute Force Attack and Detection Using Splunk

A hands-on cybersecurity home lab where I simulated a real-world RDP brute force attack from a Kali Linux machine and detected every phase of the attack using Splunk SIEM.

---

## 🔬 Lab Architecture

![Architecture](<screenshots/Project_architecture.png>)

The lab runs entirely on a single physical machine. A Kali Linux VM acts as the attacker and a Windows 10 VM acts as the target, both running inside VirtualBox. The Splunk Universal Forwarder installed on the Windows 10 VM ships live Windows Security event logs to Splunk Enterprise on the host machine in real time.

---

## 🛠️ Tools Used

| Tool | Description |
|------|-------------|
| [Hydra](https://github.com/vanhauser-thc/thc-hydra) | Open-source password cracking tool used to perform the RDP brute force attack |
| [xfreerdp](https://www.freerdp.com) | Open-source RDP client used to establish remote access after cracking the password |
| [Splunk Enterprise](https://www.splunk.com) | SIEM platform used to collect, index, and investigate all attack activity |
| [Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html) | Lightweight agent that ships Windows event logs from the target VM to Splunk in real time |

---

## ⚔️ Attack Phases

### Phase 1 — RDP Brute Force
Used Hydra to brute force the RDP login of the Windows 10 VM using the rockyou.txt wordlist. Hydra successfully cracked the password after 27 attempts.

```bash
hydra -l win_user -P /usr/share/wordlists/rockyou.txt rdp://192.168.100.51 -t 1 -W 5
```

**Splunk Detection:** 27 x EventCode 4625 (Failed Login) detected, source identified as workstation `kali`

![Failed Logins](<screenshots/Failed_Login_Attempts.png>)

---

### Phase 2 — Successful RDP Login
After cracking the password, used xfreerdp to establish a remote desktop session to the target machine.

```bash
xfreerdp /u:win_user /p:12345 /v:192.168.100.51
```

**Splunk Detection:** EventCode 4624 (Successful Login) with Logon Type 10 (RDP)

![Successful Login](<screenshots/Successful_login.png>)

---

### Phase 3 — Privilege Escalation
Once inside, created a backdoor user account and added it to the local Administrators group for persistent elevated access.

```cmd
net user Attacker Password123 /add
net localgroup Administrators Attacker /add
```

**Splunk Detection:** EventCode 4720 (User Account Created) and EventCode 4732 (User Added to Group)

![Privilege Escalation](<screenshots/New_account_created_and_move_administrative_local_group.png>)

---

### Phase 4 — Covering Tracks
Attempted to destroy evidence by clearing the Windows Security event log.

```cmd
wevtutil cl Security
```

**Splunk Detection:** EventCode 1102 (Audit Log Cleared) — still captured in Splunk despite local deletion

![Logs Cleared](<screenshots/logs_cleared_investigation.png>)

---

## 📊 Splunk Dashboard

Built a real-time dashboard showing all attack phases in a single view.

![Dashboard](<screenshots/Dashboard_give_full_Incident_report.png>)

---

## 📋 Attack Timeline

| Time | Event Code | Description |
|------|------------|-------------|
| 2026-06-23 01:11:45 | 4625 | Brute force attack started |
| 2026-06-23 01:11:45 – 01:30:17 | 4625 x27 | 27 failed login attempts from kali |
| 2026-06-23 01:42:18 | 4624 | Successful RDP login (Logon Type 10) |
| 2026-06-23 01:58:12 | 4720 | Backdoor account 'Attacker' created |
| 2026-06-23 01:58:12 | 4732 | Account added to Administrators group |
| 2026-06-23 02:13:19 | 1102 | Windows Security logs cleared |

---

## 🔑 Key Finding

> The attacker cleared Windows Security logs to destroy evidence — but it didn't work.
> The Splunk Universal Forwarder had already shipped every event to Splunk before the logs were deleted.
> **This is exactly why a centralized SIEM exists.**

---

## 💡 What I Learned

- How to set up and configure Splunk Universal Forwarder for live log ingestion
- How RDP brute force attacks work and how they appear in Windows Security logs
- How to write SPL queries to detect brute force, unauthorized access, privilege escalation, and log clearing
- Why centralized log management is critical — local log deletion is ineffective when a SIEM is in place
- How to build a Splunk dashboard for real-time incident monitoring

---

## 📄 Full Report

The full project report with detailed analysis, screenshots, and findings is available here:
[Full Report (PDF)](report/SOC_Home_Lab_Report.pdf)
