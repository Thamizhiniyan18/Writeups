---
Platform:
  - TryHackMe
Difficulty: Easy
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Machine
---
In this writeup we are going to solve the TakeOver room from TryHackMe.

Link to the Room : [https://tryhackme.com/room/takeover](https://tryhackme.com/room/takeover)

  

Let’s Start 🙌

  

First start the machine. Next add the Machines’s IP address to `/etc/hosts` file in your attack box with reference to `futurevera.thm` and save it.

![[TakeOver/assets/Untitled.png|Untitled.png]]

Now first enumerate all the services that is running on the target. In my case I used `rustscan` to perform this action.

Command: `rustscan -a <Target_IP>`

![[TakeOver/assets/Untitled 1.png|Untitled 1.png]]

The Target machine has three services running:

|   |   |
|---|---|
|**Port**|**Service**|
|22|SSH|
|80|HTTP|
|443|HTTPS|

Now lets check the HTTP website running on port 80. It was a simple landing page. Nothing interesting found.

![[TakeOver/assets/Untitled 2.png|Untitled 2.png]]

Next I started enumeration. I tried directory enumeration, but didn’t get anything.

So next I tried subdomain enumeration on the target.

I used `ffuf` for this task.

Command: `ffuf -u` `[http://futurevera.thm/](http://futurevera.thm/)` `-w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm"`

![[TakeOver/assets/Untitled 3.png|Untitled 3.png]]

The result had a lots of `302` response. So this time I filtered out all the response with a status code `302` and ran the command again.

Command: `ffuf -u http://<target_ip>/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm" -fc 302`

![[TakeOver/assets/Untitled 4.png|Untitled 4.png]]

And this time found two subdomains. Now make sure to add these subdomains to the `/etc/hosts` file. You have to append the subdomains with the full url [ Eg: `portal.futurevera.thm` ] to the previously add host separated by a space:

![[TakeOver/assets/Untitled 5.png|Untitled 5.png]]

> [!important]  
> Remember to add all the subdomains to the /etc/hosts file to the respective IP address to access those subdomains.  

Now I checked the `portal.futurevera.thm` subdomain.

![[TakeOver/assets/Untitled 6.png|Untitled 6.png]]

It wasn’t accessible. So next I checked `payroll.futurevera.thm` subdomain.

![[TakeOver/assets/Untitled 7.png|Untitled 7.png]]

That was also not accessible.

  

Now we can check the HTTPS website:

![[TakeOver/assets/Untitled 8.png|Untitled 8.png]]

It resulted in the same landing page.

  

Next run `ffuf` on the HTTPS service to find the subdomains running via HTTPS.

Command: `ffuf -u https://<target_ip>/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm" -fc 302`

![[TakeOver/assets/Untitled 9.png|Untitled 9.png]]

Since all the request has a response with a status code of 200, our filter for code 302 didn’t work. So this time I tried to filter using size, since all the response sizes are `4605`.

Command: `ffuf -u https://<target_ip>/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -H "HOST:FUZZ.futurevera.thm" -fs 4605`

![[TakeOver/assets/Untitled 10.png|Untitled 10.png]]

And found two new subdomains. Now append these subdomains to the `/etc/hosts` file.

![[TakeOver/assets/Untitled 11.png|Untitled 11.png]]

Navigate to `https://blog.futurevera.thm/`.

![[TakeOver/assets/Untitled 12.png|Untitled 12.png]]

Nothing serious found in this website.

Next I checked the `https://support.futurevera.thm/`

![[TakeOver/assets/Untitled 13.png|Untitled 13.png]]

It has also nothing.

So next I started to look for the certificates in all of the above subdomains.

You can view the certificate of a website as shown in the video below:

![[TakeOver/assets/Viewing_certificate_in_Browser.webm]]

First I checked the certificate of `blog.futurevera.thm`. Didn’t found anything interesting.

Next I checked the certificate of `support.futurevera.thm`.

![[TakeOver/assets/Untitled 14.png|Untitled 14.png]]

We can see that the value of `DNS Name` property is referring to `secrethelpdesk934752.support.futurevera.thm`.

Now append this domain to the `/etc/hosts` file.

![[TakeOver/assets/Untitled 15.png|Untitled 15.png]]

If we navigate to `secrethelpdesk934752.support.futurevera.thm`, we can see a server not found error.

![[TakeOver/assets/Untitled 16.png|Untitled 16.png]]

But if you take a look at the error, you can see the server url, which contains the flag.

  

We have successfully found the flag.

  

Thankyou……..