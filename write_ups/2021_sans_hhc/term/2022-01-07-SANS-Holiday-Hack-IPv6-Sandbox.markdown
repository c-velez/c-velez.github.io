---
layout: post
# title: "IPv6 Sandbox"
date: 2022-01-07 07:57 -500
categories: jekyll update
---

# IPv6 Sandbox

**Objective**: Jewel Loggins needs help mapping out an OPv6 network  
**Location**: Kringle Castle - 2nd Floor  
**Hints**: 
1. IPv6 Reference: Check out this [Github Gist](https://gist.github.com/chriselgee/c1c69756e527f649d0a95b6f20337c2f) with common tools used in an IPv6 context.

## Write Up

When we open up the terminal, we see what flag we need to find: "Enter the correct phrase to engage the candy striper".  
Using the hint, we know that we can use ping6 to find the link local addresses for hosts in our network segment.  

```
ping6 ff02::1 -c2
```

![Ping Your Neighbors](/assets/img/2021_sans_hhc/term/ipv6_sandbox/picture_2.png){: width="750"}
<p align="center"><strong>Figure 1: IPv6 Ping Sweep</strong></p>

It looks like we get responses from 3 unique addresses.  

Let's try scanning one of them.

```
nmap -6 fe80::42:c0ff:fea8:a002%eth0
```

![Fingerprint the Host](/assets/img/2021_sans_hhc/term/ipv6_sandbox/picture_3.png){: width="750"}
<p align="center"><strong>Figure 2: Scanning <em>fe80::42:c0ff:fea8:a002</em></strong></p>

It looks like we got some results. Let's use curl to interrogate the http port on this machine.

```
curl http://[fe80::42:c0ff:fea8:a002]:80/ --interface eth0
```

![cURL the Host](/assets/img/2021_sans_hhc/term/ipv6_sandbox/picture_4.png){: width="750"}
<p align="center"><strong>Figure 3: Interrogating Port 80 on the Enumerated Host</strong></p>

It looks like we are getting closer! Let's interrogate the other port that was enumerated to see if we can get the activation phrase for the candy striper!

![Follow the Instructions](/assets/img/2021_sans_hhc/term/ipv6_sandbox/picture_5.png){: width="750"}
<p align="center"><strong>Figure 4: Interrogating the Other Port to get the Flag</strong></p>

It looks like all the Candy Striper wants it "PieceOnEarth"! Let's give it that!

![Candy Striper Activated](/assets/img/2021_sans_hhc/term/ipv6_sandbox/picture_6.png){: width="750"}
<p align="center"><strong>Figure 5: Candy Striper Activated</strong></p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-5">< Objective 5</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>