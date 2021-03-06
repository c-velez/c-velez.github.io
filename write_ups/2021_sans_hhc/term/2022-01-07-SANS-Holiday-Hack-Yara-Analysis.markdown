---
layout: post
# title: "Yara Analysis"
date: 2022-01-07 07:57 -500
categories: jekyll update
---

# Yara Analysis

**Objective**: Yara is blocking a critical file from running. We need to edit the file in order to get it to run  
**Location**: Kringle Castle Entry  
**Hints**: 
- None

## Write Up

Yara is a tool used for Hostbase identification of malicious files.  

Looking at the home directory of the terminal, we see the critical application, `the_critical_elf_app`, as well as the `yara_rules` definitions directory.

![Home Directory Enumeration](/assets/img/2021_sans_hhc/term/yara_analysis/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: Critical App and Yara Definitions</strong></p>

### Rule 1 Bypass

Running the `the_critical_elf_app`, we see that is being blocked from execution due to `yara_rule_135`:

![First Rule](/assets/img/2021_sans_hhc/term/yara_analysis/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: Blocked by Rule 135</strong></p>

We can search the `yara_rules` definition file to see what the rule is triggering on.

```
less yara_rules/rules.yar
/yara_rule_135
```

This produces the following rule definition:

```
rule yara_rule_135 {
	meta:
		description = "binaries - file Sugar_in_the_machinery"
		author = "Sparkle Redberry"
		reference = "North Pole Malware Research Lab"
		data = "1955-04-21"
		hash = "19ecaadb2159b566c39c999b0f860b4d8fc2824eb648e275f57a6dbceaf9b488"
	strings:
		$s = "candycane"
	condition:
		$s
}
```

The metadata of the rule holds the general information about the rule. Looking at the strings section, we see that `$s` is set to `candycane`, and the condition that the rule fires on is if `$s` exists in the binary.  

We'll have to edit `the_critical_elf_app` to obfuscate, get rid of, or mangle, this string.  

I'll opt to delete every other letter in the string in this case.  

We can make vim a hex editor by doing the following:

```
vim the_critical_elf_app
:%!xxd
/candy
```

![Candycane Offset](/assets/img/2021_sans_hhc/term/yara_analysis/picture_3.png){: width="750"}
<p align="center"><strong>Figure 3: Location of the Candycane String</strong></p>

We'll make every other letter in the string a zero. Make sure to run `:%!xxd -r` before quitting the vim session. Failure to do this will corrupt the file.

![C0n0y0a0ne](/assets/img/2021_sans_hhc/term/yara_analysis/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: Editted Hexdump for Rule 135</strong></p>

### Rule 2 Bypass

We were able to bypass the first rule, but were blocked from execution due to `yara_rule_1056`.

![Rule 1056](/assets/img/2021_sans_hhc/term/yara_analysis/picture_5.png){: width="750"}
<p align="center"><strong>Figure 5: Execution Fails Due to Rule 1056</strong></p>

Looking at the rule definition reveals the following:

```
rule yara_rule_1056 {
	meta:
		description = "binaries - file frosty.exe"
		author = "Sparkle Redberry"
		reference = "North Pole Malware Research Lab"
		date = "1955-04-21"
		hash = "b9b95f671e3d54318b3fd4db1ba3b813325fcef462070da163193d7acb5fcd03"
	strings:
		$s1 = {6c 6962 632e 736f 2e36}
        $hs2 = {726f 6772 616d 2121}
    condition:
    	all of them
}
```

Looking at the rule condition, the rule will only trigger if, and only if, all of the strings are present. We'll have to take a look at the hexdump to see what these hex values are related to.

It appears that `$s1` is for `libc.so`. Messing with this set of hex will destroy the exectution of the program, so we should probably not do anything to this set of hex values.

![libc.so](/assets/img/2021_sans_hhc/term/yara_analysis/picture_6.PNG){: width="750"}
<p align="center"><strong>Figure 6: <em>$s1</em> is Related to <em>libc.so</em></strong></p>

Looking at `$hs2`, these hex values represent the string `rogram!!` in the hexdump. Since this is just a string, we can edit this part of the program to try and bypass the rule.

![Rule 1056 Second Condition](/assets/img/2021_sans_hhc/term/yara_analysis/picture_7.PNG){: width="750"}
<p align="center"><strong>Figure 6: Second String for Rule 1056</strong></p>

Adding zeroes between each letter should do the trick

![Rule 1056 Bypass](/assets/img/2021_sans_hhc/term/yara_analysis/picture_8.png){: width="750"}
<p align="center"><strong>Figure 8: Edited <em>rogram!!</em> String</strong></p>

### Rule 3 Bypass

Our bypass for `yara_rule_1056` worked, but it looks like we've encountered another rule that is blocking our execution: `yara_rule_1732`.  

![Blocked by Rule 1732](/assets/img/2021_sans_hhc/term/yara_analysis/picture_9.PNG){: width="750"}
<p align="center"><strong>Figure 9: Blocked by Rule 1732</strong></p>

Looking at the rule definition provides us with 3 conditions that could be met in order for this rulle to trigger:

```
rule yara_rule_1732 {
   meta:
      description = "binaries - alwayz_winter.exe"
      author = "Santa"
      reference = "North Pole Malware Research Lab"
      date = "1955-04-22"
      hash = "c1e31a539898aab18f483d9e7b3c698ea45799e78bddc919a7dbebb1b40193a8"
   strings:
      $s1 = "This is critical for the execution of this program!!" fullword ascii
      $s2 = "__frame_dummy_init_array_entry" fullword ascii
      $s3 = ".note.gnu.property" fullword ascii
      $s4 = ".eh_frame_hdr" fullword ascii
      $s5 = "__FRAME_END__" fullword ascii
      $s6 = "__GNU_EH_FRAME_HDR" fullword ascii
      $s7 = "frame_dummy" fullword ascii
      $s8 = ".note.gnu.build-id" fullword ascii
      $s9 = "completed.8060" fullword ascii
      $s10 = "_IO_stdin_used" fullword ascii
      $s11 = ".note.ABI-tag" fullword ascii
      $s12 = "naughty string" fullword ascii
      $s13 = "dastardly string" fullword ascii
      $s14 = "__do_global_dtors_aux_fini_array_entry" fullword ascii
      $s15 = "__libc_start_main@@GLIBC_2.2.5" fullword ascii
      $s16 = "GLIBC_2.2.5" fullword ascii
      $s17 = "its_a_holly_jolly_variable" fullword ascii
      $s18 = "__cxa_finalize" fullword ascii
      $s19 = "HolidayHackChallenge{NotReallyAFlag}" fullword ascii
      $s20 = "__libc_csu_init" fullword ascii
   condition:
      uint32(1) == 0x02464c45 and filesize < 50KB and
      10 of them
}
```

Because the condition for this rule uses `and` within the conditional statement, all of the conditions must be met. This means that if we could invalidate one of the conditions, the rule will not trigger.

Looking up the documentation for `unint32(1)`, we can note that this rule tells `yara` to look at the first byte of the file, and match what is to the right of the equality operator. Let's [convert `02464c45` to ASCII](https://www.rapidtables.com/convert/number/hex-to-ascii.html). The output is `?FLE`. This tells me 2 things: 1) the rule is reading in little endian format; 2) the rule is looking at the file header to check that it is an `ELF` file. In theory, all we would have to do it mangle the header of the file, and this should bypass the rule. Because the `ELF` header is important for execution, we'll have to mangle or delete the last byte in the series in order to not corrupt the binary.

![Before Editing](/assets/img/2021_sans_hhc/term/yara_analysis/picture_10.png){: width="750"}
<p align="center"><strong>Figure 10: <em>ELF</em> Header Before Editing</strong></p>
<p align="center">Notice how the hex appears to be "backwards" when compared to the rule <br>
	<em>454c4602</em> versus <em>02464c45</em> as defined in the rule</p>

![After Editing](/assets/img/2021_sans_hhc/term/yara_analysis/picture_11.PNG){: width="750"}
<p align="center"><strong>Figure 11: Changing <em>02</em> to <em>00</em> Should do the Trick</strong></p>

### Success

We have been able to successfully execute `the_critical_elf_app` in the environment. It is very important to test and validate host and network based rules thoroughly in your environment when creating custom rules or inheriting open source rules. With poorly tested rule sets, you can run in to a problem that could shut down operations for hours or days. It is also worth noting that signature based detection does not always work which is why it is important to have a human in the loop to validate incidents. If an adversary were able to get a hold of an organization's custom definitions, they could easily recreate the environment in their own test space to create bypasses to the rule set. Observe, validate, response.

Luckily, we caught this one quick ;)

![Full Bypass](/assets/img/2021_sans_hhc/term/yara_analysis/picture_12.PNG){: width="750"}
<p align="center"><strong>Figure 12: Success Full Bypass of All Yara Rules</strong></p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-9">< Objective 9</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>
