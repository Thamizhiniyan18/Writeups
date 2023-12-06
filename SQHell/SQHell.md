---
Platform: TryHackMe
Category: Web
Difficulty: Medium
Status: InProgress
Type: Machine
Title: SQHell
tags: 
CreatedOn: 04-12-2023
---
# SQHell

In this writeup we are going to solve the SQHell room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/sqhell](https://tryhackme.com/room/sqhell)

  

This room is completely based on SQL Injections. You can refer this link: `[https://book.hacktricks.xyz/pentesting-web/sql-injection](https://book.hacktricks.xyz/pentesting-web/sql-injection)` to know more about what is SQL Injection and also to learn more SQL Techniques.

  

Let’s Start 🙌

  

First Start the machine and wait for 1 minute for the machine to boot up completely. After 5 minutes visit the given website.

![Untitled.png](SQHell/assets/Untitled.png)

  

The given website is blog site, with two posts posted by the admin. There are options to login and register. When I tried to register, the website responded back that “Registrations are no longer open”.

![Untitled 1.png](SQHell/assets/Untitled%201.png)

So I moved on to the login page. I tried to check the entry points ( username and password fields ) whether they are vulnerable to SQL Injection by brute-forcing the entry points using the logical operations given in the word-list below.

![sqli-logic](SQHell/assets/sqli-logic.txt)

I used the community edition of burp-suite to brute-force. I first captured the login request with random credentials and then send it to intruder to brute-force with sniper attack.

First I brute-forced the username field. But it didn’t provide me with any great results.

![Untitled 2.png](SQHell/assets/Untitled%202.png)

![Untitled 3.png](SQHell/assets/Untitled%203.png)

![Untitled 4.png](SQHell/assets/Untitled%204.png)

Then I tried to brute-force the password field to check whether it is vulnerable to sql injection with the same method and it worked for the logical operation: `1'oR'1'='1`. So I tried to verify that using the Burp repeater.

![Untitled 5.png](SQHell/assets/Untitled%205.png)

![Untitled 6.png](SQHell/assets/Untitled%206.png)

And we got our first flag.

Now we can move on to the second flag. For the second flag the hint was to check the terms and conditions.

![Untitled 7.png](SQHell/assets/Untitled%207.png)

![Untitled 8.png](SQHell/assets/Untitled%208.png)

In the Terms and Conditions, its mentioned that “We log your IP address for analytics purposes”.

  

The I tried to enumerate the directories in the website using gobuster.

Command: `gobuster dir -u <targetURL> -w <wordlist>`

  

  

  

  

![Untitled 9.png](SQHell/assets/Untitled%209.png)

I found the other pages `/user` and `/post`