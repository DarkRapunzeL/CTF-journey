# Task 19 - No salt, No shame

Category: Crypto
Difficulty: Beginner

# openssl

I downloaded the file for the challenge. But as I've not worked much with crypto before I wasn't really sure how to decrypt it. After some googling I found out that openssl should be able to decrypt it. So I tried this with

```
openssl aes-256-cbc -d -nosalt -in shutdown.log-1750934543756.enc
```

And got a string that seemed to be in the format of the flag just missing the start of it. So I took it, added the missing prefix and pasted it into the site and tried submitting. And it worked.
