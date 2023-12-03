---
Platform: TryHackMe
Difficulty: Easy
Status: Rooted/Finished
Type: Machine
Title: b3dr0ck
Category: 
tags: 
CreatedOn: 04-12-2023
---
# b3dr0ck

In this writeup we are going to solve the b3dr0ck room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/b3dr0ck](https://tryhackme.com/room/b3dr0ck)

  

Lets Start……

  

First start the machine and give it around 5 minutes to boot up. Now we will take a look at the given hints…..

![[b3dr0ck/assets/Untitled.png|Untitled.png]]

From the given hints we can devise that there is a `nginx` server running on port `80`, which is redirecting all the traffic to a web server which is running on port `4040`. Also its mentioned that there is a TCP Socket listening which helps to retrieve the TLS credential files. We can perform a basic nmap service version enumeration scan on all ports to find the port on which this TCP Socket is listening to get the credential files.

Command: `nmap -sV -T4 -p- <TARGET_IP>`

![[b3dr0ck/assets/Untitled 1.png|Untitled 1.png]]

Note: It took me around 20 minutes to get the scan results, so be patient.

From the results we can see that on port `9009` we can see a service named `piChat` is running. We can further enumerate it by performing

Command: `nmap -A -T4 -p 9009 <TARGET_IP>`

![[b3dr0ck/assets/Untitled 2.png|Untitled 2.png]]

We can see that its a open tcp port which is replying for our request. Let’s try to establish connection with that port using netcat.

Command: `nc <TARGET_IP> 9009`

![[b3dr0ck/assets/Untitled 3.png|Untitled 3.png]]

We can see that we have established a connection. It looks like a chat service, which we can query to get the credential files.

![[b3dr0ck/assets/Untitled 4.png|Untitled 4.png]]

I tried some random stuff, but it didn’t work.

Then I tried the word “certificate”. It worked and it returned me a TLS certificate.

![[b3dr0ck/assets/Untitled 5.png|Untitled 5.png]]

The same way I queried for key and got the key also.

![[b3dr0ck/assets/Untitled 6.png|Untitled 6.png]]

Now we can save these key into separate files to establish a SSL connection with the target.

![[b3dr0ck/assets/Untitled 7.png|Untitled 7.png]]

You can see that I have stored the certificate and key in separate files. Now we can use OpenSSL to establish the connection. We can see SSL service is running on port `54321` in the nmap service scan results that we obtained earlier.

Command: `openssl s_client -connect <TARGET_IP>:54321 -cert` `**certificate**` `-key` `**key**`

Successfully established the connection:

![[b3dr0ck/assets/Untitled 8.png|Untitled 8.png]]

When I typed help and press enter, it returned me with a password which is most probably the password for the user `barney`. Now we can ssh into the target with these credentials.

Command: `ssh barney@<Target_IP>`

![[b3dr0ck/assets/Untitled 9.png|Untitled 9.png]]

We have successfully established a ssh connection and got the first flag barney.txt.

Now we have to find fred’s password. To do that so, we further enumerate the machine for loops holes. I checked the sudo processes for the user barney by using the following command: `sudo -l`

![[b3dr0ck/assets/Untitled 10.png|Untitled 10.png]]

From the results I got, we can see that the user barney can run the `/usr/bin/certutil` command/program as a sudo user. `certutil` is a command line tool which is used for creation and modification of certificates and keys. We can use this tool to create a certificate-key pair similar to the ones that we got for the user barney and we will enumerate the password for the user fred as we did for the user barney.

![[b3dr0ck/assets/Untitled 11.png|Untitled 11.png]]

Command to generate certificate and key: `sudo certutil fred “Fred Flintstone”`

![[b3dr0ck/assets/Untitled 12.png|Untitled 12.png]]

Now we have successfully generated the certificate and key which we can use to get the password of the user fred. Now save the certificate and key that we obtained in separate files as we did for the user barney.

![[b3dr0ck/assets/Untitled 13.png|Untitled 13.png]]

After saving the certificate and key in separate files, use the OpenSSL to establish the connection on port `54321` as we did before.

Command: `openssl s_client -connect <TARGET_IP>:54321 -cert` `**fred_certificate**` `````` ````` ```` ``` `` `-key fred_` `` ``` ```` ````` ```````**key**`

![[b3dr0ck/assets/Untitled 14.png|Untitled 14.png]]

Now we can see that I have established connection as the user fred.

Now type help to get the password for the user fred.

![[b3dr0ck/assets/Untitled 15.png|Untitled 15.png]]

Now we got fred’s password. So we can ssh into the target machine as fred to obtain the fred.txt flag.

Command: `ssh fred@<Target_IP>`

![[b3dr0ck/assets/Untitled 16.png|Untitled 16.png]]

So our next task is to find the root.txt flag. So we have to do escalate our privilege to read the `/root/root.txt` flag. So I started enumerating by checking the sudo processes of the user fred:

![[b3dr0ck/assets/Untitled 17.png|Untitled 17.png]]

We can see in the results that the user fred has the abilities to the run the following commands with root permissions:

`/usr/bin/base32 /root/pass.txt`

`/usr/bin/base64 /root/pass.txt`

Running the above commands I got the following result:

![[b3dr0ck/assets/Untitled 18.png|Untitled 18.png]]

We can see that we got two strings, one is `base32` encoded and the other one is `base64` encoded. So I tried to decode both using [CyberChef](https://gchq.github.io/CyberChef/).

First I tried to decode the `base32` string:

![[b3dr0ck/assets/Untitled 19.png|Untitled 19.png]]

Next I tried to decode the `base64` string:

![[b3dr0ck/assets/Untitled 20.png|Untitled 20.png]]

By decoding both the strings I got the same string as the result. So I taught that is the password and tried to switch user as root but i couldn’t. On further investigating I found that it is a `MD5` hash. So i used [CrackStation](https://crackstation.net/) to crack the hash.

![[b3dr0ck/assets/Untitled 21.png|Untitled 21.png]]

And finally we got the root password. So now we can switch user as root and obtain the root.txt file.

Command: `su root`

![[b3dr0ck/assets/Untitled 22.png|Untitled 22.png]]

  

We have successfully found all the Easter eggs of the machine b3dr0ck. See you guys back with another writeup.

  

Thankyou……