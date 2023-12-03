---
Platform:
  - HackTheBox
Category: Android
Difficulty: Medium
Status: TODO
Blog Posting: Not Posted
Type: Machine
---
Hey everyone, in this write-up we will be solving an HTB challenge APKey.

Link to the challenge: [https://app.hackthebox.com/challenges/240](https://app.hackthebox.com/challenges/240)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![[SeeTheSharpFlag/assets/Untitled.png|Untitled.png]]

Next I opened the apk with `jadx-gui` [ [https://github.com/skylot/jadx](https://github.com/skylot/jadx) ].

![[SeeTheSharpFlag/assets/Untitled 1.png|Untitled 1.png]]

I checked the `AndroidManifest.xml` file and found that we need at least API version 21 to run this app and the target API version is 30 i.e., Android 10. So, I installed the app on Android 10 emulator.

![[SeeTheSharpFlag/assets/Untitled 2.png|Untitled 2.png]]

Now let’s take a look at the app.