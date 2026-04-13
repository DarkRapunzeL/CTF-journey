# Fråga Fisken
## Information
Category: web
Difficulty: easy
Attachments: source code

## Summary
A file server containing information about fishes. With access to the source code it was simple to just see that the challenge
seemed to have some kind of path traversal vulnerability. Since it tried to sanitize the path parameter to remove dots from the
begining of a string. Then it was a simple matter of figuring out where the flag was stored and read it.

## How I went through it

### reading the source

Since we were able to download the source code for the server, naturally that seems like a good place to start. It was written in
typescript. And it showed that the app answered to request on three endpoints. These were the root (/), /list-dir and /read-file.
Directly this pointed towards it being some kind of LFI/Path traversal shenanigans.

Besides the endpoints I found that the GET parameter path was being "sanitized", although calling it sanitized is kind of a stretch.
It checked the start of the value for double dots. (..) The thing is that this doesn't even make it impossible to path traverse with a
string such as (../../../), since only the first dots get filtered out.

### getting the flag

After reading the source I started using the /list-dir endpoint to go up the file system hierarchy and check the contents of those
directories. With file=/../../ I found a file named flaggfisk, which certainly looks promising. So now I just had to try to read it
with the /read-file endpoint.

```
/read-file?path=/../../flaggfisk
```

And there it was.
