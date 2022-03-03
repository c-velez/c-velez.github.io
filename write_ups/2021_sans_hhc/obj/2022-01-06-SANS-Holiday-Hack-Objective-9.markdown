---
layout: post
date: 2022-01-07 14:07 -500
categories: jekyll update
---

# Splunk

**Objective**: One of the Elves has recently left the job. We need to figure out the work they have been doing... or do we? 
**Location**: The Great Room  
**Hint Terminal**: [Fitzy Shortstack](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-Yara-Analysis)  
**Hints**:
1. Sysmon Monitoring in Splunk - Sysmon network events don't reveal the process parent ID for example. Fortunately, we can pivot with a query to investigate process creation event once you get a process ID
2. Malicious NetCat?? Did you know there are multiple versions of the Netcat command that can be used maliciously? `nc.openbsd`, for example.
3. Github Monitoring in Splunk - Between Github audit log and webhook event recording, you can monitor all activity in a repository, including common `git` commands such as `git add`, `git status`, and `git commit`.

## Write Up

This challenge will help introduce us to git monitoring with Splunk. We'll have to answer a series of questions to get the whole story on what Eddie has been up to...

### Task 1

"Capture the commands Eddie ran most often, starting with git. Looking only at his process launches as reported by Sysmon, record the most common git-related CommandLine that Eddie seemed to use."

In short, we have to figure what git command Eddie liked to use the most.  

Let's build our query using template number 1:

```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational 
process_name="/usr/bin/git" EventDescription="Process creation"
```

Selecting "CommandLine", we can see that Eddie ran `git status` the most.

![Task 1 Query](/assets/img/2021_sans_hhc/obj/obj09/picture_1.PNG){: width="750"}
<p align="center"><strong>Figure 1: Task 1 Query</strong></p>

### Task 2

"Looking through the git commands Eddie ran, determine the remote repository that he configured as the origin for the `partnerapi` repo. The correct one!"  

Pivoting from our previous query, let's look for all `CommandLine` execution where the `CurrentDirectory` contains `partnerapi`

```
index=main source="Journald:Microsoft-Windows-Sysmon/Operational" 
process_name="/usr/bin/git" EventDescription="Process creation" 
CommandLine="*partnerapi*"
```

Looking at the CommandLine values again, we see that Eddie used `git@github.com:elfnp3/partnerapi.git` as the origin for the `partnerapi` repo.

