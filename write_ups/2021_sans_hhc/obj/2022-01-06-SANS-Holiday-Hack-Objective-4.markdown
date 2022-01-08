---
layout: post
date: 2022-01-06 19:51 -500
categories: jekyll update
---

# Slot Machine Investigation

**Objective**: The goal of this objective is to test the security of Jack Frost's slot machines. Specifically, we need to answer the question "What does the Jack Frost Tower casino security team threaten to do when your coin total exceeds 1000?"  
**Location**: Jack Frost Tower Lobby  
**Hint Terminal**: [Noel Boetie](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-Logic-Munchers) - Outside of Santa's Castle  
**Hints**:
1. Parameter Tampering: It seems they're susceptible to [parameter tampering](https://owasp.org/www-community/attacks/Web_Parameter_Tampering)
2. Intercepting Proxies: Web application testers can use tools like [Burp Suite](https://portswigger.net/burp/communitydownload) or even right in the browser with Firefox's [Edit and Resend](https://itectec.com/superuser/how-to-edit-parameters-sent-through-a-form-on-the-firebug-console/) feature

## Write Up

Click on a slot machine to get started. Let's play some Frosty Slots!

![Internet Slot!](/assets/img/2021_sans_hhc/obj/obj04/picture_1.png){: width="750"}
<p align="center"><strong>Figure 1: Jack Frost's Shady Slot Machine</strong></p>

When we start the game we are presented with the slot machine UI. Let's go ahead and open up dev tools (F12) and go to "Network" to see what requests look like while we play the game (I'm using Google Chrome).

Let's click spin to see what a general spin looks like without changing parameters in the game.

We see the spin request in dev tools and, selecting payload, we see that there are 3 parameters that are sent to the server. Perhaps one of these is the key!

![Spin Request](/assets/img/2021_sans_hhc/obj/obj04/picture_2.png){: width="750"}
<p align="center"><strong>Figure 2: Spin Request Payload</strong></p>

Right click on spin and select "Copy as Fetch", and paste the clipboard to the console at the bottom of dev tools.

![Actual Request](/assets/img/2021_sans_hhc/obj/obj04/picture_3.png){: width="750"}
<p align="center"><strong>Figure 3: Spin Request in Console</strong></p>

In the body of the request, we see the parameters. Generally, it's a good idea to change one variable at a time so that we can understand what that variable does without any other input.

Let's see what happens when we change the bet amount to a negative number.

![Update Bet Amount](/assets/img/2021_sans_hhc/obj/obj04/picture_4.png){: width="750"}
<p align="center"><strong>Figure 4: Update Bet Amount</strong></p>


Sending the request, we see an interesting response from the server.

![Response to Updated Bet Amount](/assets/img/2021_sans_hhc/obj/obj04/picture_5.png){: width="750"}
<p align="center"><strong>Figure 5: Response to Updated Bet Amount</strong></p>


Since I don't want to lose all my my just yet, let's change the bet amount to 0 and see what happens when we change the numline amount to a negative number.

![Negative NumLine](/assets/img/2021_sans_hhc/obj/obj04/picture_6.png){: width="750"}
<p align="center"><strong>Figure 6: Negative NumLine</strong></p>


It appears the request was sent successfully, but our number of credits did not change. I'm guessing because we didn't bet anything, we don't get anything. Let's come back to this after checking out the cpl parameter.

![Response to NumLine Tampering](/assets/img/2021_sans_hhc/obj/obj04/picture_7.png){: width="750"}
<p align="center"><strong>Figure 7: Response to NumLine Tampering</strong></p>

Let's go ahead and change the bet amount to 1, numline to 1, and cpl to -0.1. Since cpl is a fraction, I think it takes a percentage of your bet and uses that instead of your full bet when playing slots.

![Tampered Ratio](/assets/img/2021_sans_hhc/obj/obj04/picture_8.png){: width="750"}
<p align="center"><strong>Figure 8: Tampered Ratio</strong></p>


It looks like our request was successful and our credit went up by 0.1! Let's win some money!

![Response to Tampered Ratio](/assets/img/2021_sans_hhc/obj/obj04/picture_9.png){: width="750"}
<p align="center"><strong>Figure 9: Response to Tampered Ratio</strong></p>

Changing the bet amount to 10000, and keeping numline as 1 and cpl as -0.1 should give us over 1000 credits.

![Fully Tampered Request](/assets/img/2021_sans_hhc/obj/obj04/picture_10.png){: width="750"}
<p align="center"><strong>Figure 10: Updated Request to Win Big Bucks</strong></p>

And that's it! We did it. But we should probably get out of here before the bouncer trolls show up.....

![Sorry I Cheated... Not!](/assets/img/2021_sans_hhc/obj/obj04/picture_11.png){: width="750"}
<p align="center"><strong>Figure 11: Sorry I Cheated... Not!</strong></p>

Answer: ||"I'm going to have some bouncer trolls bounce you right out of this casino!"||

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-3">< Objective 3</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-5">Objective 5 ></a></p>