# Task 16 - Under Construction

Category: Boot2Root
Difficulty: Easy

# nmap

As per usual we start with an nmap scan of the target.

![1](https://github.com/user-attachments/assets/0e813d77-813e-49a8-9229-741bd3f02441)


Okay, so only SSH and HTTP. We'll I guess it's off to check out the site then.

# http

We land on this page.

![2](https://github.com/user-attachments/assets/9d9bde3f-3464-4bba-b247-eeb9335939ac)


As we look through the site we find something of interest. The URL looks really interesting in the about-section.

```
view.php?page=about.php
```

A .php file that takes a parameter to show something. What if we mess with it.

![3](https://github.com/user-attachments/assets/11e1bcd4-e34d-42f9-9a93-520d9d50825d)


Nice! We have local file inclusion. This could come in handy. And we find that we have 2 users "dev" and "ubuntu". But while we're exploring the target through this we'll start enumerating the site for hidden files in the mean time with gobuster.

```
gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-words.txt -u http://10.10.209.64/
```

I couldn't really figure out much I could use the LFI for but it kept me busy at least for a while. And when I got back to the gobuster-results I found that it had found a folder named keys.

![4](https://github.com/user-attachments/assets/34624685-b7a8-4e35-aab1-72aaf6225b3c)


![5](https://github.com/user-attachments/assets/5dbadc22-608c-493b-951c-f016ea5931fd)


When checking it out we find that it contains several files, but only one of decent size. So let's see what it contains.

It appears to be a private SSH key. That's awesome, should give us a way in. So let's download it to our local machine.

```
wget http://10.10.209.64/keys/key_09
```

So now we have a key and we have 2 usernames to choose from. Let's start with the first one.

# ssh

```
ssh dev@10.10.209.64 -i key_09
```

Ah, of course we get an error. We have to change the permissions on the key too 600 for it to be considered valid.

```
chmod 600 key_09
ssh dev@10.10.209.64 -i key_09
```

Boss, we're in! And listing the contents we see that user.txt is here in the home directory of dev. So we can just read it and submit the flag.

So what do we do now? What do we always do when we get into a system? We check if we can sudo.

```
sudo -l
```

![6](https://github.com/user-attachments/assets/439586ed-c3c9-49d7-a2cd-afccfbb21657)


Ah, we're actually able to start vi as any user without submiting a password. This is great! Now we just have to find a way to leverage this to our advantage. Let's check out GTFObins to see if we're able to gain root through this.

https://gtfobins.github.io/gtfobins/vi/#sudo

So we can use it to spawn a shell with root privileges. Seems exactly like what we would want to do.

```
sudo vi -c ':!/bin/bash'
```

![7](https://github.com/user-attachments/assets/a7f5d900-fdbf-4523-9636-ee9cde18c01e)


And we have gotten root on this machine as well. Now we can just get the root-flag and get out of here.

# final thoughts

I found this machine to be somewhat easier than the last one actually even though this one was supposed to be more difficult. It might be because I found the LFI pretty much straight away. But overall a pretty satisfying machine to work through.
