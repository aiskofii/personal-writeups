# [Hidden Deep Into My Heart](https://tryhackme.com/room/lafb2026e9)
**Platform:** TryHackMe \
**Category:** Web Exploitation \
**Difficulty:** Easy \
**Date:** 4 Oct 2026 \
**Author:** aiskofii

---

## Reconnaissance
At first, I didn't jump straight into the given web application, `http://[MACHINE_IP]:5000`, but I initiated a network port scan first before jumping straight
to see if there's a hidden surprise waiting for me.
```bash
nmap -sC -sV [MACHINE_IP]
```
**🚩 Flag Explained**
- `-sC` : Runs a built-in default script
- `-sV` : To determine the exact version running on the target

\
All I get was:
```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 9e:ac:29:e1:00:85:a3:1c:1f:35:8e:68:c0:92:91:2a (ECDSA)
|_  256 10:84:90:77:49:1b:59:fd:a5:e9:b1:f1:bc:33:e2:6f (ED25519)
5000/tcp open  http    Werkzeug httpd 3.1.5 (Python 3.10.12)
| http-robots.txt: 1 disallowed entry 
|_/cupids_secret_vault/*
|_http-server-header: Werkzeug/3.1.5 Python/3.10.12
|_http-title: Love Letters Anonymous
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
\
Wait a minute! There's one disallowed entry inside *robots.txt*. A hidden directory to */cupids_secret_vault/*. From here, I knew the existence of *robots.txt* and a hidden subdirectory.

And so I jumped straight into the web app and to */robots.txt* to see it for myself.

```txt
User-agent: *
Disallow: /cupids_secret_vault/*

# cupid_arrow_2026!!!
```

---

So, I already double-confirmed it. I went to the hidden subdirectory and immediately greeted by this:

```txt
You've found the secret vault, but there's more to discover...
```

What do you mean, *"there's more to discover..."*? I thought to myself, *"Does that mean there's more hidden directories?"* And so, I fired up *Gobuster* at the root directory.

```bash
gobuster dir -u http://10.48.133.208:5000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```
**🚩 Flag Explained**
- `-u` : specifies the target's URL
- `-w` : specifies the path to the wordlist

After waiting for minutes long, I gave up. I was getting discouraged, took a few walks here and there and by then, a thought popped up, *"Wait, I haven't enumerate directories at /cupids_secret_vault/"*

```bash
gobuster dir -u http://10.48.133.208:5000/cupids_secret_vault/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

Waiting and waiting for long... I thought I had made a mistake. Just as I was getting discouraged again, a line appeared:

```bash
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.48.133.208:5000/cupids_secret_vault/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
administrator        (Status: 200) [Size: 2381]
```

Boom! I went straight to the *administrator* directory. It was a login form. Standard username and password.
I already know since it was an administrator login form, the username was either *administrator* or *admin* but I didn't know the password. I was thinking I had to use *Hydra* to brute-force the password but no.
Suddenly, I remembered there was another line at the bottom of *robots.txt*

```txt
# cupid_arrow_2026!!!
```

Surely, it couldn't be it, right? I tried my luck with both assumed username and finally, I got the flag!

---

## 🔑 Key Takeaways

This room demonstrates a weakness by revealing obvious password hint inside *robots.txt*

The main lesson is to never **ever** reveal or hinting password anywhere.

## ⚙️ Tools Used
| Tools | Description |
| --- | --- |
| Nmap | Network Port Scanning Tool |
| Gobuster | Directory Enumeration |
