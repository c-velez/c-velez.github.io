---
layout: post
# title: "Practical Malware Analysis Chapter 1 Labs"
date: 2022-01-28 10:41 -500
categories: jekyll update
---

# Practical Malware Analysis Ch01

This chapter discusses techniques for basic static malware analysis. The binary structure of PE files can give a lot of information. This includes import and export tables, strings, resources used by the binary, and any obfuscation that may be used to prevent analysis. Often times, basic static analysis allows analysts to have a jumping off point for deeper analysis.

---

## Lab 1-1:

### File Metadata:

**Filename**: `Lab01-01.exe`   
**MD5**: `bb7425b82141a1c0f7d60e5106676bb1`   
   
**Filename**: `Lab01-01.dll`   
**MD5**: `290934c61de9176ad682ffdd65f0a669`   
   
### Questions:

**Q1. Upload `Lab01-01.exe` and `Lab01-01.dll` to [Virus Total](https://www.virustotal.com) and view the reports. Does either file match any existing anti virus signatures?**     

Virustotal is a sandbox website that allows users to upload potentially malicious files and urls as a way to get an initial understanding of the file. They use several anti-virus engines to characterize whether or not the file is malicious and to help determine what the file may do. Unfortunately, if the file that is being analyzed has never been uploaded before or there are no AV signatures created for it, it may not show up as malicious in Virus Total. As such, it is always good to do a more comprehensive analysis to determine the nature of the executable.   
   
Searching for the file hashes in Virus Total, we can note a few things:   
   
For [`Lab01-01.exe`](https://www.virustotal.com/gui/file/58898bd42c5bd3bf9b1389f0eee5b39cd59180e8370eb9ea838a0b327bd6fe47/detection):

- Virustotal indicates that 48 anti-virus engines flag this binary as malicious
- Virustotal recognizes this hash to be associated with `Lab01-01.exe`
![Lab01-01.exe Virus Total 01](/assets/img/pma/ch01/picture_01.JPG){: width="750"}
<p align="center"><strong>Figure 1: Virus Total Detection Summary for <em>Lab01-01.exe</em></strong></p>
- The majority of the anti-virus engines that recognize this executable characterize it as a `Trojan`
![Lab01-01.exe Virus Total 02](/assets/img/pma/ch01/picture_02.JPG){: width="750"}
<p align="center"><strong>Figure 2: Virus Total AV Engine Characterization for <em>Lab01-01.exe</em></strong></p>

For [`Lab01-01.dll`](https://www.virustotal.com/gui/file/f50e42c8dfaab649bde0398867e930b86c2a599e8db83b8260393082268f2dba/detection):

- Virustotal indicates that 41 anti-virus engines flag this binary as malicious
- Virustotal recognizes this hash to be associated with `Lab01-01.dll`
![Lab01-01.dll Virus Total 01](/assets/img/pma/ch01/picture_03.JPG){: width="750"}
<p align="center"><strong>Figure 3: Virus Total Detection Summary for <em>Lab01-01.dll</em></strong></p>
- The majority of the anti-virus engines also characterize this dll as a `Trojan`
![Lab01-01.dll Virus Total 02](/assets/img/pma/ch01/picture_04.JPG){: width="750"}
<p align="center"><strong>Figure 4: Virus Total AV Engine Characterization for <em>Lab01-01.dll</em></strong></p>
   
**Q2. When were the files compiled?**     
   
Compilation time can be a bit tricky to understand as this can be obfuscated or changed at will during compile time. Additionally, certain compilers always display the same compilation time (ie. Delphi alway displays a compilation time of June 19, 1992). Although the correct time isn't displayed for compilers that use a static date, this can be used as a hint as to what the malware author used to compile their malicious code.    
    
We can use PEview's `Image File Header` to understand the compile time for each of these binaries:    
   
**File**: `Lab01-01.exe`   
**Compile Time**: 2010/12/19 Sun 16:16:19 UTC
![Lab01-01.exe Compile Time](/assets/img/pma/ch01/picture_05.JPG){: width="750"}
<p align="center"><strong>Figure 5: Compile Time for <em>Lab01-01.exe</em> using PEview</strong></p>
   
**File**: `Lab01-01.dll`   
**Compile Time**: 2010/12/16 Sun 16:16:38 UTC   
![Lab01-01.dll Compile Time](/assets/img/pma/ch01/picture_06.JPG){: width="750"}
<p align="center"><strong>Figure 6: Compile Time for <em>Lab01-01.dll</em> using PEview</strong></p>
   
**Q3. Are there any indications that either of these files is packed or obfuscated? If so, what are these indicators?**    

There are several ways to determine if a file is packed or obfuscated.
- **Strings**: If there are few or no human readable strings in the binary, that could also be an indicator that the file is packed. 
- **PEiD**: Loading the file in PEiD, we can check to see if PEiD recognizes or known compiler or packer
- **Imports**: Looking at the size of the import table in addition to what is being imported. A small import table could be indicative of packing.
- **Header Information**: Looking at the `.text` header information and comparing the `Virtual Size` and `Raw Size` of the file could indicate that a file is packed. If the file is not packed, these to values should be close to equal; if the `Virtual Size` is much larger than the `Raw Size`, this could indicate the the file is packed or obfuscated.   
   
We will walk through all of the above methods for both files in this lab.   
   
**File**: `Lab01-01.exe`
**Packed**: No
**Analysis**:
- PEiD: Loading the file in to PEiD, the program recognizes that `Lab01-01.exe` was compiled using `Microsoft Visual C++ 6.0`
![Lab01-01.exe Packed Analysis PEiD](/assets/img/pma/ch01/picture_07.JPG){: width="750"}
<p align="center"><strong>Figure 7: PEiD Output for <em>Lab01-01.exe</em></strong></p>
- Strings: We can also see in the strings output that there are several readable strings, including file paths and messages, in this executable
![Lab01-01.exe Strings Analysis](/assets/img/pma/ch01/picture_08.JPG){: width="750"}
<p align="center"><strong>Figure 8: Truncated Strings Output for <em>Lab01-01.exe</em></strong></p>
- Imports: Using dependency walker, we can take a look at the import table for this file. It should be noted that the strings output above also shows what the file imports.
![Lab01-01.exe Packed Analysis DepWalker 01](/assets/img/pma/ch01/picture_09.JPG){: width="750"}
<p align="center"><strong>Figure 9: Dependency Walker Output for <em>Lab01-01.exe</em> - <em>kernel32.dll</em></strong></p>
![Lab01-01.exe Packed Analysis DepWalker 02](/assets/img/pma/ch01/picture_10.JPG){: width="750"}
<p align="center"><strong>Figure 10: Dependency Walker Output for <em>Lab01-01.exe</em> - <em>msvcrt.dll</em></strong></p>
- Header Information: Pulling the file up in PEview again, we can talk a look at the `.text` section header information to view the reported size on disk (`Raw Size`) and the reported size in memory (`Virtual Size`). The `Virtual Size` (0x0970) and `Raw Size` (0x1000) are mostly the same size. The differ by 0x0690 bytes. Additionally, the `Virtual Size` is not larger than the `Raw Size`, indicating that the file is not packed.
   
Using the above information, we can determine that the file is not packed. The most compelling pieces of evidence being that PEiD was able to recognize a known compiler for the file, as well as the comparison of the size on disk and size in memory.   
   
**File**: `Lab01-01.dll`   
**Packed**: No   
**Analysis**:
- PEid: Loading the file in to PEiD, the program recognizes that `Lab01-01.dll` was compiled using `Microsoft Visual C++ 6.0`
![Lab01-01.dll Packed Analysis PEiD](/assets/img/pma/ch01/picture_11.JPG){: width="750"}
<p align="center"><strong>Figure 11: PEiD Output for <em>Lab01-01.dll</em></strong></p>
- Strings: We can see the strings produces a readable output, including was appears to be a hardcoded IP address: `127.26.152.13`
![Lab01-01.dll Strings Analysis](/assets/img/pma/ch01/picture_12.JPG){: width="750"}
<p align="center"><strong>Figure 12: Strings Output for <em>Lab01-01.dll</em></strong></p>
- Imports: Of note, Figure 14 below only shows the ordinals of the functions that were called. We can find out what the binary actually imports by looking at the table directly below the import table.   
![Lab01-01.dll DepWalker 01](/assets/img/pma/ch01/picture_13.JPG){: width="750"}
<p align="center"><strong>Figure 13: Dependency Walker Output for <em>Lab01-01.dll</em> - <em>kernel32.dll</em></strong></p>
![Lab01-01.dll DepWalker 02](/assets/img/pma/ch01/picture_14.JPG){: width="750"}
<p align="center"><strong>Figure 14: Dependency Walker Output for <em>Lab01-01.dll</em> - <em>ws2_32.dll.dll</em></strong></p>
![Lab01-01.dll DepWalker 03](/assets/img/pma/ch01/picture_15.JPG){: width="750"}
<p align="center"><strong>Figure 15: Dependency Walker Output for <em>Lab01-01.dll</em> - <em>msvcrt.dll.dll</em></strong></p>
- Header Information: Pulling up the file in PEview, we can look at the `.text` section header. The `Virtual Size` (0x039E) and `Raw Size` (0x1000) differ quite a bit, but the `Virtual Size` is not larger than the `Raw Size`, indicating that this file is not packed.
   
The above analysis is contains enough information to determine that the `Lab01-01.dll` file is not packed. The most compelling pieces of evidence being that PEiD was able to recognize a known compiler, and the comparison of the size on disk and the size in memory.   
   
**Q4. Do any imports hint at what this malware does? If so, which imports are they?**   

When analyzing imports, especially for malware that targets the `Windows` operating system, the [Microsoft Developer Network](https://docs.microsoft.com/en-us/) documentation is **critical** in understanding what each function does. For example, doing a quick Google Search using the keywords `CreateProcessA MSDN`, yields the Microsoft Documentation for the `CreateProcessFunction` [function](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa), including the structure, and how it is used. Knowing this, we can use the import tables that we enumerated in the previous question, to help us tie together what the malware may do.   
   
**File**: `Lab01-01.exe`    
**Imports**:
- `kernel32.dll`
	- [`CloseHandle`](https://docs.microsoft.com/en-us/windows/win32/api/handleapi/nf-handleapi-closehandle)
	- [`CopyFileA`](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-copyfilea)
	- [`CreateFileA`](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea)
	- [`CreateFileMappingA`](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createfilemappinga)
	- [`FindClose`](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-findclose)
	- [`FindFirstFileA`](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-findfirstfilea)
	- [`FindNextFileA`](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-findnextfilea)
	- [`IsBadReadPtr`](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-isbadreadptr)
	- [`MapViewOfFile`](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-mapviewoffile)
	- [`UnmapViewOfFile`](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-unmapviewoffile)
- `msvcrt.dll` - These are all standard C library functions for Visual C++, so I won't go in to the documentation for these
	- `_XcptFilter`
	- `__getmainargs`
	- `__p___initenv` 
	- `__p__commode`
	- `__p__fmode`
	- `__set_app_type`
	- `__setusermatherr`
	- `_adjust_fdiv`
	- `_controlfp`
	- `_except_handler3`
	- `_exit`
	- `_initterm`
	- `_stricmp`
	- `exit`
	- `malloc`
   
Looking at the imports, we can infer that this binary iterates through the file system and copies files.   
   
**File**: `Lab01-01.dll`   
**Imports**:
- `kernel32.dll`
	- [`ClosHandle`](https://docs.microsoft.com/en-us/windows/win32/api/handleapi/nf-handleapi-closehandle)
	- [`CreateMutexA`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-createmutexa)
	- [`CreateProcessA`](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa)
	- [`OpenMutex`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-openmutexw)
	- [`Sleep`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-sleep)
- `ws2_32.dll` - Dependency Walker displays the ordinals for these functions, so we'll have to look at the table to match the function to the ordinal
	- [`closesocket`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-closesocket)
	- [`connect`](https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-connect)
	- [`htons`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-htons)
	- [`inet_addr`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-inet_addr)
	- [`recv`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-recv)
	- [`send`](https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-send)
	- [`shutdown`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-shutdown)
	- [`socket`](https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-socket)
	- [`WSAStartup`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-wsastartup)
	- [`WSACleanup`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/nf-winsock-wsacleanup)
- `msvcrt.dll`
    - `_adjustr_fdiv`
    - `innitterm`
    - `free`
    - `malloc`
    - `strncmp`
   
Based off the imports, this dll creates a socket, attempts to connect to web based resource, and sends and recieves some data. Knowing this, when conducting basic dynamic analysis, listeners and a fake network can be set up to trick the malware in to thinking it has internet connectivity to better understand the network resources it is attempting to access.   
   
**Q5. Are there any other files or host based indicators that you could look for on infected systems?**   
   
**File**: `Lab01-01.exe`   
When running strings against this file an interesting reference to a dll was discovered. We can look for `kerne132.dll` on the system as this appears to be an attempt to obfuscate this dll. Of note, the 'l' in kernel is replaced with a 1 (one). This technique is used to make the file hard to find since the human eye could easily recognize the 1 as an l at first glance.   
   
**Q6. What network-based indicators could be used to find this malware on infected machines?**   
   
**File**: `Lab01-01.dll`     
When running strings against this file, a reference to what appears to be an IP address is discovered. Based on the import table, it is likely that this binary attempts to connect to `127.26.152.13`.   
   
**Q7. What would you guess is the purpose of these files?**   
   
The purpose of these files is likely to install and run a backdoor. We know this because `CreateProcess` and `Sleep` are commonly used functions in backdoors. Additionally, the IP address that was found in the `dll` implies command and control functionality. Additionally, looking at the strings for the `.exe`, we see references to the `dll`. The `exe` likely looks for and/or installs the `dll`, replaces `kernel32.dll` with the `dll` as `kerne132.dll`, and then executes it.

---

## Lab 1-2:

### File Metadata:

**Filename**: `Lab01-02.exe`   
**MD5**: `8363436878404da0ae3e46991e355b83`   
   
### Questions:

**Q1. Upload the `Lab01-02.exe` file to [Virus Total](https://virustotal.com). Does it match any existing antivirus definitions?**   
Searching for the [hash](https://www.virustotal.com/gui/file/c876a332d7dd8da331cb8eee7ab7bf32752834d4b2b54eaa362674a2a48f64a6) in Virus Total, we see that this file is detected by 55 out of 69 anti-virus engines used by Virus Total.   
   
![Lab01-02.exe Virus Total 01](/assets/img/pma/ch01/picture_16.JPG){: width="750"}
<p align="center"><strong>Figure 1: Virus Total Detection Summary for <em>Lab01-02.exe</em></strong></p>
   
The majority of these anti-virus engines characterize this file as a `trojan`.   
   
![Lab01-02.exe Virus Total 02](/assets/img/pma/ch01/picture_17.JPG){: width="750"}
<p align="center"><strong>Figure 2: Virus Total AV Engine Characterization for <em>Lab01-02.exe</em></strong></p>
   
**Q2. Are there any indications that this file is packed or obfuscated? If so, what are these indicators? If the file is packed, unpack it if possible.**   
   
Running `strings` against the binary, the first few lines we see are references to UPX. This is an indication that the file is possibly packed.
   
![Lab01-02.exe Strings](/assets/img/pma/ch01/picture_18.JPG){: width="750"}
<p align="center"><strong>Figure 3: Strings Output for <em>Lab01-02.exe</em></strong></p>
   
To confirm our theory, we can load the executable in to PEiD. The results displayed indicate that the executable is infact packed using UPX1.
   
![Lab01-02.exe PEiD](/assets/img/pma/ch01/picture_19.JPG){: width="750"}
<p align="center"><strong>Figure 4: PEiD Results for <em>Lab01-02.exe</em></strong></p>

To unpack the binary, we can use the UPX1 utility. Using the command `upx -d Lab01-02.exe -oLab01-02_unpacked.exe` results in the following:   
   
![Lab01-02.exe Unpacking](/assets/img/pma/ch01/picture_20.JPG){: width="750"}
<p align="center"><strong>Figure 5: Unpacking <em>Lab01-02.exe</em></strong></p>

Running strings against the unpacked binary, we see that there are a lot more readable strings, as well as imports that the executable uses.   

![Lab01-02.exe Unpacked Strings](/assets/img/pma/ch01/picture_21.JPG){: width="750"}
<p align="center"><strong>Figure 6: Unpacked Strings for <em>Lab01-02.exe</em></strong></p>
   
We can also run the unpacked executable against PEiD to validate that the executable is unpacked. The result tells us that th executable was compiled with `Microsoft Visual C++ 6.0`:
   
![Lab01-02.exe Unpacked PEiD](/assets/img/pma/ch01/picture_22.JPG){: width="750"}
<p align="center"><strong>Figure 7: Unpacked PEiD Results for <em>Lab01-02.exe</em></strong></p>

**Q3. Do any imports hint at the program's functionality? If so, which imports are they and what do they tell you?**   
   
Loading the unpacked executable in to `Dependency Walker` we see that the executable loads `kernel32.dll`, `advapi32.dll`, `msvcrt.dll`, and `wininet.dll`, and imports the following functions from each library:

- `kernel32.dll`
	- [`SystemTimeToFileTime`](https://docs.microsoft.com/en-us/windows/win32/api/timezoneapi/nf-timezoneapi-systemtimetofiletime)
	- [`GetModuleFileNameA`](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulefilenamea)
	- [`CreateWaitableTimerA`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-createwaitabletimerw)
	- [`ExitProcess`](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-exitprocess)
	- [`OpenMutexA`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-openmutexw)
	- [`SetWaitableTimer`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-setwaitabletimer)
	- [`WaitForSingleObject`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject)
	- [`CreateMutexA`](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-createmutexa)
	- [`CreateThread`](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread)
- `advapi32.dll`
	- [`CreateServiceA`](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-createservicea)
	- [`StartServiceCtrlDispatcherA`](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-startservicectrldispatchera)
	- [`OpenSCManagerA`](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-openscmanagera)
- `msvcrt.dll`
	- `_exit`
	- `_XcptFilter`
	- `exit`
	- `_p__initenv`
	- `__getmainargs`
	- `_initterm`
	- `__setusermatherr`
	- `_adjust_fdiv`
	- `__p__commode`
	- `__p__fmode`
	- `__set_app_type`
	- `_except_handler3`
	- `_controlfp`
- `wininet.dll`
	- [`InternetOpenURLA`](https://docs.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetopenurla)
	- [`InternetOpenA`](https://docs.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetopena)
   
The most interesting imports here are the use of the `InternetOpenA` and `InternetOpenURLA`. `InternetOpenA` specifies the application that will use the `wininet` functions. Looking at the `strings` output, we can see references to `Internet Explorer 8.0`, which is likely the application that this function is targeting. `InternetOpenURLA` specifies the URL resource to locate through the application that is targeted by the `InternetOpenA` function. In this case, the `strings` output suggests that the target URL is likely the `http://www.malwareanalysisbook.com` domain. Additionally, the `CreateServiceA` function creates a service object and adds it to the service control manager. References to `MalService` in the `strings` output of the unpacked executable suggest that the executable creates this service and adds it to the service database.   
   
**Q4. What host- or network-based indicators could be used to identify this malware on infected machines?**   
   
A host-based indicator that could be used to hunt for, or signaturize, this executable would be for the creation of the `MalService` service. Additionally, threat hunters could search for DNS queries to `http://www.malwareanalysisbook.com` as a network-based indicator of compromise. This query could could also be signaturized at the host level, depending on the endpoit detection that is being used.