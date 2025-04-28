---
layout: post
title: "SMB Access"
date: 2025-04-27
categories: Cybersecurity hacking
---

SMB (Server Message Block) – typically uses Port 445 TCP, as I learned from the ‘Dance’ exercise on hackthebox.com.

SMB storage on the network is called a ‘share’. A share is like a folder that can be accessed. Credentials are usually required for entry. However an administrator can make mistakes. Mistake possibilities are guest accounts and anonymous logins. 

To enumerate ‘shares’ I used smbclient. It can be installed with:

sudo apt-get install smbclient

Smbclient is a utility that allows access to SMB file systems. An example of how it works: a server is used to store files, a user on a PC (aka client), can access the server to retrieve or upload files. Remember a share is used on the SMB enabled storage/server on the network. It’s accessed by a client. 

Clients can create, edit, retrieve, and remove files on a share, that’s why authentication measures are required. After running nmap we see a clear possibility to exploit a weakness: 

Port: 445/tcp
State: open
Service: microsoft-ds?

Port 445/tcp is associated with SMB and the service ‘microsoft-ds’ has known weaknesses. 

I ran smbclient, it will try to connect to the remote server/host. It checks if authentication is needed. If required it asks for a password for your ‘local’ username. For someone reason it automatically used my local machines name. It didn’t ask for a username.

Let’s see what the shares are on the SMB server by using the command:

smbclient -L {target_IP}

Recall how an admin of SMB could make mistakes. He/She could have a setup for, guest accounts and anonymous logins. So let’s see if one of them can get us in!

4 shares populate. Which gives us 4 possibilities to attempt logging in. Next is the Foothold. Let’s search for improperly configured permissions in the attempts to login. I guess the easiest method is starting with a blank password for each username. Then see if it works on any of the ‘shares’. 

The command to enter a share:

smbclient \\\\{target_IP}\\ADMIN$

The WorkShares SMB ‘share’ was not configured well. Using a blank password succeeds. We are now logged into the SMB ‘share’.  I used the command ‘help’ to see what can be done next.  

After typing ‘ls’ we see 2 directories exist. Lets see what’s in them with ‘cd’. Then ‘ls’ to see the contents of each directory.  One of the folders has the flag in it. I captured it with ‘get’ then ‘cat’ to open and read.  

We have successfully infiltrated the SMB ‘share’ on the server. I think the exercise highlights the importance of authentication. 

Thanks for reading and stay tuned for more. 
