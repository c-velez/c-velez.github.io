---
layout: post
# title: "Grepping for Gold"
date: 2022-01-07 07:57 -500
categories: jekyll update
---

# Grepping for Gold

**Objective**: Help Greasy GopherGuts parse nmap output  
**Location**: Outside Jack Frost Tower  
**Hints**:
1. Grep Cheat Sheet: Check [this](https://ryanstutorials.net/linuxtutorial/cheatsheetgrep.php) out if you need a grep refresher.

## Write Up

It looks like Greasy GopherGuts scanned a large network and needs help parsing the output with grep. Lets click on the terminal and see what we have to work with.

![Initial Terminal Output](/assets/img/2021_sans_hhc/term/grepping_for_gold/picture_2.png){: width="750"}
<p align="center"><strong>Figure 1: Initial Terminal Output</strong></p>

There are series of questions that we need to aswer in order to finish the challenge.

The first 2 questions are relatively straight forward:  

**Question 1**: What port does 34.76.1.22 have open?  
**Command**: `grep 34.76.1.22 bigscan.gnmap`  
**Answer**: 62078  

**Question 2**: What port does 34.77.207.226 have open?  
**Command**: `grep 34.77.207.226 bigscan.gnmap`  
**Answer**: 8080  

Question 3 is going to require some grep/cut/sort/uniq magic:  

**Question 3**: How many hosts appear "Up" in the scan?  
**Command**: `cat bigscan.gnmap | grep Up | cut -d' ' -f2 | sort -n | uniq | wc -l`  
**Answer**: 26054  

Using the hints in question 4, we can build our command:  

**Question 4**: How many hosts have a web port open?  
**Command**: `grep -E "(80/open|443/open|8080/open" bigscan.gnmap | wc -l`  
**Answer**: 14372  

Question 5 is a bit tricky. Thinking a bit outside of the box, we can get the answer indirectly but subtracting the number of hosts that are up from the number of hosts that were scanned. We can do the math in the terminal too!  

**Question 5**: How many hosts with status Up have no (detected) open TCP ports  
**Command**: ```echo $((`grep Up bigscan.gnmap | wc -l` - `grep open bigscan.gnmap | wc -l`))```   
**Answer**: 402   

Finally, the last question! Let's use some regex to figure this one out. We know that we are looking for the number of times the string "open" occurs in a line since this is how nmap designates when ports are open. So our regex input will look something like `(*/open.*){<some digit>,}`. Let's start with 10. If we get output, we know we need to try a high number, if we don't get output, we have to go lower. We notice that there are several hosts that have 10 or more ports open, so let's see if there are any hosts that have 15 ports open. It doesn't look like there are any hosts that have 15 or more ports open, se we know we have to go lower. Using a binary search method, let's half the search space between 10 and 15, and try 13. Since no hosts have 13 or more ports open, let's try 12. And there we go!   

**Question 6**: Whats the greatest number of TCP ports any one host has open?   
**Command**: `grep -E "(*/open.*){12,}" bigscan.gnmap`   
**Answer**: 12   

![Success](/assets/img/2021_sans_hhc/term/grepping_for_gold/picture_3.png){: width="750"}
<p align="center"><strong>Figure 2: Success!</strong></p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-3">< Objective 3</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>