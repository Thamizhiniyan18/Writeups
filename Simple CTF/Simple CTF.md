---
Platform: TryHackMe
Difficulty: Easy
Status: Rooted/Finished
Type: Machine
Title: Simple CTF
Category: 
tags: 
CreatedOn: 04-12-2023
---
# Simple CTF

Hello folks, Today we are going solve the TryHackMe machine Simple CTF.

Link to the room: [https://tryhackme.com/room/easyctf](https://tryhackme.com/room/easyctf)

  

Lets Start…

First start the machine and run a standard Nmap scan on the target.

Command: `nmap -A -T4 -v <Target_IP>`

![Untitled.png](Simple%20CTF/assets/Untitled.png)

From the output we can see that there are a total of 3 services running on the target machine:

|   |   |   |
|---|---|---|
|**PORT**|**SERVICE**|**VERSION**|
|21|FTP|vsftpd 3.0.3|
|80|HTTP|Apache httpd 2.4.18|
|2222|SSH|Open SSH 7.2p2|

Now, First I visited the Apache server hosted on port 80.

![Untitled 1.png](Simple%20CTF/assets/Untitled%201.png)

It displayed the default apache welcome page. So next I tried directory enumeration on this web server using gobuster.

Command: `gobuster dir -u http://<TARGET_IP>/ -w` `**/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt**`

![Untitled 2.png](Simple%20CTF/assets/Untitled%202.png)

From the output of the gobuster, we can see that there is `/simple` directory.

Now visit the `http://<Target_IP>/simple` location.

![Untitled 3.png](Simple%20CTF/assets/Untitled%203.png)

This web server hosts a CMS using the CMS Made Simple, which is open source CMS. On scrolling down to the end of the page we can find the version of the CMS in the footer section.

![Untitled 4.png](Simple%20CTF/assets/Untitled%204.png)

The CMS version is `2.2.8` , I searched google, looking for exploits for this version of CMS and I got the following:

![Untitled 5.png](Simple%20CTF/assets/Untitled%205.png)

This CMS is vulnerable to the exploit with the CVE `CVE-2019-9053` , which is an unauthenticated SQL Injection on this CMS.

I downloaded the exploit.

After downloading the exploit I tried to run the exploit using the following command:

`python2 <exploit.py> -u http://<targetip>/simple`

But it thrown me a error that the exploit requires a package `termcolor` to run. So I downloaded the package using the following command: `python2 -m pip install termcolor`

Then I tried to run the exploit, the exploit ran successfully and the following output was obtained:

![Untitled 6.png](Simple%20CTF/assets/Untitled%206.png)

From this we have found the username is `mitch` and we have also got the password salt and hash. Now we can try to crack this password hash using `hashcat`.

Command: `hashcat -m 20 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2` `**/usr/share/wordlists/rockyou.txt**`

![Untitled 7.png](Simple%20CTF/assets/Untitled%207.png)

And we have cracked the password the password is `secret`

Now we can login via ssh using the obtained credentials:

username: `mitch`

password: `secret`

Command: `ssh mitch@<Target_IP> -p 2222`

![Untitled 8.png](Simple%20CTF/assets/Untitled%208.png)

![Untitled 9.png](Simple%20CTF/assets/Untitled%209.png)

And now we got the user flag.

Now we have escalate our privilege to access the root flag.

On Further enumeration, I have found another user in the home directory named `sunbath`.

I was looking out for privilege escalation vectors. I tried the `sudo -l` command, to check whether the user `mitch` can run any application with root privileges.

![Untitled 10.png](Simple%20CTF/assets/Untitled%2010.png)

From the output, we can devise that the user `mitch` can run `vim` with root privileges.

On checking `gtfobins` [ [https://gtfobins.github.io/gtfobins/vim/#sudo](https://gtfobins.github.io/gtfobins/vim/#sudo) ], we can use the following command to get a shell with root privilege:

![Untitled 11.png](Simple%20CTF/assets/Untitled%2011.png)

And we got root access.

![Untitled 12.png](Simple%20CTF/assets/Untitled%2012.png)

And finally we got the root flag:

![Untitled 13.png](Simple%20CTF/assets/Untitled%2013.png)

The answers for all the Task questions:

![Untitled 14.png](Simple%20CTF/assets/Untitled%2014.png)

![Untitled 15.png](Simple%20CTF/assets/Untitled%2015.png)

  

Thank You……