---
Title: Anchored
Platform: HackTheBox
Type: Challenge
Difficulty: Easy
Category: Android
Status: Done
tags: 
CreatedOn: 04-12-2023
---

# Anchored


Hey everyone, in this write-up we will be solving an HTB challenge Anchored.

Link to the challenge: [https://app.hackthebox.com/challenges/284](https://app.hackthebox.com/challenges/284)

Let’s Start!!!!!!

First download and extract the given file.

![[Anchored/assets/Untitled.png|Untitled.png]]

From the README.txt file, we can that the application supports API level 29 or earlier.

![[Anchored/assets/Untitled 1.png|Untitled 1.png]]

Next, I extracted the apk using apktool.

![[Anchored/assets/Untitled 2.png|Untitled 2.png]]

After extracting the apk, I first take a look at the `AndroidManifest.xml` file and checked out for any minimum android version is required to run the apk.

![[Anchored/assets/Untitled 3.png|Untitled 3.png]]

But no minimum version is mentioned.

And it is also mentioned in the README file that, we have to install the apk on a non-rooted device.

![[Anchored/assets/Untitled 4.png|Untitled 4.png]]

So for this machine, I used a Android 10 virtual device, with Google Play support, installed via AVD Manager.

Let’s first install the apk on the emulator.

![[Anchored/assets/Untitled 5.png|Untitled 5.png]]

After installing, let’s take a look at the app. Its clearly mentioned in the description of the challenge that we have to intercept the traffic.

![[Anchored/assets/Untitled 6.png|Untitled 6.png]]

So setup Burp Proxy to intercept the request. Check here to know how to do it: [How to setup Burp Proxy to Intercept Android Traffic](https://www.notion.so/BurpSuite-Proxy-Setup-for-android-46c60fee75134fc98dc7e6c2b9599039?pvs=21).

When I first opened the app, the app showed a registration page for early access, so I used a random mail to register.

![[Anchored/assets/Untitled 7.png|Untitled 7.png]]

But I wasn’t able to capture the request using Burp Suite.

![[Anchored/assets/Untitled 8.png|Untitled 8.png]]

The above error was thrown by burpsuite, which states that certificate/SSL pinning is performed to prevent interception of requests.

Since this device is not rooted, we can’t use frida to bypass certificate pinning. We have to find some other way to intercept the SSL traffic.

On searching google to find a way to intercept the traffic, found the following: [https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/make-apk-accept-ca-certificate#manual](https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/make-apk-accept-ca-certificate#manual)

According to the above article, we can make the apk to accept our burp CA certificate in two methods, either manually or using a tool. We will see both methods here:

### Method - 1: Using the apk-mitm tool

- First install the tool from the following repo: [https://github.com/shroudedcode/apk-mitm](https://github.com/shroudedcode/apk-mitm)
- Next run the tool.
  ![[Anchored/assets/Untitled 9.png|Untitled 9.png]]
- Now install the patched apk file to the emulator.

> [!important]  
> NOTE: Don’t forget to uninstall the original apk before installing the patched apk.

![[Anchored/assets/Untitled 10.png|Untitled 10.png]]

- Now run the app and try to capture the request, if it didn’t work try to close the app completely [ don’t forget to clear from recent apps ] and then try once again.
- You can see the request in the HTTP history tab of the proxy menu, and if you check the raw request you can find the flag.

![[Anchored/assets/Untitled 11.png|Untitled 11.png]]

### Method - 2: Manually patching the apk

- If we take a look at the `AndroidManifest.xml` file, you can see that the `networkSecurityConfig` option is already configured.

![[Anchored/assets/Untitled 12.png|Untitled 12.png]]

- So we can directly modify the `network_security_config.xml` file located at `res/xml` path.

![[Anchored/assets/Untitled 13.png|Untitled 13.png]]

- Open the `network_security_config.xml` file in your favourite text editor, modify the `src` value of the certificates tag to `user` and save the file.

![[Anchored/assets/Untitled 14.png|Untitled 14.png]]

- Now it’s time to recompile the extracted apk files to again to apk. To do that go to the parent directory where the extracted apk files directory is located and run the command `apktool b <extracted_directory_path>`, to compile the files to apk.

![[Anchored/assets/Untitled 15.png|Untitled 15.png]]

- The new built apk is located at the `Anchored/dist/` path.

![[Anchored/assets/Untitled 16.png|Untitled 16.png]]

- Now its time to sign the apk. To do that, first we need a key. We can generate a key using the `keytool` tool. The command is `keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000`

![[Anchored/assets/Untitled 17.png|Untitled 17.png]]

- After generating the key, its time to sign the apk using the `jarsigner` tool. The command is `jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore Anchored.apk alias_name` , Enter the password that you used while creating the key.
- Now we have done all the necessary steps and have successfully patched the file. Now its time to install the application.

> [!important]  
> NOTE: Don’t forget to uninstall the original apk before installing the patched apk.

![[Anchored/assets/Untitled 18.png|Untitled 18.png]]

- Now run the app and try to capture the request, if it didn’t work try to close the app completely [ don’t forget to clear from recent apps ] and then try once again.
- You can see the request in the HTTP history tab of the proxy menu, and if you check the raw request you can find the flag.

![[Anchored/assets/Untitled 19.png|Untitled 19.png]]

We have successfully obtained the flag……

Thank You…..
