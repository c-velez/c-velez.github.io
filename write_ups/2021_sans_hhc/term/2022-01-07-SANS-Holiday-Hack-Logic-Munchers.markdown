---
layout: post
# title: "Document Analysis"
date: 2022-01-07 07:57 -500
categories: jekyll update
---

# Logic Munchers

**Objective**: Beat the Trollogs!  
**Location**: Outside of Kringle Castle  
**Hints**:
1. Boolean Logic: There are lots of special symbols for logic and set notation. This one (http://notes.imt-decal.org/sets/cheat-sheet.html) covers AND, NOT, and OR at the bottom.
2. AND, OR, NOT, XOR: This (http://www.natna.info/English/Teaching/CSI30-materials/Chapter1-cheat-sheet.pdf) might be a handy reference too.

## Write Up

Let's speak with Noel Boetie outside of Kringle Castle.  

It looks like we have to play a game that involves boolean logic. In order for the task to be complete, we have to beat the game in the Potpourri setting at Stage 3 (Intermediate) or higher.  

Let's see what this is about.  

![Game UI](/assets/img/2021_sans_hhc/term/logic_munchers/picture_2.png){: width="750"}
<p align="center"><strong>Figure 1: Play Logic Chompers with Intermediate and Potpourri Settings</strong></p>

Set the appropriate settings and start the game!  

Boolean Algebra Primer:
Boolean logic using binary operation to determine states of given conditions.  
The main operators in boolean logic are AND, OR, NOT, and XOR.   
- AND (&, &&): All must be true in order for the output to be true, otherwise, the output is false
- OR (\|, \|\|): At least one must be true in order for the output to be true, otherwise the output is false
- NOT(~, !): If the input is true, the output is false. If the input is false, the output is true
- XOR(^): Only one input must be true in order for the output to be true, otherwise the output is false
- Here is a truth table for all of the above operations (1 is TRUE, 0 is FALSE):

```
		===========================================
		| a | b | a & b | a || b | ~a | !b | a ^ b|
		|-----------------------------------------|
		| 0 | 0 |   0   |   0    |  1 |  1 |   0  |
		| 0 | 1 |   0   |   1    |  1 |  0 |   1  |
		| 1 | 0 |   0   |   1    |  0 |  1 |   1  |
		| 1 | 1 |   1   |   1    |  0 |  0 |   0  |
		===========================================
```

Binary operators allow you to do bit shifts when programming. The 2 that we should be concerned with are left shift (<<) and right shift (>>).
- Left shift:
	- a << b means to take the binary input of a and shift the bit b times.
		- Example
			- a = 3, b = 2 
			- a << b -> 3 << 2 -> 0011 << 2 -> 1100
			- If b = 1, the output would be 0110, since we are onl shifting one time.
		- In this case, we are not doing a cyclic shift, so the bits fall off the end
		- Example:
			- a = 9, b = 2
			- a << b -> 9 << 2 -> 1001 << 2 -> 0100
- Right shift
	- a >> b means to take the binary input of a and shift it b times
		- Example:
			- a = 12, b = 2
			- a >> b -> 12 >> 2 -> 1100 >> 2 -> 0011
			- If b = 1, the output would be 0110
		- In this case, we are not doing a cyclic shift, so the bits call off the end
		- Example:
			- a = 3, b = 2
			- a >> b -> 3 >> 2 -> 0011 >> 2 -> 0000  
With all of the above information, you should be able to more complex examples that you may face in the challenge:

![Example Problems](/assets/img/2021_sans_hhc/term/logic_munchers/picture_3.png){: width="750"}
<p align="center"><strong>Figure 2: Example Problem Sets</strong></p>
<p align="center">Solutions for the above samples below~</p>

```
	2^^3 = 8 is True
	30 > 10 is True
	10 % 5 = 3 is False
	33 / 15 = 2 is True
	12 / 1 = 12 is True
	4^^2 = 20 is false
	(not False and False) and (True) is False
	1^^3 = 3 is false
	1 = 1 is true
	0x04 ^ 0x01 = 0x05 is True
	(not False) or (not True or True) is True
	0b0011 ^ 0b10000 = 0b10011 is True
	(True and False) or (False and False) is False
	(True or True) or (True or False) is True
	4 - 23 = -17 is false
	(1 = 0) or (not True) is False
	5^^3 = 125 is True
	2^3 = 16 is False
	0x0c || 0x06 = 0x0c is False
	3 + 29 = 32 is True
	0x0e << 3 = 0x70 is False
	0x08 ^ 0x10 = 0x18 is true
	(False or True) or (not True and False) is True
	17 >= 7 is True
	not False or False is True
	True is True
	0b1011 >> 3 = 0b0011 is false
	0x04 >> 2 = 0x05 is false
```

Good luck and watch out for the Trollogs!

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-4">< Objective 4</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>