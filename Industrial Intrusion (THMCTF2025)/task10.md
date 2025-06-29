# Task 10 - Brr v1

Category: Web
Difficulty: Easy

# nmap

As per usual we scan the target with nmap to see what we're dealing with.

```
sudo nmap -sC -sS -sV -O -Pn -T4 -On nmap_init 10.10.4.89
```

We find that ports 22, 80, 5901 and 8080 are open. Since it is supposed to be a web challenge we start out with 80 and 8080 which are both running HTTP.

# http

When trying to access port 80 we find out that it isn't allowing GET as method. So in this early stage we'll just check out 8080 then and come back to 80 later.

![1](https://github.com/user-attachments/assets/988dac71-f36d-4500-a2ac-b517ef168386)


At 8080 we're greeted with a log in. We don't have any credentials at the moment. But we see that it is running something called ScadaBR. And by a little quick googling we find the default credentials for it. So let's try it.

And it works. So now what do we do? Let's see what information we can gather.

![2](https://github.com/user-attachments/assets/5916f486-9ee9-45fe-8400-f104a0110fe0)


By going to the help page, we see that the version is 0.7. That's good, now we can start to look for known vulnerabilities in this version. We quickly find that it is vulnurable to CVE-2021-26828. Where remote users can upload and execute JSP-files. Sounds promising.

And after a bit more looking into it we find this
https://github.com/hev0x/CVE-2021-26828_ScadaBR_RCE/tree/main

A ready script for exploiting it. Unfortunately when we try to execute it, we get complaints about indentation. So we'll have to tidy it up a bit. But finally we're able to run it

```
python3 LinScada_RCE.py 10.10.4.89 8080 admin admin
```

![3](https://github.com/user-attachments/assets/b2d683c0-17eb-4941-9dd4-dfd86c112ee9)


Now we're getting somewhere. So let's see what we can find on the system. Actually we're able to find root.txt in our working directory right from the get go. Nice!

So let's just read it and submit the flag. And we're done with another challenge.

# final thoughts

I was kind of disapointed to find the flag just sitting there in the directory. Was actually hoping for more while I got the RCE working. But hey, shouldn't complain when we succeeded with a challenge right?
