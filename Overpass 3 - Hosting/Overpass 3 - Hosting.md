---
Platform:
  - TryHackMe
Difficulty: Medium
Status: TODO
Blog Posting: Not Posted
Type: Machine
---
Hello folks, Today we are going solve the TryHackMe machine Overpass 3 - Hosting.

Link to the room: [https://tryhackme.com/room/overpass3hosting](https://tryhackme.com/room/overpass3hosting)

  

Lets Start !!!!!!

  

## Recon

First Start the machine and do a basic nmap scan.

Command: `nmap -A -T4 -v <TargetIP>`

![[Overpass 3 - Hosting/assets/Untitled.png|Untitled.png]]

From the response we can see 3 open ports: 21, 22, 80.

- On port 21 a FTP server is running.
- On port 22 SSH service is running.
- On port 80 a Apache web server is running.

Next we can take a look at the website that is running on port 80.

![[Overpass 3 - Hosting/assets/Untitled 1.png|Untitled 1.png]]

There was no additional links present in the website. Even I checked the source code nothing interesting found.

![[Overpass 3 - Hosting/assets/Untitled 2.png|Untitled 2.png]]

Even `robots.txt` not found.

![[Overpass 3 - Hosting/assets/Untitled 3.png|Untitled 3.png]]

Next I tried to Directory enumeration using `gobuster`.

Command: `gobuster dir -u http://<TargetIP>/ -w` `**/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt**`

![[Overpass 3 - Hosting/assets/Untitled 4.png|Untitled 4.png]]

From the results of `gobuster` , we can see that there is a folder named `/backups`.

Now I visited `http://<TargetIP>/backups`.

![[Overpass 3 - Hosting/assets/Untitled 5.png|Untitled 5.png]]

In the `/backups` directory, we can see that there is a `backup.zip` file. I downloaded and extracted the zip file.

![[Overpass 3 - Hosting/assets/Untitled 6.png|Untitled 6.png]]

From the extracted zip file, I got two files, `CustomerDetails.xlsx.gpg` and `priv.key`.

Now to decrypt the `CustomerDetails.xlsx.gpg` file, first we have to import the `priv.key` file using `gpg`.

Command: `gpg --import priv.key`

![[Overpass 3 - Hosting/assets/Untitled 7.png|Untitled 7.png]]

Next, decrypt the file and store the output using the following command:

Command: `gpg -d` `**CustomerDetails.xlsx.gpg**` `**>**` `CustomerDetails.xlsx`

![[Overpass 3 - Hosting/assets/Untitled 8.png|Untitled 8.png]]

Now if we open the `CustomerDetails.xlsx` file with a document viewer of your choice. In my I case I opened it with `libre office calc`.

![[Overpass 3 - Hosting/assets/Untitled 9.png|Untitled 9.png]]

We have found some interesting customer details, usernames, passwords and credit card details.

I noted these details and further continued my investigation.

Now I started to enumerate port 21, FTP server, to check for something interesting.

I tried to login with FTP by using the credentials that we got from `CustomerDetails.xlsx` file.

Command: `ftp <Target_IP>`

Username: `paradox`

Password: `ShibesAreGreat123`

![[Overpass 3 - Hosting/assets/Untitled 10.png|Untitled 10.png]]

I was able to login to FTP with the above credentials. Next to tried list the contents by using the `ls` command. From the output of the `ls` command, we can devise that this FTP server directory is the hosted apache server on port 80.

So, I tried to upload a reverse shell into the FTP server. In this case I used the reverse shell from the following link: [https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell).

> [!important]  
> Note: Don’t forget to update your the IP address in the reverse shell, to your TryHackMe tunnel IP address ( You can find it in the navbar of the tryhackme website )![[Overpass 3 - Hosting/assets/Untitled 11.png|Untitled 11.png]]  

To upload to a FTP server use the following command:

Command: `put <file_name>`

![[Overpass 3 - Hosting/assets/Untitled 12.png|Untitled 12.png]]

After successfully uploading, start a listener in your local machine using `netcat` . In my case, I have mentioned the port as 1234 in the reverse shell file. Check it for your file and create a listener accordingly.

![[Overpass 3 - Hosting/assets/Untitled 13.png|Untitled 13.png]]

Now open your browser and open the following link: [http://<TargetIP>/revshell.php](http://10.10.110.240/revshell.php). The website will be continuously loading.

![[Overpass 3 - Hosting/assets/Untitled 14.png|Untitled 14.png]]

Now check your netcat listener, you can see the reverse shell has executed successfully.

Now we have logged in as the user `apache`

Now spawn a tty shell to make your shell more stable by using the following command:

`python3 -c 'import pty; pty.spawn("/bin/sh")’`

![[Overpass 3 - Hosting/assets/Untitled 15.png|Untitled 15.png]]

Now check the home directory to find the web flag.

![[Overpass 3 - Hosting/assets/Untitled 16.png|Untitled 16.png]]

Now we have successfully found the First flag. Next we have to find the user flag.

  

[https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/apache-conf-privilege-escalation/](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/apache-conf-privilege-escalation/)

![[Overpass 3 - Hosting/assets/Untitled 17.png|Untitled 17.png]]