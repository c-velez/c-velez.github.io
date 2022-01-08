---
layout: post
# title: "Strange USB Device"
date: 2022-01-06 21:37 -500
categories: jekyll update
---

# Strange USB Device

**Objective**: One of the Elves found a strange USB device and they need our help to figure out what it does  
**Location**: Kringle Castle - 2nd Floor Speaker UNPreparation Room  
**Hint Terminal**: [Jewel Loggins](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-IPv6-Sandbox)  
**Hints**:
1. Ducky Script - [Ducky Script](https://docs.hak5.org/hc/en-us/articles/360010555153-Ducky-Script-the-USB-Rubber-Ducky-language) is the language for the USB Rubber Ducky
2. Duck Encoder - Attackers can encode Ducky Script using a [duck encoder](https://docs.hak5.org/hc/en-us/articles/360010471234-Writing-your-first-USB-Rubber-Ducky-Payload)
3. Mitre ATT&CK and Ducky - The [MITRE ATT&CK tactice T1098.004](https://attack.mitre.org/techniques/T1098/004/) describes SSH persistence techniques thought authorized keys files.
4. Ducky RE with Mallard - It's also possible to reverse engineer encoded Ducky Script using [Mallard](https://github.com/dagonis/Mallard)

## Write Up

Let's pop open the terminal and see what we have to work with

![Initial Terminal Output](/assets/img/2021_sans_hhc/obj/obj05/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: Initial Terminal Output</strong></p>

It looks like we'll have to evaluate a USB device and answer a series of flags for this objective!

Taking a look at `/mnt/USBDEVICE` we see there is a file there called inject.bin. Let's go ahead and copy that file to our home directory where mallard.py lives.

```
cp /mnt/USBDEVICE/inject.bin ~/
```

![Copy File](/assets/img/2021_sans_hhc/obj/obj05/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: Copy the File to the Home Directory</strong></p>

Let's run `mallard.py` against the `inject.bin` to see what the decoded output looks like.

```
$ ./mallard.py --file inject.bin
```

It looks like it worked! We see the routine that the inject.bin file contains. It looks like someone was enumerating the user and attempting to call back to `trollfun.jackfrosttower.com/1337`. There's also a base64 encoded string with a series of commands that were used to generate it.

![Mallard Output](/assets/img/2021_sans_hhc/obj/obj05/picture_3.png){: width="750"}
<p align="center"><strong>Figure 3: Mallard Decoded Output</strong></p>

Let's do some decoding. We see that the string was encoded, and then reversed. Looking at the actual command, we should be able to copy and paste it in to the terminal and get the decoded output

![Decode Suspect String](/assets/img/2021_sans_hhc/obj/obj05/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: Decode the Suspicous String!</strong></p>

It looks like we found the culprit!

![The Culprit](/assets/img/2021_sans_hhc/obj/obj05/picture_4.png){: width="750"}
<p align="center"><strong>Figure 5: The Culprit</strong></p>

Answer: ||Icky McGoop||

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-4">< Objective 4</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-6">Objective 6 ></a></p>