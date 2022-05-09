---
layout: post
# title: "Practical Malware Analysis Chapter 1 Labs"
date: 2022-05-05 08:37 -500
categories: jekyll update
---

# Lab 1-2

## File Metadata:

**Filename**: `Lab01-02.exe`   
**MD5**: `8363436878404da0ae3e46991e355b83`   
   
## Questions:

**Q1. Upload the `Lab01-02.exe` file to [Virus Total](https://virustotal.com). Does it match any existing antivirus definitions?**   
Searching for the [hash](https://www.virustotal.com/gui/file/c876a332d7dd8da331cb8eee7ab7bf32752834d4b2b54eaa362674a2a48f64a6) in Virus Total, we see that this file is detected by 55 out of 69 anti-virus engines used by Virus Total.   
   
![Lab01-02.exe Virus Total 01](/assets/img/pma/ch01/lab02/picture_16.JPG){: width="750"}
<p align="center"><strong>Figure 1: Virus Total Detection Summary for <em>Lab01-02.exe</em></strong></p>
   
The majority of these anti-virus engines characterize this file as a `trojan`.   
   
![Lab01-02.exe Virus Total 02](/assets/img/pma/ch01/lab02/picture_17.JPG){: width="750"}
<p align="center"><strong>Figure 2: Virus Total AV Engine Characterization for <em>Lab01-02.exe</em></strong></p>
   
**Q2. Are there any indications that this file is packed or obfuscated? If so, what are these indicators? If the file is packed, unpack it if possible.**   
   
Running `strings` against the binary, the first few lines we see are references to UPX. This is an indication that the file is possibly packed.
   
![Lab01-02.exe Strings](/assets/img/pma/ch01/lab02/picture_18.JPG){: width="750"}
<p align="center"><strong>Figure 3: Strings Output for <em>Lab01-02.exe</em></strong></p>
   
To confirm our theory, we can load the executable in to PEiD. The results displayed indicate that the executable is infact packed using UPX1.
   
![Lab01-02.exe PEiD](/assets/img/pma/ch01/lab02/picture_19.JPG){: width="750"}
<p align="center"><strong>Figure 4: PEiD Results for <em>Lab01-02.exe</em></strong></p>

To unpack the binary, we can use the UPX1 utility. Using the command `upx -d Lab01-02.exe -oLab01-02_unpacked.exe` results in the following:   
   
![Lab01-02.exe Unpacking](/assets/img/pma/ch01/lab02/picture_20.JPG){: width="750"}
<p align="center"><strong>Figure 5: Unpacking <em>Lab01-02.exe</em></strong></p>

Running strings against the unpacked binary, we see that there are a lot more readable strings, as well as imports that the executable uses.   

![Lab01-02.exe Unpacked Strings](/assets/img/pma/ch01/lab02/picture_21.JPG){: width="750"}
<p align="center"><strong>Figure 6: Unpacked Strings for <em>Lab01-02.exe</em></strong></p>
   
We can also run the unpacked executable against PEiD to validate that the executable is unpacked. The result tells us that th executable was compiled with `Microsoft Visual C++ 6.0`:
   
![Lab01-02.exe Unpacked PEiD](/assets/img/pma/ch01/lab02/picture_22.JPG){: width="750"}
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
   
The most interesting imports here are the use of the `InternetOpenA` and `InternetOpenURLA`. `InternetOpenA` specifies the application that will use the `wininet` functions. Looking at the `strings` output, we can see references to `Internet Explorer 8.0`, which is likely the application that this function is targeting. `InternetOpenURLA` specifies the URL resource to locate through the application that is targeted by the `InternetOpenA` function. In this case, the `strings` output suggests that the target URL is likely the `http://www[.]malwareanalysisbook[.]com` domain. Additionally, the `CreateServiceA` function creates a service object and adds it to the service control manager. References to `MalService` in the `strings` output of the unpacked executable suggest that the executable creates this service and adds it to the service database.   
   
**Q4. What host- or network-based indicators could be used to identify this malware on infected machines?**   
   
A host-based indicator that could be used to hunt for, or signaturize, this executable would be for the creation of the `MalService` service. Additionally, threat hunters could search for DNS queries to `http://www[.]malwareanalysisbook[.]com` as a network-based indicator of compromise. This query could could also be signaturized at the host level, depending on the endpoit detection that is being used.


---
<p align="center"><a href="/write_ups/pma/ch01/chapter_01_directory">Chapter 1 Labs Directory</a> :: <a href="/practical-malware-analysis/">Practical Malware Analysis Labs Directory</a></p>