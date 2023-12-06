---
Platform: TryHackMe
Difficulty: Medium
Status: Rooted/Finished
Type: Machine
Title: Capstone Challenge
Category: 
tags: 
CreatedOn: 04-12-2023
---
# Capstone Challenge

  

Hello everyone, this write-up is a walk-through of the Capstone Challenge, which is the final task of the TryHackMe room Linux Privilege Escalation.

Link to the Room: [https://tryhackme.com/room/linprivesc](https://tryhackme.com/room/linprivesc)

  

Lets Start……

  

First start the machine and login via SSH with the given credentials.

Username: `leonard`

Password: `Penny123`

Command: `ssh leonard@<Target_IP>`

  

Now I started to look out for the privilege escalation vectors.

I was looking out for files with SUID permissions with the following command:

`find / -type f -perm -04000 2>/dev/null` and found that the `/usr/bin/base64` file has SUID permission.

![Untitled.png](Capstone%20Challenge/assets/Untitled.png)

So I checked [https://gtfobins.github.io/gtfobins/base64/](https://gtfobins.github.io/gtfobins/base64/), looking out for SUID privesc and got the following:

![Untitled 1.png](Capstone%20Challenge/assets/Untitled%201.png)

I tried the first command `sudo insall -m =xs $(which base64)` , but it didn’t work.

![Untitled 2.png](Capstone%20Challenge/assets/Untitled%202.png)

Next I tried the following command:

`base64 /etc/shadow | base64 --decode`

![Untitled 3.png](Capstone%20Challenge/assets/Untitled%203.png)

And got the contents of the `/etc/shadow` file.

Now I tried to crack the password hash of the user `missy` using `johntheripper`. I copied the hash and put that In a `hash.txt` file.

![Untitled 4.png](Capstone%20Challenge/assets/Untitled%204.png)

Now I used `john` to crack the hash.

Command: `john --wordlist=/usr/share/wordlists/rockyou.txt ~/Desktop/hash.txt`

![Untitled 5.png](Capstone%20Challenge/assets/Untitled%205.png)

And we got the password for the user `missy`. Now using the credentials, login as the user missy using the following command `su missy`.

![Untitled 6.png](Capstone%20Challenge/assets/Untitled%206.png)

Now we have logged in as the user `missy` . Now I searched for the `flag1.txt` file using the find command: `find / -name flag1.txt 2>/dev/null`

![Untitled 7.png](Capstone%20Challenge/assets/Untitled%207.png)

and found the flag at `/home/missy/Documents/flag1.txt`.

![Untitled 8.png](Capstone%20Challenge/assets/Untitled%208.png)

Next we have to find the root flag. Again I started looking out for privilege escalation vectors.

I checked the commands that the user `missy` can run with root privileges using the following command: `sudo -l`

![Untitled 9.png](Capstone%20Challenge/assets/Untitled%209.png)

And found that the user `missy` can run the find command with root privileges. So, again looking for ways to escalate privilege at [https://gtfobins.github.io/gtfobins/find/](https://gtfobins.github.io/gtfobins/find/) , found the following:

![Untitled 10.png](Capstone%20Challenge/assets/Untitled%2010.png)

I tried the above command and got a shell with root privilege.

![Untitled 11.png](Capstone%20Challenge/assets/Untitled%2011.png)

This time I used the find command to search the flag2.txt file.

Command: `find / -name flag2.txt 2>/dev/null`

And found the file at `/home/rootflag/flag2.txt`

![Untitled 12.png](Capstone%20Challenge/assets/Untitled%2012.png)

And finally we got the root flag.

  

Thank you ….. See you guys with another awesome writeup….