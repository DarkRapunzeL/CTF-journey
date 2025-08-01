# Library
URL: https://tryhackme.com/room/bsidesgtlibrary

Difficulty: Easy

# nmap

As usual we start off with an nmap scan to see which services are available to be interacted with on the server.

```sudo nmap -A -T4 -oN nmap_init 10.10.30.53```

<img width="834" height="518" alt="library1" src="https://github.com/user-attachments/assets/d9807d2d-c4b5-4482-bf09-e743e0dfcdca" />

We see that we have both an HTTP server and SSH running. And there is one disallowed entry in the robots.txt file. So let's check out the HTTP server.

# http

While we browse the site we might as well start a scan to enumerate the server for files and directories.

```gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.30.53 -t 40```

Now let's see what we can gather from the site.

<img width="1117" height="607" alt="library2" src="https://github.com/user-attachments/assets/eba28697-dfdf-4ded-b6f2-290b58137fff" />

Seems to be some kind of blog, what can we learn from this? If we look at the comments we see that one is made from www-data and one is made from root. Which suggests that we might find some usernames readily available on the site. Where else can we see something like it? The author of the post.

Now there was the robots.txt as well. Let's see what it contains.

<img width="280" height="87" alt="library3" src="https://github.com/user-attachments/assets/acc521ee-93f5-4704-835c-25d5f8172f6f" />

Not much special. But kind of a weird entry with rockyou. Might suggest that we should bruteforce something with that wordlist.

So now we have a possible username. But we don't have any password. If we go back to the scan let's see if it found something interesting. Nope, nothing of note.

# ssh

Now we have a username and a reference to the wordlist rockyou.txt. So let's try to  brute force an SSH login.

```hydra -W /usr/share/wordlists/rockyou.txt -l <username we found> ssh://10.10.30.53```

After a little while we get confirmation of a pair of credentials. So now we just have to log in.

```ssh <username>@10.10.30.53```

And we enter our newly found password. And we're in. Here in the users home directory we find the user.txt file. So now we just need to get root and get the root.txt flag.

# priv esc

How are we going to do this? First off let's try looking to see if our user has any sudo privileges.

```sudo -l```

Ah, awesome! We see that it has the privileges to run python on a script that is in the users home directory where we have write privileges. This should be easily abusable. Let's make a backup of the original file as it is just good practice to do.

```mv bak.py bak.py.old```

Now we just need to remake the file with what we want it to do. So let's open up a file called bak.py in vim (in this case I found out that vim wasn't installed. So let's use vi instead)

```vi bak.py```

Now what can we do to give us root access? If you've worked with reverse shells before, you know that you need to upgrade it to a capable tty to be able to utilize some commands. And this knowledge comes in handy here. We'll just tell python to spawn us a bash session in the same way.

```
#!/usr/bin/python
import pty
pty.spawn("/bin/bash")
```
Now we got a fully working shell with root privileges. So let's get that root-flag.

# final thoughts
It took a little while for me to find the username. Since there were so much lorem ipsum everywhere, it kind of muddled the waters and drowned out what information there were on the site. But in the end it was extremely straight forward and a room I'd absolutely would recommend to a beginner.
