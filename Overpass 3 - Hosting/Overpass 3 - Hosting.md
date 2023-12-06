---
Platform: TryHackMe
Difficulty: Medium
Status: TODO
Type: Machine
Title: Overpass 3 - Hosting
Category: 
tags: 
CreatedOn: 04-12-2023
---
# Overpass 3 - Hosting

Hello folks, Today we are going solve the TryHackMe machine Overpass 3 - Hosting.

Link to the room: [https://tryhackme.com/room/overpass3hosting](https://tryhackme.com/room/overpass3hosting)

  

Lets Start !!!!!!

  

## Recon

First Start the machine and do a basic nmap scan.

Command: `nmap -A -T4 -v <TargetIP>`

![Untitled.png](Overpass%203%20-%20Hosting/assets/Untitled.png)

From the response we can see 3 open ports: 21, 22, 80.

- On port 21 a FTP server is running.
- On port 22 SSH service is running.
- On port 80 a Apache web server is running.

Next we can take a look at the website that is running on port 80.

![Untitled 1.png](Overpass%203%20-%20Hosting/assets/Untitled%201.png)

There was no additional links present in the website. Even I checked the source code nothing interesting found.

![Untitled 2.png](Overpass%203%20-%20Hosting/assets/Untitled%202.png)

Even `robots.txt` not found.

![Untitled 3.png](Overpass%203%20-%20Hosting/assets/Untitled%203.png)

Next I tried to Directory enumeration using `gobuster`.

Command: `gobuster dir -u http://<TargetIP>/ -w` `**/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt**`

![Untitled 4.png](Overpass%203%20-%20Hosting/assets/Untitled%204.png)

From the results of `gobuster` , we can see that there is a folder named `/backups`.

Now I visited `http://<TargetIP>/backups`.

![Untitled 5.png](Overpass%203%20-%20Hosting/assets/Untitled%205.png)

In the `/backups` directory, we can see that there is a `backup.zip` file. I downloaded and extracted the zip file.

![Untitled 6.png](Overpass%203%20-%20Hosting/assets/Untitled%206.png)

From the extracted zip file, I got two files, `CustomerDetails.xlsx.gpg` and `priv.key`.

Now to decrypt the `CustomerDetails.xlsx.gpg` file, first we have to import the `priv.key` file using `gpg`.

Command: `gpg --import priv.key`

![Untitled 7.png](Overpass%203%20-%20Hosting/assets/Untitled%207.png)

Next, decrypt the file and store the output using the following command:

Command: `gpg -d` `**CustomerDetails.xlsx.gpg**` `**>**` `CustomerDetails.xlsx`

![Untitled 8.png](Overpass%203%20-%20Hosting/assets/Untitled%208.png)

Now if we open the `CustomerDetails.xlsx` file with a document viewer of your choice. In my I case I opened it with `libre office calc`.

![Untitled 9.png](Overpass%203%20-%20Hosting/assets/Untitled%209.png)

We have found some interesting customer details, usernames, passwords and credit card details.

I noted these details and further continued my investigation.

Now I started to enumerate port 21, FTP server, to check for something interesting.

I tried to login with FTP by using the credentials that we got from `CustomerDetails.xlsx` file.

Command: `ftp <Target_IP>`

Username: `paradox`

Password: `ShibesAreGreat123`

![Untitled 10.png](Overpass%203%20-%20Hosting/assets/Untitled%2010.png)

I was able to login to FTP with the above credentials. Next to tried list the contents by using the `ls` command. From the output of the `ls` command, we can devise that this FTP server directory is the hosted apache server on port 80.

So, I tried to upload a reverse shell into the FTP server. In this case I used the reverse shell from the following link: [https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell).

> [!important]  
> Note: Don’t forget to update your the IP address in the reverse shell, to your TryHackMe tunnel IP address ( You can find it in the navbar of the tryhackme website )![Untitled 11.png](Overpass%203%20-%20Hosting/assets/Untitled%2011.png)  

To upload to a FTP server use the following command:

Command: `put <file_name>`

![Untitled 12.png](Overpass%203%20-%20Hosting/assets/Untitled%2012.png)

After successfully uploading, start a listener in your local machine using `netcat` . In my case, I have mentioned the port as 1234 in the reverse shell file. Check it for your file and create a listener accordingly.

![Untitled 13.png](Overpass%203%20-%20Hosting/assets/Untitled%2013.png)

Now open your browser and open the following link: [http://<TargetIP>/revshell.php](http://10.10.110.240/revshell.php). The website will be continuously loading.

![Untitled 14.png](Overpass%203%20-%20Hosting/assets/Untitled%2014.png)

Now check your netcat listener, you can see the reverse shell has executed successfully.

Now we have logged in as the user `apache`

Now spawn a tty shell to make your shell more stable by using the following command:

`python3 -c 'import pty; pty.spawn("/bin/sh")’`

![Untitled 15.png](Overpass%203%20-%20Hosting/assets/Untitled%2015.png)

Now check the home directory to find the web flag.

![Untitled 16.png](Overpass%203%20-%20Hosting/assets/Untitled%2016.png)

Now we have successfully found the First flag. Next we have to find the user flag.

  

[https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/apache-conf-privilege-escalation/](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/apache-conf-privilege-escalation/)

![Untitled 17.png](Overpass%203%20-%20Hosting/assets/Untitled%2017.png)