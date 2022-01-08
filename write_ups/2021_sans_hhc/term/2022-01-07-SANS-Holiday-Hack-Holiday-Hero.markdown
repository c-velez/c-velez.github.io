---
layout: post
# title: "Holiday Hero"
date: 2022-01-07 07:57 -500
categories: jekyll update
---

# Holiday Hero

**Objective**: We need to power up Santa's Sleigh  
**Location**: Netwars - Roof of Kringle Castle  
**Hints**: 
1. Chimney Scissorsticks says that there is a way to enable single player mode.

## Write Up

Click the sleigh and create a room. It looks like we need to play the game in order to charge Santa's Sleigh. Let's snoop around to see what we can find.

![Main Menu](/assets/img/2021_sans_hhc/term/holiday_hero/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: Holiday Hero Main Menu</strong></p>

![Instructions](/assets/img/2021_sans_hhc/term/holiday_hero/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: How to Play Holiday Hero</strong></p>

Opening up dev tools in Chrome, browse to the sources to get a glimpse of the game engine

![Read the Source Code](/assets/img/2021_sans_hhc/term/holiday_hero/picture_3.png){: width="750"}
<p align="center"><strong>Figure 3: Holiday Hero Code</strong></p>

Reading the source carefully, we see code block that appears to be related to making Player 2 a computer. The condition is that `single_player_mode` must be true.

![Interesting Code Block](/assets/img/2021_sans_hhc/term/holiday_hero/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: Interesting Code Block</strong></p>

Let's go to the console and see if we can manipulate that variable. Make sure you are in the right context for the variable. By default, dev tools in chrome is in the context of the top most window. We can change this by selecting the drop down and moving to `hero.kringlecastle.com`

![Dropdown](/assets/img/2021_sans_hhc/term/holiday_hero/picture_6.png){: width="750"}
<p align="center"><strong>Figure 5: Make Sure The Context is Right</strong></p>

![Set the Variable](/assets/img/2021_sans_hhc/term/holiday_hero/picture_5.png){: width="750"}
<p align="center"><strong>Figure 6: Enable Single Player Mode</strong></p>

It looks like we have activated single player mode as the player 2 computer has shown up!

![Computer Player is Set!](/assets/img/2021_sans_hhc/term/holiday_hero/picture_7.png){: width="750"}
<p align="center"><strong>Figure 7: Player 2 has Entered the Game!</strong></p>

Now toggle player 1 on and get that sleigh fueled up!

![Player 1 Ready](/assets/img/2021_sans_hhc/term/holiday_hero/picture_8.png){: width="750"}
<p align="center"><strong>Figure 8: Player 1 Ready</strong></p>

![Rock On](/assets/img/2021_sans_hhc/term/holiday_hero/picture_9.png){: width="750"}
<p align="center"><strong>Figure 9: Rock On!</strong></p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-6">< Objective 6</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>