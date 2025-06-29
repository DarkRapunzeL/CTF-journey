# Task 25 - Access Granted

Category: Reversing
Difficulty: Easy

# strings

We download the files and unzip it. When we have it unzipped let's see what clear text strings we can find in the executable.

```
strings access_granted
```

We find the area where the strings for the password check is. A bit below that there is a string with a word that seems like it doesn't really belong. So that might actually be it.

Let's try it.

```
nc 10.10.25.145 9009
```

It was correct! We have our flag.
