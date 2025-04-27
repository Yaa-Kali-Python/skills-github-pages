---
layout: post
title: "FTP Vulnerabilities"
date: 2025-04-27
categories: FTP

I recently completed the very easy 'Fawn' exercise and learned an important point about FTP security vulnerabilities:
• FTP is designed to operate on port 21 and, by default, does not use encryption.

• The default username is often programmed as 'anonymous', allowing users to log in without needing proper credentials.

 • If attempting an FTP brute-force login, it’s a good idea to first try 'anonymous' as the username. If accepted, any password is typically allowed, making it extremely easy to gain access.
 
 • Additionally, I learned that with FTP, you cannot simply 'cat' a file directly from the server. You must first download the file using the 'get' command before you can view its contents.
 
I just wanted to share this information in case it might be helpful for anyone else working through the exercise or interested in exploiting FTP.

Thanks for tuning in and have a nice day. 
