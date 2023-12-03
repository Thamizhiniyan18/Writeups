---
Platform: HackTheBox
Category: Crypto
Difficulty: Easy
Status: Rooted/Finished
Type: Challenge
Title: Weak RSA
tags: 
CreatedOn: 04-12-2023
---
# Weak RSA

**Link to the Challenge:** [https://app.hackthebox.com/challenges/6](https://app.hackthebox.com/challenges/6)

  

First download the given file.

The give file is a zip file. Extract the zip file using the following command and the given password:

Command: `unzip <zip_file>`

password: `hackthebox`

  

The zip file contains two files: `flag.enc` and `key.pub`

From the challenge name, we can assume that the encryption used to encrypt the `flag.enc`

file is RSA.

We need the private key to decrypt the `flag.enc` file, since the public key is given which shows that asymmetric encryption is used to encrypt the file `flag.enc`.

The public key in the `key.pub` is smaller, i.e., of less length. Its possible to retrieve private keys from short or smaller public keys.

![[Weak RSA/assets/Untitled.png|Untitled.png]]

We can do this by using the tool [RsaCtfTool](https://github.com/RsaCtfTool/RsaCtfTool). To install and run this tool execute the commands given below:

```
// Installing the necessary dependencies
sudo apt-get install libgmp3-dev libmpc-dev

// Downloading the repository
git clone https://github.com/RsaCtfTool/RsaCtfTool.git

// Creating a virtual environment to download the requirements for the tool to run
python3 -m venv env

// Activating the virtual environment
source ./env/bin/activate

// NOTE: To deactivate the virtual environment after job is done use the following command:
deactivate

// Installing the requirements
pip3 install -r ./RsaCtfTool/requirements.txt
```

  

Now we have successfully installed the tool. Now we have to retrieve the private key by using the following command:

```
python3 ./RsaCtfTool/RsaCtfTool.py --publickey ./key.pub --private

// Here 
// --publickey - mention the path to public key file.
// --private - Display private key if recovered.
```

![[Weak RSA/assets/Untitled 1.png|Untitled 1.png]]

We have successfully recovered the private key. Now copy and store the private key in a `private.key` file.

![[Weak RSA/assets/Untitled 2.png|Untitled 2.png]]

Now we can use this private key to decrypt the contents of the `flag.enc` file. We can do this by using the `openssl` utility.

Use the following command to decrypt the `flag.enc` file and store the output in `flag.txt` file.

```
openssl pkeyutl -in flag.enc -out flag.txt -decrypt -inkey ./private.key

// Here
// pkeyutl - Public key algorithm cryptographic operation command.
// -in - input file.
// -inkey - input private key file.
// -decrypt - Decrypt input data with private key.	
// -out - Decrypted output file name and location.
```

![[Weak RSA/assets/Untitled 3.png|Untitled 3.png]]

We have successfully decrypted the file and got the flag!!!!!

  

Thankyou !!!!!