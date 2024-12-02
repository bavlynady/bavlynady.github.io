---
title: "ConvertMyVideo CTF Write-up"
date: 2023-10-07
image: "/images/writeups/convertmyvideo.jpg"
description: "A comprehensive write-up on the ConvertMyVideo CTF challenge."
draft: false
type: "tryhackme"
---

<!-- Write-up content here -->


**Welcome, Hackers!**

Today, we'll be solving the "ConvertMyVideo" Capture The Flag (CTF) challenge. This medium-difficulty challenge requires a solid understanding of common web attacks and privilege escalation techniques. Let's dive in!

## Initial Reconnaissance

First, we perform an Nmap scan to identify open ports on the target machine.

**Image 1: Nmap Scan Results**

![Nmap Scan](1-nmap-scan.png)

The scan reveals that ports 22 (SSH) and 80 (HTTP) are open. Since we don't have any valid SSH credentials, we'll focus on exploring the website hosted on port 80.

## Exploring the Website

**Image 2: Website Homepage**

![Website Homepage](2-website.png)

The website appears to accept YouTube links for video conversion. We test it by submitting a simple input.

**Image 3: Input Test**

![Input Test](3-input-test.png)

On the surface, nothing interesting happens. To investigate further, we inspect the HTTP responses using Burp Suite's repeater.

**Image 4: Burp Suite Repeater Response**

![Burp Suite Response](4-repeater-response.png)

We notice that the server returns an error, indicating potential vulnerabilities. Given that the site processes URLs, we consider testing for Server-Side Request Forgery (SSRF) and command injection attacks.

## Testing for SSRF and Command Injection

We set up a simple HTTP server and observe that the target server makes a GET request to our server when we input our server's address.

**Image 5: GET Request Captured**

![GET Request](10-get.png)

However, we can't exploit this behavior further through SSRF. Next, we attempt command injection by manipulating the input to execute system commands.

By crafting inputs that include command execution syntax, we discover that the server processes certain characters that allow command execution. This opens the door to potentially gaining a shell on the server.

## Bypassing Input Restrictions

We encounter an issue where the server doesn't handle spaces in our commands. To bypass this restriction, we use alternative methods to represent spaces in the command syntax.

**Image 6: Bypassing Space Restrictions**

![Bypassing Spaces](14-IFS.png)

Using this technique, we can execute commands without spaces, allowing us to download a reverse shell script to the target server.

**Image 7: Downloading the Reverse Shell Script**

![Downloading Script](12-wget.png)

## Establishing a Reverse Shell

After transferring the shell script to the server, we adjust its permissions and execute it to establish a reverse shell connection back to our machine.

**Image 8: Executing the Shell Script**

![Executing Shell](13-shell.png)

With the shell connection established, we can interact with the server at a deeper level.

## Privilege Escalation

While exploring the server, we find two noteworthy files: `clean.sh` and `.htpasswd`. The `.htpasswd` file contains hashed credentials for a user named `itsmeadmin`.

We extract the hash and attempt to crack it using a tool like Hashcat. By identifying the correct hash type and mode, we successfully retrieve the plaintext password.

**Image 9: Cracking the Password**

![Cracking Password](19-cracking.png)

However, logging in via SSH with these credentials doesn't grant us elevated privileges. We need to find another way.

## Exploiting Cron Jobs

We check the cron jobs running on the server and notice that the `cron` daemon is executing the `clean.sh` script periodically.

To monitor the cron activities, we use a tool called `pspy`, which allows us to see processes without requiring elevated privileges.

**Image 10: Monitoring Cron Jobs with pspy**

![pspy Output](21-pspy.png)

Since we have write permissions for `clean.sh`, we modify it to include a command that initiates a reverse shell as the root user.

After setting up a listener on our machine, we wait for the cron job to execute our modified script.

**Image 11: Receiving the Root Shell**

![Root Shell](17-shell.png)

Once the cron job runs, we receive a shell with root privileges, successfully completing the challenge.

## Conclusion

We navigated through initial reconnaissance, exploited command injection vulnerabilities, bypassed input restrictions, and leveraged cron jobs for privilege escalation to root the server.

I hope you enjoyed this write-up and found it informative. Stay tuned for more challenges!

**Author:** Bavly Nady
