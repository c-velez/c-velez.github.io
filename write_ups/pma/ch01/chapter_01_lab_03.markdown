---
layout: post
# title: "Practical Malware Analysis Chapter 1 Labs"
date: 2022-05-05 08:37 -500
categories: jekyll update
---

# Lab 1-3

## File Metadata:

**Filename**: `Lab01-03.exe`   
**MD5**: `9c5c27494c28ed0b14853b346b113145`   
   
## Questions:

**Q1. Upload the `Lab01-03.exe` file to [Virus Total](https://virustotal.com). Does it match any existing antivirus definitions?**   
   
Searching for the [hash](https://www.virustotal.com/gui/file/7983a582939924c70e3da2da80fd3352ebc90de7b8c4c427d484ff4f050f0aec), we can see that 58 out of 68 anti-virus engines flagged this binary as malicious.
   
![Lab01-03.exe Virus Total 01](/assets/img/pma/ch01/lab03/picture_01.JPG){: width="750"}
<p align="center"><strong>Figure 1: Virus Total Detection Summary for <em>Lab01-03.exe</em></strong></p>
   
The majority of the anti-virus engines flagged the binary as a `trojan`.
   
![Lab01-03.exe Virus Total Characterization](/assets/img/pma/ch01/lab03/picture_02.JPG){: width="750"}
<p align="center"><strong>Figure 2: Virus Total Characterization for <em>Lab01-03.exe</em></strong></p>
   
**Q2. Are there any indications that the file was packed or obfuscated? If so, what are these indicators? If the file is packed, unpack it if possible.**   
   
Running strings on the file, it is observed the the listing is small, with references to `LoadLibraryA` and `GetProcAddress`. These function calls are known to be seen with packed binaries since the program need to reference these functions in order to unpack itself
   
![Lab01-03.exe Strings Output](/assets/img/pma/ch01/lab03/picture_03.JPG){: width="750"}
<p align="center"><strong>Figure 3: Strings Output of <em>Lab01-03.exe</em></strong></p>
   
We can also check to see what the import table looks like for this binary. Loading this binary into `Dependency Walker`, we can see a very small import table, which doesn't make sense for a program in general. Additionally, the output for dependency walker validates our findings in the `strings` out of the file where `LoadLibraryA` and `GetProcAddress` were found.
   
![Lab01-03.exe Dependency Walker Output](/assets/img/pma/ch01/lab03/picture_05.JPG){: width="750"}
<p align="center"><strong>Figure 4: Dependency Walker Output of <em>Lab01-03.exe</em></strong></p>
   
Loading the binary in to PEiD, the program recognizes that packer as FSG 1.0. Unpacking this binary is covered later in the book.
   
![Lab01-03.exe PEiD Output](/assets/img/pma/ch01/lab03/picture_04.JPG){: width="750"}
<p align="center"><strong>Figure 5: PEiD Output of <em>Lab01-03.exe</em></strong></p>
   
Since we are unable to proceed without addtional knowledge, this ends our analysis of this binary.


---
<p align="center"><a href="/write_ups/pma/ch01/chapter_01_directory">Chapter 1 Labs Directory</a> :: <a href="/practical-malware-analysis/">Practical Malware Analysis Labs Directory</a></p>