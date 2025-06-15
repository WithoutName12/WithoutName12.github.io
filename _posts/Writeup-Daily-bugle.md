---
title: "Daily Bugle's Writeup"
date: 14:05:2025
categories: [CyberSecurity]
tags: [Writeup]
---

# Daily Bugle
> Writeup by: WithoutName12 
### Difficulty: Hard
### Description:
Compromise a Joomla CMS account via SQLi, practise cracking hashes and escalate your privileges by taking advantage of yum.
## Flags:
- Access the web server, who robbed the bank?

Firstly use nmap to have general idea about infrastructure:`sudo nmap -sV -sC -T4 -O -Pn <IP> -oN <OUT_FILE>`

![](assets/2025-06-14-13-15-26.png)

**NOTE** that there is /administrator/ login page for **Joomla**

By visiting webpage  we can see name of the robber.

![](assets/2025-06-14-13-11-21.png)
- What is the Joomla version?

Webpage is running **Joomla CMS** 

By searching where default manifest file is we can see /administrator/manifests/files/joomla.xml and there is it's version.

- What is Jonah's cracked password?

Joomla version that is running on server in vulnerable to sql injection.

After searching for exploit I saw script by **XiphosResearch** called *joomblah*. It is python2 script and after running it, I got:
```
['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '<REDACTED>', '', '']
```

Hash starts with **2$y** so it is *bcrypt* hash, after checking mode option for **bcrypt** in hashcat's manual I cracked the password with rockyou wordlist:
```
hashcat -a 0 -m 3200 hash /usr/share/wordlists/rockyou.txt
```


Now I can login as Jonah in login page for joomla.

- What is the user flag?

Jonah is super user so we can change php template, so it runs reverse shell as we visit it. Change template for Protostar index.php to pentester monkey's php reverse shell, do not forget to change ip address and port.

Now do `nc -lvnp <PORT>` and reload main page.

![](assets/2025-06-14-14-14-25.png)
And I got the shell.
But we can not access home directory - /home/jjameson - as apache user.

Make shell interactive to be comfortable with:
```
python -c "import pty; pty.spawn('/bin/bash')"
<CTRL - Z> 
stty raw -echo; fg
```

Upon exploring filesystem I saw configuration file - configuration.php in /var/www/html 

![](assets/2025-06-14-14-22-07.png)

![](assets/2025-06-14-14-22-29.png)

Maybe it is password for jjameson, and it certainly is!

I can now see user.txt flag in home directory

### Privilege Escalation

- What is the root flag?

With `sudo -l`, we can see that jjameson user can run **yum** with sudo.

![](assets/2025-06-14-14-26-35.png)

*yum* is a tool for managing RPM (Red hat Package Manager) packages.

I searched for yum in infamous **GTFObins** website.

![](assets/2025-06-14-14-35-22.png)
Executed that and got a root user.

![](assets/2025-06-14-14-36-58.png)

And lab is complete. It was a good machine Credits to **TryHackMe**

I hope you enjoyed reading. It's my first writeup so I hope it is not that bad.

Thank you for reading! Bye:)
