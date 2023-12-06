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

![Untitled.png](Manager/assets/Untitled.png)

Next I used apktool to extract the apk file.

![Untitled 1.png](Manager/assets/Untitled%201.png)

After extracting the apk, I first took a look at the `AndroidManifest.xml` file to check whether there is any minimum API level or android version is required to run the app.

![Untitled 2.png](Manager/assets/Untitled%202.png)

There was nothing mentioned about that. So, In my case I installed the apk on an Android 6 / API 23 Virtual Device.

To install the `apk`, I used `adb`.

![Untitled 4.png](APKrypt/assets/Untitled%204.png)

Next I just take a look at the app and its functionality.

Initially, when I first Opened the application, it asked for the IP address and port number to connect to the server.

![Untitled 3.png](Manager/assets/Untitled%203.png)

Start the machine instance in HackTheBox and enter the IP address and port number and press connect.

![Untitled 4.png](Manager/assets/Untitled%204.png)

![Untitled 5.png](Manager/assets/Untitled%205.png)

After connecting to the server, the next page is a Login/Register page.

![Untitled 6.png](Manager/assets/Untitled%206.png)

Next, I registered a new user to check the functionality.

I also set up the burp proxy, to intercept the traffic.

For that check this Blog: [How to setup Burp Proxy to Intercept Android Traffic](https://www.notion.so/BurpSuite-Proxy-Setup-for-android-46c60fee75134fc98dc7e6c2b9599039?pvs=21)

![Untitled 7.png](Manager/assets/Untitled%207.png)

![Untitled 8.png](Manager/assets/Untitled%208.png)

You can see that the register request that is send to the server. After successfully registering the new user, the application shows a Manager page, which contains the ID, Username, Password and Role fields.

In this page only the Password is editable and updatable.

![Untitled 9.png](Manager/assets/Untitled%209.png)

Let’s try to update the password and capture the request to see what details are sent to the server.

![Untitled 10.png](Manager/assets/Untitled%2010.png)

You can see that the request is made to `manage.php` , with username and password fields.

Let’s try to change the admins password by tampering this request. To do that first send this request to the `repeater` tab, and modify the `username` to `admin` and set the password as your wish and send the request.

![Untitled 11.png](Manager/assets/Untitled%2011.png)

You can see the response that the password is updated successfully.

Now let’s try to login as admin with the updated credentials.

![Untitled 12.png](Manager/assets/Untitled%2012.png)

And we have successfully logged in.

![Untitled 13.png](Manager/assets/Untitled%2013.png)

We have successfully obtained the flag. If you can’t able to copy the flag, try to make the login request again using the repeater tab, from the response tab you can copy the flag from the raw response.

  

Thankyou…..