---
layout: post
title: "Redis Database Navigation"
date: 2025-04-30
categories: Cybersecurity 
---

There are different types of databases. One of them is called ‘Redis’ (Remote Dictionary Server). It uses RAM to store data, which makes it very fast. Databases like ‘Redis’ are also called ‘In-memory’ databases. Other database types use Hard-Drives or SSD’s. Other traditional databases are MySQL and MongoDB. Much of the info here was learned from Redeemer on hackthebox.com

A reason for using ‘Redis’ is to leverage the speed of RAM. It can cache data on it. The type of data to cache would be high volume requests. Like a website that returns a value for price(s). Perhaps a high volume of people have been seeking the price of an item. When the user clicks the item/price the website may be written to 1st check if the requested price(s) are in ‘Redis’.  If not then it will check the other databases MySQL or MongoDB. 

When the value is loaded from the database it is then stored in ‘Redis’ for short periods of time. It can be stored, for seconds, minutes or hours. If a similar request is activated from a web-user in the time-frame, it’s already stored on the RAM and can be retrieved very quickly. High traffic websites can use the ‘Redis’ design for greatly increased speeds for users to retrieve data. 

Here’s my experience enumerating and breaching a ‘Redis’ server. 
 
nmap -sV {target_IP} did not return results. Next a full port scan, nmap -p- -sV {target_IP}.

After using the 2nd option and letting it scan to 24% complete, 5 hours went by. I decided to abort the scan. Maybe there is a different option to use with nmap? No, the next step should be, disconnecting from the current running process of ‘openvpn’ and start a new session. Perhaps the session from yesterday isn’t stable or lost connection. I reconnect with:

sudo openvpn {filename.ovpn}

After reconnecting and running nmap again the scan completes to about 83% in a minute. However nmap waits if a port doesn’t reply for a full 5 seconds before moving to the next! It has 65,535 to scan! I search how to speed it up. With the -T4 command a fast scan, and -T5 the fastest, we can adjust the scan to complete in a reasonable amount of time.  

Another option to speed it up is to limit the port range: nmap -p 1-1000 -sV {target_IP}. I try the -T5 option and the scan completes in about 2 minutes. 

Next I wonder since ‘Redis’ uses key-value data, I wonder if that means it can be created in Python with a dictionary. Which would be nice to know since I program with Python and would like to learn more about its capabilities. The answer is YES! When working with ‘Redis’ in python, you 1st build the data in Python dictionaries, then send it to ‘Redis’. Next let’s take a closer look at the process to understand how it works conceptually.

The syntax in python for creating a dictionary is key:value. Such as “Username”:“Jason”. The key is Username and the value is Jason. Once the dictionary is loaded in Python then each key:value pair from ‘redis’ can be interacted with using Python. Let’s delve deeper into the matter by examining the coding. pip install redis can be used in the terminal window of PyCharm. In the code we:

The following section about Python interaction with Redis was partially learned from ChatGPT.

import redis

##Connect to Redis server (default is localhost:6379)  ## It lets your python script connect to and interact with a Redis database.## 

#The line here creates a connection object (r) to a Redis server.  r is the variable. 
#host=’localhost’   ---- connects to Redis running on your own machine!
#Redis has a default port number of 6379.
#db: Redis supports multiple databases (0 – 15 by default). This one is connected to database number 0   
##so basically now the ‘r’ variable, acts like your ‘remote control’ to talk to Redis!##
r = redis.Redis(host='localhost', port=6379, db=0)

##Create a Python dictionary##
#with ‘data’ as the variable we are adding to the dictionary! We are adding two ‘key-value’ pairs:
##"username" maps to "yakov"
##"role" maps to "admin"
data = {"username": "yakov", "role": "admin"}

STORE Key-Value in Redis
#Now that we’ve added 2 ‘key-values’
#We use a for loop that will iterate each ‘key’ and ‘value’. In the ‘data’ dictionary.
#.items() returns the ‘key-value’ pairs such as (“username”, “yakov”).
for key, value in data.items():
    # r.set ‘stores/saves’ them in Redis!!
    r.set(key, value)

((so what it’s doing is: 
r.set("username", "yakov")
r.set("role", "admin")
))

RETRIEVE VALUES FROM Redis:
#This line asks Redis for the value associated with the key ‘username’. 
#r.get fetches the value of ‘username’ ---- Redis will return binary data. Bytes.
#so output should be seen as: b’yakov’
##if interested in converting it to a normal string: print(r.get(‘username’).decode()) #output: yakov##
print(r.get('username'))  # Output: b'yakov'

Okay now that we’ve taken a closer look at the inner dimension of interacting with ‘redis’ let’s move on and find a method of entry into the redis server. Since the database of redis is stored in RAM and writes the content of the RAM to a hard-disk or SSD, it creates more storage devices to access, increasing our opportunity to infiltrate. If we can’t gain entry to the RAM we can try the hard-disks instead. 

A script is used to interact with the redis server remotely. So let’s keep in mind the server listens for connections through the CLI or clients. I downloaded redis-cli.

sudo apt install redis-tools

Connect to the redis server with the -h option for host:

redis-cli -h {target_IP}

Scrolling down to the section # Keyspace shows us stats on each database. Including the number of keys and the number of keys with an expiration. Only 1 database exists. We can select the database by using ‘select’ command, followed by the database number that we desire to select. 

Select 0

Now we can list all the keys in the database.

Keys *
 
The keys are then listed and we can download them with ‘get’. 

This exercise taught me tuning a port scan is critical for saving time. Additionally I learned how to interact with a database in Python and how to save data and send it to redis. I think the knowledge of database management could lead to ideas on how to create a database business like Oracle. But would God deem me deserving of creating that kind of business when I eat non kosher? Larry Ellison very likely eats non kosher and he created Oracle. So I can perhaps take that one off the list of things held against me. #the struggles of a Jewish life. 
