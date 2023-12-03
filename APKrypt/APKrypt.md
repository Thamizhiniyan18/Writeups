---
Platform: HackTheBox
Category: Android
Difficulty: Easy
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Challenge
Title: APKrypt
tags: 
CreatedOn: 04-12-2023
---
# APKrypt

Hey everyone, in this write-up we will be solving an HTB challenge APKrypt.

Link to the challenge: [https://app.hackthebox.com/challenges/285](https://app.hackthebox.com/challenges/285)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![[APKrypt/assets/Untitled.png|Untitled.png]]

From the README.txt file, we can see that the application supports API level 29 or earlier.

![[APKrypt/assets/Untitled 1.png|Untitled 1.png]]

Next I used `apktool` to extract the apk file.

![[APKrypt/assets/Untitled 2.png|Untitled 2.png]]

After extracting the apk, I first took a look at the `AndroidManifest.xml` file to check whether there is any minimum SDK version is required to run the app.

![[APKrypt/assets/Untitled 3.png|Untitled 3.png]]

There was nothing mentioned about that. So, In my case I installed the apk on an Android 10 / API 29 Virtual Device with Google API’s.

To install the `apk`, I used `adb`.

![[APKrypt/assets/Untitled 4.png|Untitled 4.png]]

Next I just take a look at the app and its functionality.

![[APKrypt/assets/Untitled 5.png|Untitled 5.png]]

The app loads with a page, where we have to enter the VIP code to get the ticket. There is nothing else in the page except a submit button. So I moved on to analyse the decompiled `smali` code.

I first opened the apk with `jadx-gui` [ [https://github.com/skylot/jadx](https://github.com/skylot/jadx) ], to get the basic understanding of the application.

In the `MainActivity` class file, you can find the main function which checks whether the VIP code is valid or not and if valid, it returns the flag.

![[APKrypt/assets/Untitled 6.png|Untitled 6.png]]

- The if condition checks whether the MD5 version of the code that we input is equal to the MD5 hashed VIP code. For that, it uses the `equals` method.
- Now to bypass this check, we can modify the `equals` method to `notEqual`, so that even if the input is empty or invalid the flag will be shown.
- To modify the above condition, we have to first decompile the `apk` and modify the `smali` code of this particular if condition and recompile the `apk` and sign it with a key, to bypass the check.

To perform the above task, I used `APKlab` ( [https://github.com/APKLab/APKLab](https://github.com/APKLab/APKLab) ) tool to Modify the apk.

First open vscode and use the shortcut key `ctrl + shift + p` to open the command pallet and search for `APKLab: Open an APK` option and click it.

![[APKey/assets/Untitled 5.png|Untitled 5.png]]

Now locate the apk file and select it.

![[APKrypt/assets/Untitled 7.png|Untitled 7.png]]

Next leave the defaults in the popup and click OK.

![[APKey/assets/Untitled 7.png|Untitled 7.png]]

After pressing ok, `APKLab` will decompile the android application and will load a new window with the decompiled files.

I used the search feature in the vscode to find the condition, by looking out for the MD5 hashed version of the VIP code to which our input code is compared.

![[APKrypt/assets/Untitled 8.png|Untitled 8.png]]

From the results, we have found the if condition.  
Now its time to modify the if condition.

Replace the `if-eqz` to `if-nez`, which means `not equals` and save the file.

![[APKrypt/assets/Untitled 9.png|Untitled 9.png]]

Now it’s time to compile and sign the apk.

To do that select the `apktool.yml` file in the file explorer → right click to view the options → click on the option `APKLab: Rebuild the APK`

![[APKrypt/assets/Untitled 10.png|Untitled 10.png]]

After the build process is completed you should see a output similar to this:

![[APKrypt/assets/Untitled 11.png|Untitled 11.png]]

Now its time to install app. Before installing the modified version of the app, ensure that you have uninstalled the unmodified version of the app.

![[APKrypt/assets/Untitled 12.png|Untitled 12.png]]

After installing the app, open the app and press the submit button [ with or without the VIP code ( Both should WORK ) ].

![[APKrypt/assets/Untitled 13.png|Untitled 13.png]]

You can see the toast message with the flag.

We have successfully obtained the flag.

  

  

Thank You !!!!!!