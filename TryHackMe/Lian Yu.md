# nmap
I started out with an nmap-scan. It revealed 4 open ports.

# web enumeration
Since the server had port 80 open, I started a bruteforce enumeration of it with gobuster.

I found a directory called island. And through that I found what seems to at least be a "hidden" password: vigilante. And the word Lian_Yu in bold. Might it be the username? But since it speaks about a place, I'm not certain. And after trying it with both ssh and ftp. I didn't get any closer.

So I started another gobuster scan in this directory and found another one, 2100. There was supposed to be a video from youtube on this page. But it had been removed. But in the HTML-code I found an interesting comment.
 you can avail your .ticket here but how?
 
.ticket sounds like it might be a file-extension. Let's see if we can find anything. I'll fire up a dirbuster search, to see if we'll find any other sub-directories or if we can find a .ticket-file.

We found a file, green_arrow.ticket.

That contained something that looked encoded. After a few tries I found out that it was encoded with base58. !#th3h00d

# ftp

After stumbling around a bit, trying to figure out how to use the info we had gathered. I finally tried with to connect to the ftp-server with the credentials vigilante:!#th3h00d and downloaded the files that was listed. One of them was a .png that seemed corrupted.

I tried fixing that .png for a bit. But had no luck, so I figured that there were more files to check out before I got too focused in on that .png. And since there was a .jpg among the files. I decided to do a steghide -info on it. But it seemed to be password protected. So I tried cracking it with stegseek, and got a zip-file out of it. That contained 2 files, passwd.txt and shado.

I was stuck here for a while. I just couldn't figure out what username to use to connect to SSH, which it seemed like the password was for. After a while I remembered that on the ftp-server the working directory was /home/vigilante. What if I could cd up a step? There might be other users home-directories. And there was, now I had a username as well.


# ssh

Now that we're logged in on ssh on the server. I checked the listing of the current folder and found the user.txt flag. And as a standard mode of operation I checked if the user had any sudo-privileges. And sure enough it was able to run pkexec as root.

``sudo pkexec --user root /bin/bash``

Now I had a root-shell and was able to read the root.txt flag as well.


# final thoughts

It was quite a fun machine. It was kind of smooth sailing for the most part. I'll have to blame myself for getting stuck there a while, since I could have been way more thorough. Either way it was well spent time.
