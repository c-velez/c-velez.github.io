---
layout: post
# title: "Customer Complaint Analysis"
date: 2022-01-07 21:42 -500
categories: jekyll update
---

# Customer Complaint Analysis

**Objective**: A human has left a review of the Jack Frost Tower Hotel. The problem is, the reviews are only supposed to be for the trolls to complain about the patrons! Can we figure out which trolls are complaining about the human that left the review?  
**Location**: Jack Frost Tower Talks Lobby  
**Hint Terminal**: [Tinsel Upatree](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-Strace-Ltrace-Retrace)    
**Hints**:
1. Evil Bit RFC - [RFC3514](https://datatracker.ietf.org/doc/html/rfc3514) defines the usage of the "Evil Bit" in IPv4 headers.
2. Wireshark Display Filters - Different from BPF capture filters, Wireshark's [display filters](https://wiki.wireshark.org/DisplayFilters) can find text with the `contains` keyword - and evil bits with `ip.flag.rb`.
3. Talking to the Pat Tronizer, we know all the valid hosts on their network should be RFC3514 compliant. That is, the evil bit is on.

## Write Up

Clicking on the objective in the badge, we are given a packet capture file to analyze. Fire up [Wireshark](https://www.wireshark.org/), and let's get analyzing.  

We know that all valid hosts on the network are RFC3514 compliant, so let's look for the host that isn't RFC compliant.

```
ip.flags.rb == 0 && http
```

!["Evil" Host](/assets/img/2021_sans_hhc/obj/obj11/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: The Host That Does Not Belong</strong></p>

It looks like the host `10.70.84.251` is the host that does not belong. Looking at the encoded form that was submitted, it appears that the human is in room 1024.

![Poor Human...](/assets/img/2021_sans_hhc/obj/obj11/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: The Poor Lady Staying at this Hotel...</strong></p>

Let's take a look at some of the other forms to see if there are any places we can use a key word.

![Room Numbers](/assets/img/2021_sans_hhc/obj/obj11/picture_3.png){: width="750"}
<p align="center"><strong>Figure 3: The Trolls Use the Room Numbers as Client Identifiers</strong></p>

Let's develop a query to look for url encoded values that contain "1024".
```
urlencoded-form.value contains "1024"
```

![Annoyed Trolls](/assets/img/2021_sans_hhc/obj/obj11/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: The Trolls That Commented on the Pesky Human</strong></p>

Looking through the posted forms, we can see that Flud, Hagg, and Yaqh were the three trolls that complained about the guest in room 1024... they are not happy with her...

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-10">< Objective 10</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>