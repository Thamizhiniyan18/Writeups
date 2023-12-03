---
Platform:
  - HackTheBox
Category: Web
Difficulty: Easy
tags:
  - jinja
  - template-injection
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Challenge
---
**Link to the Challenge:** [https://app.hackthebox.com/challenges/templated](https://app.hackthebox.com/challenges/templated)

  

First start the instance and navigate to the given IP address.

![[Templated/assets/Untitled.png|Untitled.png]]

We got the response back as site under construction, with a message `Proudly powered by Flask/Jinja2`. From this we can devise that the server is made up of Flask and it uses `Jinja2` template engine.

  

If we try to access a route which is not available, the server responds with a page not found error.

![[Templated/assets/Untitled 1.png|Untitled 1.png]]

If we take a look at the error, we can see that the error has reflected the route which we tried to access. This might be vulnerable to template injection.

So I looked out for Jinja2 payloads and found the following website: [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server Side Template Injection/README.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)

  

I tried the Detection payload from the above website to check whether it is vulnerable to template injection.

![[Templated/assets/Untitled 2.png|Untitled 2.png]]

  

I used Postman to send requests because it will be easy to modify and send the request each time.

![[Templated/assets/Untitled 3.png|Untitled 3.png]]

As mentioned in the Detection section, the server thrown a Error. So this site is vulnerable to template injection.

  

Next I tried some random payloads from the site and found the following which worked:

![[Templated/assets/Untitled 4.png|Untitled 4.png]]

  

When I tried one of the above mentioned payloads, It worked:

![[Templated/assets/Untitled 5.png|Untitled 5.png]]

  

Next again I tried some of the payloads in the above section and found the following payload to be working:

![[Templated/assets/Untitled 6.png|Untitled 6.png]]

![[Templated/assets/Untitled 7.png|Untitled 7.png]]

  

Okay, now we are able to execute commands and get the output for those commands. So I modified the request to list the contents of the directory:

**Modified Payload:** `{{ self.``**init**``.``**globals**``.``**builtins**``.``**import**``('os').popen('ls').read() }}`

![[Templated/assets/Untitled 8.png|Untitled 8.png]]

If we take a look at the response we can see the flag file `flag.txt`. This time I modified the payload to view the contents of the `flag.txt` file:

**Modified Payload:** `{{ self.``**init**``.``**globals**``.``**builtins**``.``**import**``('os').popen('cat flag.txt').read() }}`

![[Templated/assets/Untitled 9.png|Untitled 9.png]]

  

And we got the flag…….