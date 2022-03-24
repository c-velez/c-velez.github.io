---
layout: post
# title: "Document Analysis"
date: 2022-01-07 07:57 -500
categories: jekyll update
---

# Document Analysis

Piney Sappington needs our help!

It seems someone has tampered with Santa's Files.

Let's click on the terminal to have a look.

![Document Terminal](/assets/img/2021_sans_hhc/term/document_analysis/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: Document Terminal</strong></p>

It looks like we need to find out which record Jack Frost has modified. Luckily exiftool is installed on the terminal.

exiftool is a tool that allows you to analyze the metadata of a file. This includes creation times, access times, modification time, and a load of other metadata.

Let's go ahead and run `exiftool * | less`. This command will run exiftool against all files in the current directory, and pipe the output to less.
You can search in less by inputting `/<search term>`. In our case, let's search for `Jack`

It looks like Jack modified the file before 2021-12-22.docx.

Let's go ahead and verify this by running `exiftool 2021-12-21.docx`

Looking at the bottom of the output, we can indeed see that Jack Frost modified 2021-12-21.docx.

![We Found Jack Frost](/assets/img/2021_sans_hhc/term/document_analysis/picture_3.PNG){: width="750"}
<p align="center"><strong>Figure 2: We Found Jack Frost!</strong></p>

Let's validate this by entering this answer to the top terminal of the RPi terminal.
And just like that we found the correct document!

![Success](/assets/img/2021_sans_hhc/term/document_analysis/picture_5.png){: width="750"}
<p align="center"><strong>Figure 3: Success!</strong></p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-2">< Objective 2</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>
