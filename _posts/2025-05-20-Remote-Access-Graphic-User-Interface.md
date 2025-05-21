---
layout: post
tite: "How I Accessed a Remote Host GUI"
date: 2025-05-20
categories: htb redream, cybersecurity, computers, pen testing
---

Hello everyone and welcome back to another exciting entry in my cybersecurity blog! Today, I tackled a Hack The Box challenge where five different ports were open. Each presenting its own potential vulnerability. At first, I wasn't sure which direction to pursue. But after thoroughly researching each service, the correct path become transpicuous. Intriguingly it lead to remotely accessing a Graphic User Interface (GUI)!

Before diving into the exploit, let's breifly explore Remote Access Protocols. Protocols like Telnet (port 23/TCP) and SSH (port 22/TCP) are commonly used for remote access and system management. Telnet while historically popular, is considered insecure today because it transmits data, including credentials, in plaintext. In contrast, Secure Shell (SSH) provides encrypted communication and secure authentication using public-key cryptography, making it the preferred chocie for secure remote sessions. Let’s now take a compendious look at how public-key encryption works and why it's fundamental to secure access protocols like SSH.

A client can create a key pair using:

Step 1 
`ssh-keygen`
This will generate:
id_rsa – your private key (do not share)
id_rsa.pub – your public key (safe to share)

Step 2
Copy the Public Key to the server.
The command: `ssh-copy-id user@remote_server`
    • This command will tell the server your public key and trust it. 
    • The public key is copied to
        ◦  the server’s  ~/.ssh/authorized_keys file

Step 3
The client will now initiate SSH connection.
Command: `ssh user@remote_server`
    • Now the ‘server’ will send a challenge. 
    • The Server generates a random ‘encrypted’ challenge using the ‘client’s public key’. 
    • Only someone with the matching ‘private key’ can decrypt it. 	
        ◦ The client now decrypts it.
        ◦ The SSH client uses the private key to decrypt the challenge. 
        ◦ The decrypted value is sent back. 
            ▪ The Server compares it with what it originally sent. 
            ▪ If they match the authentication is successful. 
Step 4
Session encryption:
After authentication, SSH uses a symmetric encryption key (like AES) for the session itself. 
    • SSH uses Diffie-Hellman (DH) or Elliptic Curve Diffie-Hellman (ECDH) to agree on a shared session key.
    • This key is used for encrypting the data going forward (faster than public-key).

I delved into Public-Key-Encryption to learn about the complexities of SSH security. It appears to be a secure method of remote connection opposed to Telnet. Luckily, this challenge doesn't focus on testing the parameters of SSH, exploiting it would take way more time and skill than I'm ready to spend at the moment. 

The reason we aren't focued on SSH or Telnet is becuase the goal is to use a unique feature called 'Display Projection.' SSH and Telnet only offer access to a remote terminal (CLI). To do this we have a tool called xfreerdp. Tools like these are called Remote Desktop Tools. 

Two of the most prevalent Remote Desktop Tools are Teamviewer and Windows Remote Desktop Connection, once known as Terminal Services Client.

Okay let’s being the Enumeration:
I started with a ping. It worked then I moved to the next step. An nmap scan.
`Sudo nmap -sV {target_IP}` The scan returns 5 open ports. From using the -sV tag we can see the versions and services running on each open port. Examining them may reveal potential vulnerabilities due to outdated software.

The 5 ports are: 135/TCP, 139/TCP, 445/TCP, 3389/TCP, 5357/TCP, I was apprehensive about researching 5 ports yet resolute. Expecting to make a new exciting discovery I began. I inspected them in order until I reached the 4th one, port 3389/TCP. It's asociated with Remote Desktop Protocol. Boom, that's it, I identified the port to infiltrate! 

Out of curiosity I actually looked up port 5357/TCP, it also has weaknesses. 

Now that I've researched the ports, let’s see if we can check for misconfigurations by trying to connect to port 3389 without valid credentials. Kali Linux has the tool, `xfreerdp3` a script for remote login. It allows users to connect to an RDP server, such as those built into many editions of Windows. I typed it in the terminal window and the scripts help menu outputs.

If it has to be installed:
`sudo apt-get install freerdp2-x11` 

First I tried to form an RDP session by not providing any additional info. The script will automatically use your own username as the login username for the RDP session. 
`/v:{target_IP}` Specifies the target IP of the host we would like to connect to.

After running the command it does not accept my username. The next step would be to attempt a login as the Administrator. We can change the script xfreerdp to bypass requirements for a security certificate. Normally the script would ask for them. To change the script a few commands are needed:

`/cert:ignore` 		-- This is the code that will tell the script to ignore the security certificate. 
	
`/u:Administrator`	-- This will change the login username to ‘Administrator.’

`/v:{target_IP}`	-- Target IP of the host we want to connect to. 

All together the command should look like:

`xfreerdp3 /v:{target_IP} /cert:ignore /u:Administrator`

The output I received was server. I pressed enter, and it prompted for a password. Pressing enter again, logged me in and the remote desktop GUI was activated. The remote hosts display becamev visible, and the flag was located directly on the desktop. This login behavior, gaining access without credentials, is a clear misconfiguration. 

This exercise reinforced the critical importance of proper credential management in system security. A simple misconfiguration granted unauthorized access to a remote system. The time spent researching open ports proved valuable. Port 3389/TCP which supports Windows Remote Desktop Protocol (RDP), was the primary gateway. Additionally exploring the roles and potential vulnerabilities of other open ports (139, 445, and 5357) offered useful insight. Even though they weren't necessary for this specific challenge, they may prove beneficial in future scenarios. 

Thanks for reading, and until next time stay curious and stay secure.    
