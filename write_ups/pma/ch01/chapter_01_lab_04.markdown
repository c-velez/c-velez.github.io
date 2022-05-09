---
layout: post
# title: "Practical Malware Analysis Chapter 1 Labs"
date: 2022-05-09 09:40 -500
categories: jekyll update
---

# Lab 1-4

## File Metadata:

**Filename**: `Lab01-04.exe`   
**MD5**: `625ac05fd47adc3c63700c3b30de79ab`   
   
## Questions:

**Q1. Upload the `Lab01-03.exe` file to [Virus Total](https://virustotal.com). Does it match any existing antivirus definitions?**  
   
Searching for the hash in [Virus Total](https://www.virustotal.com/gui/file/0fa1498340fca6c562cfa389ad3e93395f44c72fd128d7ba08579a69aaf3b126), it looks like 54 out of 67 anit-virus engines have identified this binary as malicious.
   
![Lab01-04.exe Virus Total 01](/assets/img/pma/ch01/lab04/picture_01.JPG){: width="750"}
<p align="center"><strong>Figure 1: Virus Total Detection Summary for <em>Lab01-03.exe</em></strong></p>
   
The majority of these engines characterize this binary as a `trojan`.
   
![Lab01-04.exe Virus Total Characterization](/assets/img/pma/ch01/lab04/picture_02.JPG){: width="750"}
<p align="center"><strong>Figure 2: Virus Total Characterization for <em>Lab01-03.exe</em></strong></p>

**Q2. Are there any indications that the file was packed or obfuscated? If so, what are these indicators? If the file is packed, unpack it if possible.**
   
Running `strings` against the binary, we can see several function calls that appear to be imports.
   
![Lab01-04.exe Strings Output](/assets/img/pma/ch01/lab04/picture_03.JPG){: width="750"}
<p align="center"><strong>Figure 3: Strings Output of <em>Lab01-03.exe</em></strong></p>

Additionally, there appear to be references to different file names, file paths, and a domain.
   
![Lab01-04.exe Strings Output 2](/assets/img/pma/ch01/lab04/picture_04.JPG){: width="750"}
<p align="center"><strong>Figure 4: References to Filenames, File paths, and a Domain</strong></p>
   
This is a good indication that the binary is not packed, but let's throw it in to PEiD just to be sure.
   
![Lab01-04.exe Strings Output 2](/assets/img/pma/ch01/lab04/picture_05.JPG){: width="750"}
<p align="center"><strong>Figure 5: PEiD Output</strong></p>
   
PEiD identifies the binary as being compiled with `Microsoft Visual C++ 6.0`. This binary is not packed, so we should be able to get the import table.
   
**Q3. When was the program compiled?**
   
We can find the compile time using PEView under the `IMAGE_FILE_HEADER` of the binary. According to PEView, this binary was compiled 2019/08/30. Since this book was published in 2012, it is likely that this compile time if fake.

![Lab01-04.exe PEView](/assets/img/pma/ch01/lab04/picture_09.JPG){: width="750"}
<p align="center"><strong>Figure 6: PEView Output</strong></p>

**Q4. Do any imports hint at this program's functionality? If so, which imports are they and what do they tell you?**
   
Loading the binary in to `Dependency Walker`, we see that the execuatble loads `kernel32.dll`, `advapi32.dll`, and `msvcrt.dll`.

From `kernel32.dll`, the binary loads the following:
   
![Lab01-04.exe Depends: kernel32.dll](/assets/img/pma/ch01/lab04/picture_06.JPG){: width="750"}
<p align="center"><strong>Figure 7: <em>kernel32.dll</em> Imports</strong></p>
   
From `advapi32.dll`, the binary loads the following:
   
![Lab01-04.exe Depends: advapi32.dll](/assets/img/pma/ch01/lab04/picture_07.JPG){: width="750"}
<p align="center"><strong>Figure 8: <em>advapi32.dll</em> Imports</strong></p>
   
From `msvcrt.dll`, the binary loads the following:
   
![Lab01-04.exe Depends: msvcrt.dll](/assets/img/pma/ch01/lab04/picture_08.JPG){: width="750"}
<p align="center"><strong>Figure 9: <em>msvcrt.dll</em> Imports</strong></p>
   
Looking at the imports tables, we can see that the binary is looking for a file (`FindResourceA`) within a directory. Presumably, after it finds the resource, it will write the resource to disk (`WriteFile`), attempt to load (`LoadResource`), and then execute the resource using `WinExec`. Additionally, looking at the imports from `advapi32.dll`, the binary appears to also manipulate the permissions for a file in some way.

**Q5. What host- or network-based indicators could be used to identify the malware on infected machines?**
   
Looking at the `strings` output, we could use references to `\system32\wupdmgrd.exe` and `\winup.exe` as host-based indicators. Additionally, we could use the reference to `http://www[.]practicalmalwareanalysis[.]com/updater[.]exe` as a networkd-based indicator.

**Q6. This file has one resource in the resource section. Use Resource Hacker to examine that resource, and then use it to extract the resource. What can you learn from the resource?**
   
When loading the binary in to `Resource Hacker`, we see that it finds a binary in the resources section of the original executable.
   
![Lab01-04.exe Resource Hacker](/assets/img/pma/ch01/lab04/picture_10.JPG){: width="750"}
<p align="center"><strong>Figure 10: Resource Hacker Output</strong></p>

We can use `Resource Hacker` to carve this binary out of the original executable (**Action>Save Resource to a BIN file...**) to continue analysis.
   
Looking at the imports of the new file, we see that it imports functions from `kernel32.dll`, `urlmon.dll`, and `msvcrt.dll`.
   
From `kernel32.dll`, the binary loads the following:
   
![Lab01-04_resource.exe Depends: kernel32.dll](/assets/img/pma/ch01/lab04/picture_11.JPG){: width="750"}
<p align="center"><strong>Figure 11: <em>kernel32.dll</em> Imports from the New Binary</strong></p>
   
From `urlmon.dll`, the binary loads the following:
   
![Lab01-04_resource.exe Depends: urlmon.dll](/assets/img/pma/ch01/lab04/picture_12.JPG){: width="750"}
<p align="center"><strong>Figure 12: <em>urlmon.dll</em> Imports from the New Binary</strong></p>

From `msvcrt.dll`, the binary loads the following:
   
![Lab01-04_resrouce.exe Depends: msvcrt.dll](/assets/img/pma/ch01/lab04/picture_13.JPG){: width="750"}
<p align="center"><strong>Figure 13: <em>msvcrt.dll</em> Imports from the New Binary</strong></p>

Using the import table and the strings output, we can infer a few things about this malware.

1. The original binary was a shell that would unpack the downloader and load it. This can be inferred because of the references to `FindResourceA` in the original binary, that was not in the resource that we carved out.
2. The resource that was carved out attempts to download a file to disk from `http://www[.]practicalmalwareanalysis[.]com/updater[.]exe` and likely saves it to `\system32\wupdmgrd.exe`.
3. The resource also attempts to execute the downloaded file. This can be inferred because of the reference to `WinExec` in the import table.


---
<p align="center"><a href="/write_ups/pma/ch01/chapter_01_directory">Chapter 1 Labs Directory</a> :: <a href="/practical-malware-analysis/">Practical Malware Analysis Labs Directory</a></p>