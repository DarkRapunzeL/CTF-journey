# Åtkomst Nekad
## Information
Category: Misc

Difficulty: Medium

Attachments: Admittence papers, Database dump

## Summary
This was my favorite challenge that I did in this CTF. The story was that there was a Sysadmin that had been admitted to a psychiatric ward.
This caused problems since there was an SSH server that was integral to their work that they needed access to.
But it was only the admin that got admitted that had access to it. Therefore we had to try to unveil a way to get some access.
We could do this by using clues from the journal and brute forcing the password. Which let us in to the server where
we were able to use curl to contact an HTTP server running in the network, and finally exploit a vulnerability in PHPMyAdmin
to read the flag.

## How I went through it

### Getting started
Since we had these attachments that was were I started. Looking through the admittence papers there were some interesting comments written
by the doctors. The admin was a huge Pokémon fanatic, on the verge of obsession. There was one note about him using Vaporeon as his
standard username. So there we had a good lead straigh away. But we could also find out that he usually made up his passwords by
combining a pokémon and a year, as in Butterfree2001, Squirtle1998 and so on.

Now that I had an idea of what would be a likely composition of the password, I looked for a wordlist containing all 151 first gen
pokémons. Then I used this list to make a python script that took these and appended a year between 1900-2050. Might have been a bit
overkill with such a large span. But whatever. The script then made a new wordlist out of these combinations. And since
we had a dump of the users table from a SQL database. I looked through it to find the password hash for our target.

### Getting in to the server

<img width="868" height="58" alt="pwhash" src="https://github.com/user-attachments/assets/70695aeb-bce1-4757-9f40-f14d77125a14" />

Now that I had a likely username and a password hash. I decided to try to crack the hash with John The Ripper, using my newly
composed wordlist. This went like a charm. And I was able to get a cleartext password from the hash. Equiped with both a username
and a password. I tried to log in to the SSH server with this and it worked flawlessly.

Now that I had access to the SSH server I started to enumerate it. And found that there was a script in the users home directory.
Looking into this script it seemed like it was trying to find out which services was available in the network. Where two of the
entries were HTTP.

### The HTTP server and the exploit

With having the IP address to an internal HTTP server within the same network as the SSH server. My first thought was to set up a
tunnel via SSH to be able to access this server from my own host. This would seem to not work out though. As the ability to do forwarding
via SSH had been blocked. So I had to find some other way to keep going.

What I eventually did was try to enumerate what commands I had at my disposal on the server. And one of the ones I found was curl.
Which felt absolutely fantastic since I had an HTTP server to work with. So I tried fetching some pages from this server and found
that it was running PHPMyAdmin. And on one of the pages it also specified the version of it, 4.8.1. This made it trivial to look
for known vulnerabilities in the software. And I found out that there was a LFI-vulnerability. Which sounded like exactly what I needed.
Since I just had to read the flag file. Which I found the path to on one of the pages.

Now I hit a little snag. I looked around for a while to find an easily digestible construction of the payload needed to exploit the
vulnerability. After a while I found a PowerShell script that showed me exactly what I needed to know. 

<img width="986" height="92" alt="exploit" src="https://github.com/user-attachments/assets/8326b8f0-cf1f-499e-a1cd-bef269e3322e" />

Exploit time! I crafted
a payload and sent the request to the server via cURL. There it was. Another flag.

<img width="1049" height="131" alt="exploit2" src="https://github.com/user-attachments/assets/4ab40854-3e90-43cb-a69f-2e4ef3f4e65c" />


<img width="899" height="391" alt="flag" src="https://github.com/user-attachments/assets/00d1508e-110d-46ae-a639-d99ae3e8b44e" />