![Task 2 Query](/assets/img/2021_sans_hhc/obj/obj09/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: Task 2 Query</strong></p>

### Task 3

"The 'partnerapi' project that Eddie worked on uses Docker. Gather the full dockercommand line that Eddie used to start the 'partnerapi' project in his workstation."  

Since we are still searching for command line operations, we can edit our query in Task 2 to search for instances of "docker" in the command line parameter. We'll also have to change the process name to docker.

```
index=main source="Journald:Microsoft-Windows-Sysmon/Operational" 
process_name="/usr/bin/docker" EventDescription="Process creation" 
CommandLine="*docker*"
```

Eddie used the command `docker compose up` to start the docker container.

![Task 3 Query](/assets/img/2021_sans_hhc/obj/obj09/picture_3.png){: width="750"}
<p align="center"><strong>Figure 3: Task 3 Query</strong></p>

### Task 4

"Eddie had been testing automated static application security testing (SAST) in GitHub. Vulnerability reports have been coming into Splunk in JSON format via GitHub webhooks. Search all the events in the main index in Splunk and use the sourcetype field to locate these reports. Determine the name of the vulnerable GitHub repository that the elves cloned for testing and document it here. Inspect the repository.name field in Splunk."

For this query, we will have to switch our sourcetype, and then inspect the `repository.name` field as suggested by the question, to figure out which vulnerable repository was used. 

```
index=main source="github_json"
```

The `dvws-node` repository was the one that was cloned for testing

![Task 4 Query](/assets/img/2021_sans_hhc/obj/obj09/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: Task 4 Answer</strong></p>

### Task 5

"Santa asked Eddie to add a JavaScript library from NPM to the 'partnerapi' project. Determine the name of the library and record it here for our workshop documentation."  

In order to install a JavaScript library with npm, we would use `node npm install <library>`. Let's look for command line instances of `npm install` in our main index.

```
index=main source="Journald:Microsoft-Windows-Sysmon/Operational" CommandLine="*npm install*" 
CommandLine="*"
```

The above query produces a promising result. It looks like Eddie installed the `holiday-utils-js` library.

![Task 5 Query](/assets/img/2021_sans_hhc/obj/obj09/picture_5.png){: width="750"}
<p align="center"><strong>Figure 5: Task 5 Query</strong></p>

### Task 6

"Another elf started gathering a baseline of the network activity that Eddie generated. Start with their search and capture the full process_name field of anything that looks suspicious."  

The query that was given to us definitely looks for all of Eddie's network activity.

```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational EventCode=3 
user=eddie NOT dest_ip IN (127.0.0.*) NOT dest_port IN (22,53,80,443) | stats count by dest_ip 
dest_port
```

Reviewing the processes that generated the activity, it appears that `/usr/bin/nc.openbsd`, a known malicious process, was seen generating activity in the network. Uh oh...

![Task 6 Query](/assets/img/2021_sans_hhc/obj/obj09/picture_6.png){: width="750"}
<p align="center"><strong>Figure 6: Task 6 Results</strong></p>

### Task 7

"Uh oh. This documentation exercise just turned into an investigation. Starting with the process identified in the previous task, look for additional suspicious commands launched by the same parent process. One thing to know about these Sysmon events is that Network connection events don't indicate the parent process ID, but Process creation events do! Determine the number of files that were accessed by a related process and record it here."   

The first thing we need to do is find the process that executed `/usr/bin/nc.openbsd`. We can do this by searching for all instances with the `process_name="/usr/bin/nc.openbsd`, and then looking at the parent process id of the executed program.

Our full query looks like:

```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational 
process_name="/usr/bin/nc.openbsd" parent_process_id="*"
```

![Task 7 PPID](/assets/img/2021_sans_hhc/obj/obj09/picture_7.png){: width="750"}
<p align="center"><strong>Figure 7: Parent Process ID of <em>nc.openbsd</em></strong></p>

It appears that `/bin/bash` is the culprit, so let's pivot form here to see how many files were accessed.

```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational 
process_name="/bin/bash"
```

Looking at the `file_name` field on the left side, we can see that this process accessed `6` files in the `/tmp/` directory. Threat actors are known to stage malicious files in this directory.

![Task 7 Questionable Files](/assets/img/2021_sans_hhc/obj/obj09/picture_8.png){: width="750"}
<p align="center"><strong>Figure 8: Questional Files Accessed by <em>/bin/bash</em></strong></p>

### Task 8

"Use Splunk and Sysmon Process creation data to identify the name of the Bash script that accessed sensitive files and (likely) transmitted them to a remote IP address."   

For this task, we will have to look at the parent process of the shell that was created. Using our first query in Task 7, we know that the shell process that was spun up was `6788`. With this information we can find the parent process id.  

```
index=main sourcetype=journald source=Journald:Microsoft-Windows-Sysmon/Operational 
process_id="6788" parent_process_id="*"
```

The above query generates one result. This record shows that process `6784` generated the malicious shell. Additionally, we can look at the `ParentCommandLine` field to determine what command invoked the shell. It appears the that `/bin/bash preinstall.sh` script is the malicious script.

![Task 8 Malicious Script](/assets/img/2021_sans_hhc/obj/obj09/picture_9.png){: width="750"}
<p align="center"><strong>Figure 9: Got'em</strong></p>

### Success

Find the malicious script, Santa is apparently impressed with our skill and calls us a `whiz`

![Success](/assets/img/2021_sans_hhc/obj/obj09/picture_10.PNG){: width="750"}
<p align="center"><strong>Figure 10: Impressed Santa</strong></p>


---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-8">< Objective 8</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-11">Objective 11 ></a></p>