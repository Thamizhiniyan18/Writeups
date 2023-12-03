---
Platform:
  - HackTheBox
Category: Android
Difficulty: Very Easy
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Challenge
---
Hey everyone, in this write-up we will be solving an HTB challenge Don’t Overreact.

Link to the challenge: [https://app.hackthebox.com/challenges/255](https://app.hackthebox.com/challenges/255)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![[Don’t Overreact/assets/Untitled.png|Untitled.png]]

Next I opened the apk with `jadx-gui` [ [https://github.com/skylot/jadx](https://github.com/skylot/jadx) ].

![[Don’t Overreact/assets/Untitled 1.png|Untitled 1.png]]

I checked the `AndroidManifest.xml` file and found that we need at least API version 21 to run this app and the target API version is 29 i.e., Android 10. So, I installed the app on Android 10 emulator.

![[Don’t Overreact/assets/Untitled 2.png|Untitled 2.png]]

Now let’s open the app and have look at it.

![[Don’t Overreact/assets/Untitled 3.png|Untitled 3.png]]

It’s just a simple page with HackTheBox Logo.

Let’s take a look at the source code.

On taking a look at the `MainApplication` class file, we can see that its a React Native application.

![[Don’t Overreact/assets/Untitled 4.png|Untitled 4.png]]

React Native application usually stores all the javascript as a single bundle file under the assets directory. So let’s take a look at it.

![[Don’t Overreact/assets/Untitled 5.png|Untitled 5.png]]

In the `index.android.bundle`, file if you scroll down to the end, you can find a base64 encoded version of the flag. Copy that and decode it using CyberChef.

![[Don’t Overreact/assets/Untitled 6.png|Untitled 6.png]]

We have successfully obtained the flag……..

  

Thank You………