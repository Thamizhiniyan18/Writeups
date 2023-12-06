---
Platform: HackTheBox
Category: Android
Difficulty: Easy
Status: Rooted/Finished
Type: Challenge
Title: Cat
tags: 
CreatedOn: 04-12-2023
---
# Cat

Hey everyone, in this write-up we will be solving an HTB challenge Cat.

Link to the challenge: [https://app.hackthebox.com/challenges/cat](https://app.hackthebox.com/challenges/cat)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![Untitled.png](Cat/assets/Untitled.png)

After extracting the zip, we can see a file named `cat.ab`. I used `file` command on the `Cat.ab` file.

![Untitled 1.png](Cat/assets/Untitled%201.png)

From the output of the file command, we can devise that the given file is android backup file.

You can extract file from a android backup using `android-backup-extractor` [ [https://github.com/nelenkov/android-backup-extractor/releases](https://github.com/nelenkov/android-backup-extractor/releases) ]. Download it.

![Untitled 2.png](Cat/assets/Untitled%202.png)

Using the command: `java -jar abe.jar unpack cat.ab file.tar` , you can convert a android backup file to a tar file.

![Untitled 3.png](Cat/assets/Untitled%203.png)

Now its time to extract the tar file using the command `tar xvf file.tar`.

![Untitled 4.png](Cat/assets/Untitled%204.png)

From the output of the `tar` tool, we can see that there are some images in the `shared/0/Pictures` directory.

![Untitled 5.png](Cat/assets/Untitled%205.png)

On taking a look at those images, one of the image contains the flag in it.

![Untitled 6.png](Cat/assets/Untitled%206.png)

We have successfully obtained the flag.

  

  

Thank you…….