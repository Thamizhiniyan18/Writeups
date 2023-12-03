---
Platform:
  - TryHackMe
Difficulty: Medium
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Machine
---
  

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

![[Capstone Challenge/assets/Untitled.png|Untitled.png]]

So I checked [https://gtfobins.github.io/gtfobins/base64/](https://gtfobins.github.io/gtfobins/base64/), looking out for SUID privesc and got the following:

![[Capstone Challenge/assets/Untitled 1.png|Untitled 1.png]]

I tried the first command `sudo insall -m =xs $(which base64)` , but it didn’t work.

![[Capstone Challenge/assets/Untitled 2.png|Untitled 2.png]]

Next I tried the following command:

`base64 /etc/shadow | base64 --decode`

![[Capstone Challenge/assets/Untitled 3.png|Untitled 3.png]]

And got the contents of the `/etc/shadow` file.

Now I tried to crack the password hash of the user `missy` using `johntheripper`. I copied the hash and put that In a `hash.txt` file.

![[Capstone Challenge/assets/Untitled 4.png|Untitled 4.png]]

Now I used `john` to crack the hash.

Command: `john --wordlist=/usr/share/wordlists/rockyou.txt ~/Desktop/hash.txt`

![[Capstone Challenge/assets/Untitled 5.png|Untitled 5.png]]

And we got the password for the user `missy`. Now using the credentials, login as the user missy using the following command `su missy`.

![[Capstone Challenge/assets/Untitled 6.png|Untitled 6.png]]

Now we have logged in as the user `missy` . Now I searched for the `flag1.txt` file using the find command: `find / -name flag1.txt 2>/dev/null`

![[Capstone Challenge/assets/Untitled 7.png|Untitled 7.png]]

and found the flag at `/home/missy/Documents/flag1.txt`.

![[Capstone Challenge/assets/Untitled 8.png|Untitled 8.png]]

Next we have to find the root flag. Again I started looking out for privilege escalation vectors.

I checked the commands that the user `missy` can run with root privileges using the following command: `sudo -l`

![[Capstone Challenge/assets/Untitled 9.png|Untitled 9.png]]

And found that the user `missy` can run the find command with root privileges. So, again looking for ways to escalate privilege at [https://gtfobins.github.io/gtfobins/find/](https://gtfobins.github.io/gtfobins/find/) , found the following:

![[Capstone Challenge/assets/Untitled 10.png|Untitled 10.png]]

I tried the above command and got a shell with root privilege.

![[Capstone Challenge/assets/Untitled 11.png|Untitled 11.png]]

This time I used the find command to search the flag2.txt file.

Command: `find / -name flag2.txt 2>/dev/null`

And found the file at `/home/rootflag/flag2.txt`

![[Capstone Challenge/assets/Untitled 12.png|Untitled 12.png]]

And finally we got the root flag.

  

Thank you ….. See you guys with another awesome writeup….