---
Platform:
  - HackTheBox
Category: Android
Difficulty: Easy
Status: Rooted/Finished
Blog Posting: Not Posted
Type: Challenge
---
Hey everyone, in this write-up we will be solving an HTB challenge Cat.

Link to the challenge: [https://app.hackthebox.com/challenges/cat](https://app.hackthebox.com/challenges/cat)

  

Let’s Start!!!!!!

  

First download and extract the given file.

![[Cat/assets/Untitled.png|Untitled.png]]

After extracting the zip, we can see a file named `cat.ab`. I used `file` command on the `Cat.ab` file.

![[Cat/assets/Untitled 1.png|Untitled 1.png]]

From the output of the file command, we can devise that the given file is android backup file.

You can extract file from a android backup using `android-backup-extractor` [ [https://github.com/nelenkov/android-backup-extractor/releases](https://github.com/nelenkov/android-backup-extractor/releases) ]. Download it.

![[Cat/assets/Untitled 2.png|Untitled 2.png]]

Using the command: `java -jar abe.jar unpack cat.ab file.tar` , you can convert a android backup file to a tar file.

![[Cat/assets/Untitled 3.png|Untitled 3.png]]

Now its time to extract the tar file using the command `tar xvf file.tar`.

![[Cat/assets/Untitled 4.png|Untitled 4.png]]

From the output of the `tar` tool, we can see that there are some images in the `shared/0/Pictures` directory.

![[Cat/assets/Untitled 5.png|Untitled 5.png]]

On taking a look at those images, one of the image contains the flag in it.

![[Cat/assets/Untitled 6.png|Untitled 6.png]]

We have successfully obtained the flag.

  

  

Thank you…….