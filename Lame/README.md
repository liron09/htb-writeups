# HTB: Lame

| | |
|---|---|
| **OS** | Linux |
| **Difficulty** | Easy |
| **Key vulnerability** | Samba 3.0.20 `username map script` command injection, **CVE-2007-2447** |
| **Result** | Root shell |

## 1. Recon
Full TCP port scan with service and version detection:
```bash
nmap -sC -sV -p- <TARGET_IP>
```
Key services: **FTP (vsftpd 2.3.4)**, **SSH**, **SMB (Samba 3.0.20)**.

## 2. Enumeration
- **vsftpd 2.3.4** is known for a backdoor, but on this machine the backdoor port is not reachable, so it is a dead end.
- **Samba 3.0.20** is vulnerable to **CVE-2007-2447**: shell metacharacters in the username are passed to `/bin/sh` when the `username map script` option is enabled.
```bash
smbclient -L //<TARGET_IP>/ -N
searchsploit samba 3.0.20
```

## 3. Exploitation
The username field is injected with a command, so the Samba service executes it. Because Samba runs as root, the resulting shell is **root directly**, with no separate privilege-escalation step.

Public exploit options: the Metasploit module `exploit/multi/samba/usermap_script`, or a manual payload (a reverse shell command in the username field) with a listener on the attacker machine.

## 4. Proof
Root access obtained; user and root flags captured.

## 5. Lessons Learned
- **Version enumeration is everything:** the exploit path came straight from the Samba version string.
- **Don't tunnel-vision on the first "obvious" exploit** (vsftpd). Verify it is reachable, then move on.
- **Service privileges matter:** a vulnerable service running as root turns a single bug into full system compromise.
- **Defensive takeaway:** patch legacy services, avoid running network daemons as root and restrict SMB exposure.
