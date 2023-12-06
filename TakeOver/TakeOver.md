---
Platform: TryHackMe
Difficulty: Easy
Status: Rooted/Finished
Type: Machine
Title: TakeOver
Category: 
tags: 
CreatedOn: 04-12-2023
---
# TakeOver

In this writeup we are going to solve the TakeOver room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/takeover](https://tryhackme.com/room/takeover)

  

Let’s Start 🙌

  

First start the machine. Next add the Machines’s IP address to `/etc/hosts` file in your attack box with reference to `futurevera.thm` and save it.

![Untitled.png](TakeOver/assets/Untitled.png)

Now first enumerate all the services that is running on the target. In my case I used `rustscan` to perform this action.

Command: `rustscan -a <Target_IP>`

![Untitled 1.png](TakeOver/assets/Untitled%201.png)

The Target machine has three services running:

|   |   |
|---|---|
|**Port**|**Service**|
|22|SSH|
|80|HTTP|
|443|HTTPS|

Now lets check the HTTP website running on port 80. It was a simple landing page. Nothing interesting found.

![Untitled 2.png](TakeOver/assets/Untitled%202.png)

Next I started enumeration. I tried directory enumeration, but didn’t get anything.

So next I tried subdomain enumeration on the target.

I used `ffuf` for this task.

Command: `ffuf -u` `[http://futurevera.thm/](http://futurevera.thm/)` `-w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm"`

![Untitled 3.png](TakeOver/assets/Untitled%203.png)

The result had a lots of `302` response. So this time I filtered out all the response with a status code `302` and ran the command again.

Command: `ffuf -u http://<target_ip>/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm" -fc 302`

![Untitled 4.png](TakeOver/assets/Untitled%204.png)

And this time found two subdomains. Now make sure to add these subdomains to the `/etc/hosts` file. You have to append the subdomains with the full url [ Eg: `portal.futurevera.thm` ] to the previously add host separated by a space:

![Untitled 5.png](TakeOver/assets/Untitled%205.png)

> [!important]  
> Remember to add all the subdomains to the /etc/hosts file to the respective IP address to access those subdomains.  

Now I checked the `portal.futurevera.thm` subdomain.

![Untitled 6.png](TakeOver/assets/Untitled%206.png)

It wasn’t accessible. So next I checked `payroll.futurevera.thm` subdomain.

![Untitled 7.png](TakeOver/assets/Untitled%207.png)

That was also not accessible.

  

Now we can check the HTTPS website:

![Untitled 8.png](TakeOver/assets/Untitled%208.png)

It resulted in the same landing page.

  

Next run `ffuf` on the HTTPS service to find the subdomains running via HTTPS.

Command: `ffuf -u https://<target_ip>/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm" -fc 302`

![Untitled 9.png](TakeOver/assets/Untitled%209.png)

Since all the request has a response with a status code of 200, our filter for code 302 didn’t work. So this time I tried to filter using size, since all the response sizes are `4605`.

Command: `ffuf -u https://<target_ip>/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm" -fs 4605`

![Untitled 10.png](TakeOver/assets/Untitled%2010.png)

And found two new subdomains. Now append these subdomains to the `/etc/hosts` file.

![Untitled 11.png](TakeOver/assets/Untitled%2011.png)

Navigate to `https://blog.futurevera.thm/`.

![Untitled 12.png](TakeOver/assets/Untitled%2012.png)

Nothing serious found in this website.

Next I checked the `https://support.futurevera.thm/`

![Untitled 13.png](TakeOver/assets/Untitled%2013.png)

It has also nothing.

So next I started to look for the certificates in all of the above subdomains.

You can view the certificate of a website as shown in the video below:

![Viewing_certificate_in_Browser](TakeOver/assets/Viewing_certificate_in_Browser.webm)

First I checked the certificate of `blog.futurevera.thm`. Didn’t found anything interesting.

Next I checked the certificate of `support.futurevera.thm`.

![Untitled 14.png](TakeOver/assets/Untitled%2014.png)

We can see that the value of `DNS Name` property is referring to `secrethelpdesk934752.support.futurevera.thm`.

Now append this domain to the `/etc/hosts` file.

![Untitled 15.png](TakeOver/assets/Untitled%2015.png)

If we navigate to `secrethelpdesk934752.support.futurevera.thm`, we can see a server not found error.

![Untitled 16.png](TakeOver/assets/Untitled%2016.png)

But if you take a look at the error, you can see the server url, which contains the flag.

  

We have successfully found the flag.

  

Thankyou……..