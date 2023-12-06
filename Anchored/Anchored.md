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

![Untitled.png](Anchored/assets/Untitled.png)

From the README.txt file, we can that the application supports API level 29 or earlier.

![Untitled 1.png](Anchored/assets/Untitled%201.png)

Next, I extracted the apk using apktool.

![Untitled 2.png](Anchored/assets/Untitled%202.png)

After extracting the apk, I first take a look at the `AndroidManifest.xml` file and checked out for any minimum android version is required to run the apk.

![Untitled 3.png](Anchored/assets/Untitled%203.png)

But no minimum version is mentioned.

And it is also mentioned in the README file that, we have to install the apk on a non-rooted device.

![Untitled 4.png](Anchored/assets/Untitled%204.png)

So for this machine, I used a Android 10 virtual device, with Google Play support, installed via AVD Manager.

Let’s first install the apk on the emulator.

![Untitled 5.png](Anchored/assets/Untitled%205.png)

After installing, let’s take a look at the app. Its clearly mentioned in the description of the challenge that we have to intercept the traffic.

![Untitled 6.png](Anchored/assets/Untitled%206.png)

So setup Burp Proxy to intercept the request. Check here to know how to do it: [How to setup Burp Proxy to Intercept Android Traffic](https://www.notion.so/BurpSuite-Proxy-Setup-for-android-46c60fee75134fc98dc7e6c2b9599039?pvs=21).

When I first opened the app, the app showed a registration page for early access, so I used a random mail to register.

![Untitled 7.png](Anchored/assets/Untitled%207.png)

But I wasn’t able to capture the request using Burp Suite.

![Untitled 8.png](Anchored/assets/Untitled%208.png)

The above error was thrown by burpsuite, which states that certificate/SSL pinning is performed to prevent interception of requests.

Since this device is not rooted, we can’t use frida to bypass certificate pinning. We have to find some other way to intercept the SSL traffic.

On searching google to find a way to intercept the traffic, found the following: [https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/make-apk-accept-ca-certificate#manual](https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/make-apk-accept-ca-certificate#manual)

According to the above article, we can make the apk to accept our burp CA certificate in two methods, either manually or using a tool. We will see both methods here:

### Method - 1: Using the apk-mitm tool

- First install the tool from the following repo: [https://github.com/shroudedcode/apk-mitm](https://github.com/shroudedcode/apk-mitm)
- Next run the tool.
  ![Untitled 9.png](Anchored/assets/Untitled%209.png)
- Now install the patched apk file to the emulator.

> [!important]  
> NOTE: Don’t forget to uninstall the original apk before installing the patched apk.

![Untitled 10.png](Anchored/assets/Untitled%2010.png)

- Now run the app and try to capture the request, if it didn’t work try to close the app completely [ don’t forget to clear from recent apps ] and then try once again.
- You can see the request in the HTTP history tab of the proxy menu, and if you check the raw request you can find the flag.

![Untitled 11.png](Anchored/assets/Untitled%2011.png)

### Method - 2: Manually patching the apk

- If we take a look at the `AndroidManifest.xml` file, you can see that the `networkSecurityConfig` option is already configured.

![Untitled 12.png](Anchored/assets/Untitled%2012.png)

- So we can directly modify the `network_security_config.xml` file located at `res/xml` path.

![Untitled 13.png](Anchored/assets/Untitled%2013.png)

- Open the `network_security_config.xml` file in your favourite text editor, modify the `src` value of the certificates tag to `user` and save the file.

![Untitled 14.png](Anchored/assets/Untitled%2014.png)

- Now it’s time to recompile the extracted apk files to again to apk. To do that go to the parent directory where the extracted apk files directory is located and run the command `apktool b <extracted_directory_path>`, to compile the files to apk.

![Untitled 15.png](Anchored/assets/Untitled%2015.png)

- The new built apk is located at the `Anchored/dist/` path.

![Untitled 16.png](Anchored/assets/Untitled%2016.png)

- Now its time to sign the apk. To do that, first we need a key. We can generate a key using the `keytool` tool. The command is `keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000`

![Untitled 17.png](Anchored/assets/Untitled%2017.png)

- After generating the key, its time to sign the apk using the `jarsigner` tool. The command is `jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore Anchored.apk alias_name` , Enter the password that you used while creating the key.
- Now we have done all the necessary steps and have successfully patched the file. Now its time to install the application.

> [!important]  
> NOTE: Don’t forget to uninstall the original apk before installing the patched apk.

![Untitled 18.png](Anchored/assets/Untitled%2018.png)

- Now run the app and try to capture the request, if it didn’t work try to close the app completely [ don’t forget to clear from recent apps ] and then try once again.
- You can see the request in the HTTP history tab of the proxy menu, and if you check the raw request you can find the flag.

![Untitled 19.png](Anchored/assets/Untitled%2019.png)

We have successfully obtained the flag……

Thank You…..
