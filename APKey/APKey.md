---
Platform: HackTheBox
Category: Android
Difficulty: Easy
Status: Rooted/Finished
Type: Challenge
Title: APKey
tags: 
CreatedOn: 04-12-2023
---

# APKey

Hey everyone, in this write-up we will be solving an HTB challenge APKey.

Link to the challenge: [https://app.hackthebox.com/challenges/240](https://app.hackthebox.com/challenges/240)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![Untitled.png](APKey/assets/Untitled.png)

Next I opened the apk with `jadx-gui` [ [https://github.com/skylot/jadx](https://github.com/skylot/jadx) ].

![Untitled 1.png](APKey/assets/Untitled%201.png)

I checked the `AndroidManifest.xml` file and found that we need at least API version 16 to run this app and the target API version is 30 i.e., Android 10. So, I installed the app on Android 10 emulator.

![Untitled 2.png](APKey/assets/Untitled%202.png)

Now let’s take a look at the app.

![Untitled 3.png](APKey/assets/Untitled%203.png)

It opened with a login page, and if we enter wrong credentials, it throws a toast stating “Wrong Credentials”.

Nothing else is found in the app.

Now let’s take a look at the source code of the application.

![Untitled 4.png](APKey/assets/Untitled%204.png)

From the MainActivity class file, we can see a if condition, which looks out for the user `admin` and checks whether the md5 version of the entered password matches the predefined hash in the second if condition, and if the condition satisfies the application throws a toast with the key.

- The if condition checks whether the MD5 version of the code that we input is equal to the MD5 hashed VIP code. For that, it uses the `equals` method.
- Now to bypass this check, we can modify the `equals` method to `notEqual`, so that even if the input is empty or invalid the flag will be shown.
- To modify the above condition, we have to first decompile the `apk` and modify the `smali` code of this particular if condition and recompile the `apk` and sign it with a key, to bypass the check.

To perform the above task, I used `APKlab` ( [https://github.com/APKLab/APKLab](https://github.com/APKLab/APKLab) ) tool to Modify the apk.

First open vscode and use the shortcut key `ctrl + shift + p` to open the command pallet and search for `APKLab: Open an APK` option and click it.

![Untitled 5.png](APKey/assets/Untitled%205.png)

Now locate the apk file and select it.

![Untitled 6.png](APKey/assets/Untitled%206.png)

Next leave the defaults in the popup and click OK.

![Untitled 7.png](APKey/assets/Untitled%207.png)

After pressing ok, `APKLab` will decompile the android application and will load a new window with the decompiled files.

I used the search feature in the vscode to find the condition, by looking out for the MD5 hashed version of the password to which our input code is compared.

![Untitled 8.png](APKey/assets/Untitled%208.png)

![Untitled 9.png](APKey/assets/Untitled%209.png)

From the results, we have found the if condition.  
Now its time to modify the if condition.

Replace the `if-eqz` to `if-nez`, which means `not equals` and save the file.

![Untitled 10.png](APKey/assets/Untitled%2010.png)

Now it’s time to compile it into apk,sign the apk and install the apk onto the emulator.

To do that select the `apktool.yml` file in the file explorer → right click to view the options → click on the option `APKLab: Rebuild and Install the APK`

> [!important]  
> NOTE: Before performing the above task, don’t forget to uninstall the original version of the app from the emulator.![Untitled 11.png](APKey/assets/Untitled%2011.png)  

![Untitled 12.png](APKey/assets/Untitled%2012.png)

After the build process is completed and the app is successfully installed, you should see a output similar to this:

![Untitled 13.png](APKey/assets/Untitled%2013.png)

Now open the app and type the user name as `admin` and enter some random password.

![Untitled 14.png](APKey/assets/Untitled%2014.png)

You can see the toast message with the flag.

We have successfully obtained the flag.

  

  

Thank You !!!!!!