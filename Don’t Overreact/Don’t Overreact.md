---
Platform: HackTheBox
Category: Android
Difficulty: Very Easy
Status: Rooted/Finished
Type: Challenge
Title: Don’t Overreact
tags: 
CreatedOn: 04-12-2023
---
# Don’t Overreact

Hey everyone, in this write-up we will be solving an HTB challenge Don’t Overreact.

Link to the challenge: [https://app.hackthebox.com/challenges/255](https://app.hackthebox.com/challenges/255)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![Untitled.png](Don’t%20Overreact/assets/Untitled.png)

Next I opened the apk with `jadx-gui` [ [https://github.com/skylot/jadx](https://github.com/skylot/jadx) ].

![Untitled 1.png](Don’t%20Overreact/assets/Untitled%201.png)

I checked the `AndroidManifest.xml` file and found that we need at least API version 21 to run this app and the target API version is 29 i.e., Android 10. So, I installed the app on Android 10 emulator.

![Untitled 2.png](Don’t%20Overreact/assets/Untitled%202.png)

Now let’s open the app and have look at it.

![Untitled 3.png](Don’t%20Overreact/assets/Untitled%203.png)

It’s just a simple page with HackTheBox Logo.

Let’s take a look at the source code.

On taking a look at the `MainApplication` class file, we can see that its a React Native application.

![Untitled 4.png](Don’t%20Overreact/assets/Untitled%204.png)

React Native application usually stores all the javascript as a single bundle file under the assets directory. So let’s take a look at it.

![Untitled 5.png](Don’t%20Overreact/assets/Untitled%205.png)

In the `index.android.bundle`, file if you scroll down to the end, you can find a base64 encoded version of the flag. Copy that and decode it using CyberChef.

![Untitled 6.png](Don’t%20Overreact/assets/Untitled%206.png)

We have successfully obtained the flag……..

  

Thank You………