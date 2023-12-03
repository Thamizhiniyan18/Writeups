---
Platform: HackTheBox
Category: Android
Difficulty: Easy
tags:
  - Android
  - certificate-pinning-bypass
  - frida
Status: Rooted/Finished
Type: Challenge
Title: Pinned
CreatedOn: 04-12-2023
---
# Pinned

Hey everyone, in this write-up we will be solving an HTB challenge Pinned.

Link to the challenge: [https://app.hackthebox.com/challenges/282](https://app.hackthebox.com/challenges/282)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![[Pinned/assets/Untitled.png|Untitled.png]]

From the README.txt file, we see can that the application supports API level 29 or earlier.

  

First I used `apktool` to extract the contents of the `apk` file.

![[Pinned/assets/Untitled 1.png|Untitled 1.png]]

After extracting the apk, I checked the `AndroidManifest.xml` file to look out for minimum required API level. But there was no minimum API level mentioned in the `AndroidManifest.xml` file.

![[Pinned/assets/Untitled 2.png|Untitled 2.png]]

So, I decided to use Android 6 ( API 23 ).

Next I installed the `pinned.apk` using adb on to the emulated android 6 device.

![[Pinned/assets/Untitled 3.png|Untitled 3.png]]

  

I opened the application in the android emulator and it opened with credentials filled by default.

![[Pinned/assets/Untitled 4.png|Untitled 4.png]]

I clicked on LOGIN and it toasted a message that “You are logged in”.

![[Pinned/assets/Untitled 5.png|Untitled 5.png]]

  

From the challenge description, we can see that the user had tried to intercept the request, but the connection seems to be secure.

![[Pinned/assets/Untitled 6.png|Untitled 6.png]]

Let’s try to intercept the login request using burpsuite.

For that check this Blog: [How to setup Burp Proxy to Intercept Android Traffic](https://www.notion.so/BurpSuite-Proxy-Setup-for-android-46c60fee75134fc98dc7e6c2b9599039?pvs=21)

Now Drop the above request and try to intercept the login request by trying to login from the pinned app.

But I can’t able to intercept the request. Instead burpsuite thrown the following error:

![[Pinned/assets/Untitled 7.png|Untitled 7.png]]

So let’s take a look at the code. I used `jadx` viewer to view the code. You can get it from here:

https://github.com/skylot/jadx

Open the apk in `jadx` Viewer. I first checked the `MainActivity.class` file and found the function that checks the credentials.

![[Pinned/assets/Untitled 8.png|Untitled 8.png]]

And also I found another function above the validation function which was generating SSL connection.

![[Pinned/assets/Untitled 9.png|Untitled 9.png]]

On further investing I found the `run()` function which runs while generating SSL connection in the `MainActivity` file:

![[Pinned/assets/Untitled 10.png|Untitled 10.png]]

The above function is a custom certificate pinning function which checks whether the certificate used is valid or not. To make burpsuite intercept the login request, we have to bypass the above check.

On searching for SSL pinning bypass in google, found the following: [https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/](https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/)

  

To use the above exploit, first we need to install `Frida`. Follow the below commands to install `frida`:

```
python3 -m venv env
source ./env/bin/activate
pip3 install frida-tools
```

After installing frida on your local system, next you have to install frida server on the android device. To do that follow the below steps:

1. Download the latest frida server release for android that matches your android device’s architecture from the following link: [https://github.com/frida/frida/releases](https://github.com/frida/frida/releases)
2. The file you download will be in `.xz` format. To extract it follow the steps:
    1. `sudo apt-get install xz-utils`
    2. `unxz frida-server-16.1.4-android-x86_64.xz`
    3. Rename the extracted file to `frida-server` using the command: `mv frida-server-16.1.4-android-x86_64 ./frida-server`
3. Next we have to push this `frida-server` file to the android device using the following command: `adb push frida-server /data/local/tmp/`
4. Now we have change the permissions of the `frida-server` file using the command: `adb shell "chmod 755 /data/local/tmp/frida-server"`
5. Now open `adb shell` and move to the directory where we pushed the `frida-server` and run the `frida-server` using the command: `./frida-server`.

![[Pinned/assets/Untitled 11.png|Untitled 11.png]]

1. Now open a new terminal and again initialise the python environment using the command: `source ./env/bin/activate`
2. Now run the command: `frida-ps -U`, to check whether the frida server is running. This command should list all the packages.

![[Pinned/assets/Untitled 12.png|Untitled 12.png]]

Now we have successfully established the connection to the frida-server. Now its time to run the exploit. Before running the exploit, let’s take a look at it:

![[Pinned/assets/Untitled 13.png|Untitled 13.png]]

We can see that it is generating the SSL using a custom certificate, which is the burpsuite ca certificate in our case.

So to successfully run the exploit first we have to push the burp certificate to the location mentioned in the script, with the same name as mentioned in the script. Since we have the burp suite certificate already in our local machine which we downloaded and renamed as `cacert.pem` , we can use the same.

To push the certificate to the android device, use the following command:

`adb push ./cacert.pem /data/local/tmp/cert-der.crt`

![[Pinned/assets/Untitled 14.png|Untitled 14.png]]

After successfully pushing the file, its time to execute the exploit. Make sure that you have turned burp intercept on and also make sure that you have connected to burp proxy in the android device.

Now run the following command to execute the payload:

`frida --codeshare pcipolloni/universal-android-ssl-pinning-bypass-with-frida -U -f com.example.pinned`

![[Pinned/assets/Untitled 15.png|Untitled 15.png]]

The exploit has executed successfully and is waiting for the request.

Now go to the Pinned application and try to login and check the captured request in burp:

![[Pinned/assets/Untitled 16.png|Untitled 16.png]]

We have successfully obtained the flag……

  

Thank you…….