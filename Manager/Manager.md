---
Platform: HackTheBox
Category: Android
Difficulty: Easy
Status: Rooted/Finished
Type: Challenge
Title: Manager
tags: 
CreatedOn: 04-12-2023
---
# Manager

Hey everyone, in this write-up we will be solving an HTB challenge Manager.

Link to the challenge: [https://app.hackthebox.com/challenges/283](https://app.hackthebox.com/challenges/283)

  

Let’s Start!!!!!!

  

First download and extract the given file.

From the README.txt file, we can see that the application supports API level 29 or earlier.

![[Manager/assets/Untitled.png|Untitled.png]]

Next I used apktool to extract the apk file.

![[Manager/assets/Untitled 1.png|Untitled 1.png]]

After extracting the apk, I first took a look at the `AndroidManifest.xml` file to check whether there is any minimum API level or android version is required to run the app.

![[Manager/assets/Untitled 2.png|Untitled 2.png]]

There was nothing mentioned about that. So, In my case I installed the apk on an Android 6 / API 23 Virtual Device.

To install the `apk`, I used `adb`.

![[APKrypt/assets/Untitled 4.png|Untitled 4.png]]

Next I just take a look at the app and its functionality.

Initially, when I first Opened the application, it asked for the IP address and port number to connect to the server.

![[Manager/assets/Untitled 3.png|Untitled 3.png]]

Start the machine instance in HackTheBox and enter the IP address and port number and press connect.

![[Manager/assets/Untitled 4.png|Untitled 4.png]]

![[Manager/assets/Untitled 5.png|Untitled 5.png]]

After connecting to the server, the next page is a Login/Register page.

![[Manager/assets/Untitled 6.png|Untitled 6.png]]

Next, I registered a new user to check the functionality.

I also set up the burp proxy, to intercept the traffic.

For that check this Blog: [How to setup Burp Proxy to Intercept Android Traffic](https://www.notion.so/BurpSuite-Proxy-Setup-for-android-46c60fee75134fc98dc7e6c2b9599039?pvs=21)

![[Manager/assets/Untitled 7.png|Untitled 7.png]]

![[Manager/assets/Untitled 8.png|Untitled 8.png]]

You can see that the register request that is send to the server. After successfully registering the new user, the application shows a Manager page, which contains the ID, Username, Password and Role fields.

In this page only the Password is editable and updatable.

![[Manager/assets/Untitled 9.png|Untitled 9.png]]

Let’s try to update the password and capture the request to see what details are sent to the server.

![[Manager/assets/Untitled 10.png|Untitled 10.png]]

You can see that the request is made to `manage.php` , with username and password fields.

Let’s try to change the admins password by tampering this request. To do that first send this request to the `repeater` tab, and modify the `username` to `admin` and set the password as your wish and send the request.

![[Manager/assets/Untitled 11.png|Untitled 11.png]]

You can see the response that the password is updated successfully.

Now let’s try to login as admin with the updated credentials.

![[Manager/assets/Untitled 12.png|Untitled 12.png]]

And we have successfully logged in.

![[Manager/assets/Untitled 13.png|Untitled 13.png]]

We have successfully obtained the flag. If you can’t able to copy the flag, try to make the login request again using the repeater tab, from the response tab you can copy the flag from the raw response.

  

Thankyou…..