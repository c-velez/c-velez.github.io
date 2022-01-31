---
layout: post
# title: "Practical Malware Analysis Chapter 1 Labs"
date: 2022-01-28 10:41 -500
categories: jekyll update
---

# Practical Malware Analysis Chapter 1

This chapter discusses techniques for basic static malware analysis. The binary structure of PE files can give a lot of information. This includes import and export tables, strings, resources used by the binary, and any obfuscation that may be used to prevent analysis. Often times, basic static analysis allows analysts to have a jumping off point for deeper analysis.

---

## Lab 1-1:

### File Metadata:

**Filename**: `Lab01-01.exe`   
**MD5**: `bb7425b82141a1c0f7d60e5106676bb1`   
   
**Filename**: `Lab01-01.dll`   
**MD5**: `290934c61de9176ad682ffdd65f0a669`   

### Questions:

1. Upload `Lab01-01.exe` and `Lab01-01.dll` to [Virus Total](https://www.virustotal.com) and view the reports. Does either file match any existing anti virus signatures?

Virustotal is a sandbox website that allows users to upload potentially malicious files and urls as a way to get an initial understanding of the file. They use several anti-virus engines to characterize whether or not the file is malicious and to help determine what the file may do. Unfortunately, if the file that is being analyzed has never been uploaded before or there are no AV signatures created for it, it may not show up as malicious in Virus Total. As such, it is always good to do a more comprehensive analysis to determine the nature of the executable.

Searching for the file hashes in Virus Total, we can note a few things:

For [`Lab01-01.exe`](https://www.virustotal.com/gui/file/58898bd42c5bd3bf9b1389f0eee5b39cd59180e8370eb9ea838a0b327bd6fe47/detection):

- Virustotal indicates that 48 anti-virus engines flag this binary as malicious
- Virustotal recognizes this hash to be associated with `Lab01-01.exe`
![Lab01-01.exe Virus Total 01](/assets/img/pma/ch01/picture_01.jpg){: width="750"}
<p align="center"><strong>Figure 1: Virus Total Detection for Lab01-01.exe</strong></p>
- The majority of the anti-virus engines that recognize this executable characterize it as a `Trojan`
![Lab01-01.exe Virus Total 02](/assets/img/pma/ch01/picture_02.jpg){: width="750"}
<p align="center"><strong>Figure 2: Virus Total AV Engine Characterization for Lab01-01.exe</strong></p>

For [`Lab01-01.dll`](https://www.virustotal.com/gui/file/f50e42c8dfaab649bde0398867e930b86c2a599e8db83b8260393082268f2dba/detection):

- Virustotal indicates that 39 anti-virus engines flag this binary as malicious
- Virustotal recognizes this hash to be associated with `Lab01-01.dll`
![Lab01-01.dll Virus Total 01](/assets/img/pma/ch01/picture_03.jpg){: width="750"}
<p align="center"><strong>Figure 2: Virus Total AV Engine Characterization for Lab01-01.exe</strong></p>
- 

2. When were the files compiled

3. Are there any indications that either of these files is packed or obfuscated? If so, what are these indicators?

4. Do any imports hint at what this malware does? If so, which imports are they?

5. Are there any other files or host based indicators that you could look for on infected systems?

6. What network-based indicators could be used to find this malware on infected machines?

7. What would you guess is the purpose of these files?

---

## Lab 1-2:

### File Metadata:

**Filename**: `Lab01-02.exe`   
**MD5**: `8363436878404da0ae3e46991e355b83`   

### Questions:

1. Upload the `Lab01-02.exe` file to [Virus Total](https://www.virustotal.com). Does it match any existing antivirus definitions?

2. Are there any indications that this file is packed or abfuscated? If so, what are these indicators? If the file is packed, unpack it if possible.

3. Do any imports hint at this program's functionality? If so, which imports are they and what do they tell you?

4. What host or network-based indicators could be used to identify this malware on infected machines?

---

## Lab 1-3:

### File Metadata:

**Filename**: `Lab01-03.exe`   
**MD5**: `9c5c27494c28ed0b14853b346b113145`   

### Questions:

1. Upload the file to [Virus Total](https://virustotal.com). Does it match any existing antivirus definitions?

2. Are there any indications that this file is packed or obfuscated? If so, what are these indicators? If the file is packed, unpack it if possible.

3. Do any imports hint at this program's functionality? If so, which imports are they and what do they tell you?

4. What host or network-based indicators could be used to identify this malware on infected machiens?

---

## Lab 1-4

### File Metadata:

**Filename**: `Lab01-04.exe`   
**MD5**: `624ac05fd47adc3c63700c3b30de79ab`   

### Questions:

1. Upload the file to [Virus Total](https://virustotal.com). Does it match any existing antivirus definitions?

2. Are there any indications that this file is packed or obfuscated? If so, what are these indicators? If the file is packed, unpack it if possible.

3. When was this program compiled?

4. Do any imports hint at this program's functionality? If so, which imports are they and what do they tell you?

5. What host or network-based indicators could be used to identify this malware or infected machines?

6. This file has one resource in the resource section. Use Resource Hacker to examine that resource, and then use it to extract the resource. What can you learn from the resource?

---
