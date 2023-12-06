---
Platform: HackTheBox
Category: Reversing
Difficulty: Easy
Status: Rooted/Finished
Type: Challenge
Title: Find The Easy Pass
tags: 
CreatedOn: 04-12-2023
---
# Find The Easy Pass

**Link to the Challenge:** [https://app.hackthebox.com/challenges/5](https://app.hackthebox.com/challenges/5)

  

First download the given file.

The give file is a zip file. Extract the zip file using the following command and the given password:

Command: `unzip <zip_file>`

password: `hackthebox`

  

We have got an exe file in the zip name `EasyPass.exe`.

I ran the executable and it asked for password:

![Untitled.png](Find%20The%20Easy%20Pass/assets/Untitled.png)

I tried some random password, but it throwed me an error stating `Wrong Password!`.

![Untitled 1.png](Find%20The%20Easy%20Pass/assets/Untitled%201.png)

So I used the `strings` tool to check out for useful strings/passwords.

![Untitled 2.png](Find%20The%20Easy%20Pass/assets/Untitled%202.png)

From the response of the `strings` tool, I didn’t find anything interesting.

Next I tried to use a debugger to simulate and check what is happening when I enter a password. In my case, I used the `Immunity Debugger`, use can use a debugger of your choice.

  

I opened the `EasyPass.exe` executable in `Immunity Debugger` and scrolling up the first tab, looking out for the string `Wrong Password!`. I successfully found the string. I added a breakpoint to that line by pressing the `F2` key.

![Untitled 3.png](Find%20The%20Easy%20Pass/assets/Untitled%203.png)

Next press `F9` key to run the simulate the executable to debug. Once you hit the `F9` key, you can see the program screen asking for password:

![Untitled 4.png](Find%20The%20Easy%20Pass/assets/Untitled%204.png)

Enter some random password and click check password in the program and check the immunity debugger’s 4th tab [ right side bottom ], you can the the password that you entered. On the next few lines, there is another string `fortran!`, to which the password that we enter might be compared to.

![Untitled 5.png](Find%20The%20Easy%20Pass/assets/Untitled%205.png)

Let’s try `fortran!` in the `Enter Password` field.

![Untitled 6.png](Find%20The%20Easy%20Pass/assets/Untitled%206.png)

We have successfully found the password.

  

Thank you!!!!!!