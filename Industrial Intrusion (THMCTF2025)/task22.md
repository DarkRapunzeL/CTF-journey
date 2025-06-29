# Task 22 - Rogue Poller

Category: Network
Difficulty: Beginner

# wireshark

Let's download the Task Files and see what we're dealing with. It is a package capture. So let's open it in wireshark.

```
wireshark rogue-poller-1750969333044.pcapng
```

When looking through it we see that there are pretty much three categories of packages. Modbus Query, Modbus Response and TCP ACK. And in the response packages we start seeing something that looks to be a flag. But let's just make it easy for ourselves and follow the TCP stream.

Right click a package -> Follow -> TCP Stream

And there we see our flag. So if we copy it and paste it into mousepad or some other notepad-like program. We can just go to find and replace and replace the dots with nothing to remove them. And now we have the flag ready to be pasted in to the site and submitted.
