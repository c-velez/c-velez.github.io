---
layout: post
# title: "strace ltrace retrace"
date: 2022-01-07 21:52 -500
categories: jekyll update
---

# strace ltrace retrace

**Objective**: The candy machine is broken! Someone reloaded the software, but it can't find the registration file. We'll have to recreate it using strace and ltrace  
**Location**: Kringle Castle Kitchen  
**Hints**:   
- None

## Write Up

strace and ltrace are tools that trace the execution of a program. It can allow developers to see where their programs are failing. In this context, we can use strace and ltrace to see how and why the candy machine software is failing.  

We'll start by running the program to see the output.

![Execution Failed](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: The Candy Maker Can't Find the Config</strong></p>

Running the program confirms that it is looking for a configuration file, but it can't find one.  

Running `ltrace ./make_the_candy`, we see that the program is looking for a `registration.json` file to read. So, let's give it what it wants. We'll make a `registration.json` file and fill it with garbage.

![Trying to Open `registration.json`](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: Trying to Open <em>registration.json</em></strong></p>

![Create the File it is Looking For](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_3.PNG){: width="750"}
<p align="center"><strong>Figure 3: Create the File the Program is Looking For</strong></p>

Running the program after creating the registration file, we see that the program is looking for the term `Registration` inside of the file. We know this because of the call to the `strstr()` function. This function takes 2 arguments, the first argument is the string that is found, the second argument is the string it is looking for. The function returns a pointer to the beginning of the substring (the second argument) if it is found. If it doesn't find the substring, it returns `NULL`. 

![Registration File Contents](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: The Program is Looking for the String <em>Registration</em></strong></p>

Let's edit our registration file to make the word `Registration` the key, for the key, value pair.

![Edit the Key](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_5.PNG){: width="750"}
<p align="center"><strong>Figure 5: Updated Contents of <em>registration.json</em></strong></p>

Running the program with the editted configuration file still yields an unregistered error. Let's run `ltrace` against the program again and take a look at the output.

![What Value?](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_6.png){: width="750"}
<p align="center"><strong>Figure 6: <em>ltrace</em> Output After Editing the File Contents</strong></p>

The program uses the `strstr()` function again to look for the value `True`, after `Registration` within the file. Editing the `registration.json` with a key:value pair of `Registration:True`, should yield us good results.

![New File Contents](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_7.png){: width="750"}
<p align="center"><strong>Figure 7: Updated Key:Value Pair in the Registration File</strong></p>

And it looks like our execution was successful!

![New File Contents](/assets/img/2021_sans_hhc/term/strace_ltrace_retrace/picture_8.png){: width="750"}
<p align="center"><strong>Figure 8: We've Created the Correct Registration File</strong></p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-11">< Objective 11</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>