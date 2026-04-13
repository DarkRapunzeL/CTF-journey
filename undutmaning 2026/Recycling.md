# Recycling
## information
Category: Web

Difficulty: Easy

Attachments: .pcap

## Summary
The password to log in to the interface had been forgotten. Some old network data had been found that might be of use. 
Looking into the .pcap with Wireshark, it was possible to find some HTTP communication. Since it was HTTP it was easy to just
look at the headers and find a session cookie. This could then be copied and pasted into the browser, working as authentication
on the app. And thereby the flag was open for grabs.

## How I went through it
I started by accessing the page that we were given to try to understand what we were dealing with. There was just a 
single login though. So not much to go off of. But we were also given a .pcap for the challenge, so I started looking through
this. As I kept looking, I found that there were some ordinary HTTP communication happening instead of encrypted HTTPS.
This made it possible to read the communication in cleartext.

I figured I'd start by looking for a package containing the word password. And found a response from the server,
serving the login page. I didn't get the password, but at least now I knew the IP address of the server within this connection.
So my next step was to filter packages by the IP 192.168.100.10, to find all communcation between the server and the client.

With this filtered list of packages. I just went through the packages and found a GET request to / with a cookie called
SECRET. Which sounded promising. I added this to my browser and tried refreshing the site. And I was in, and was able to
grab the flag.

<img width="695" height="159" alt="cookie" src="https://github.com/user-attachments/assets/2b67c5d2-d35f-4bb9-a35d-a8a64cdc7e58" />
