---
layout: post
date: 2022-01-06 20:54 -500
categories: jekyll update
---

# Where in the World is Caramel Santiago

**Objective**: The goal of this objective is to find a wayward elf. The objective indicates that we should talk to Piney Sappington for hints on how to solve this challenge.  
**Location**: Kringle Castle Courtyard  
**Hint Terminal**: [Piney Sappington](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-Document-Analysis)  
**Hints**:
1. Flask Cookies: While Flask cookies can't generally be forged without the secret, they can often be [decoded and read](https://gist.github.com/chriselgee/b9f1861dd9b99a8c1ed30066b25ff80b)
2. OSINT Talk: Clay Moody is giving [a talk](https://www.youtube.com/watch?v=tAot_mcBT9c) about OSINT techniques right now!
2. Coordinate Systems: Don't forget coordinate systems other than lat/long like [MGRS](https://en.wikipedia.org/wiki/Military_Grid_Reference_System) and [what3words](https://what3words.com/)

## Write Up

Taking a look at the hint's, I chose to go down the Flask Cookie path, rather than play the game.  
When you open up the terminal, you are presented with the user interface for the game.  

![Game UI](/assets/img/2021_sans_hhc/obj/obj02/picture_3.png){: width="750"}
<p align="center"><strong>Figure 1: Game Interface</strong></p>

Starting the game should initialize the cookie, so let's open up dev tools and then start the game.

![Game Cookie](/assets/img/2021_sans_hhc/obj/obj02/picture_4.png){: width="750"}
<p align="center"><strong>Figure 2: Game Cookie</strong></p>

Looking at the Flask Cookie hint, the following python script should decode the generated cookie. 

```
#!/usr/bin/env python3

import zlib
import itsdangerous

cookie = <cookie>

decoded = zlib.decompress(itsdangerous.base64_decode(cookie))

print(decoded)
```

The output should give you a JSON similar to this:

```
{
  "elf": "Fitzy Shortstack",
  "elfHints": [
    "The elf mentioned something about Stack Overflow and Python.",
    "Oh, I noticed they had a Firefly themed phone case.",
    "They kept checking their Twitter app.",
    "The elf got really heated about using tabs for indents.",
    "hard"
  ],
  "location": "Santa's Castle",
  "options": [
    [
      "Tokyo, Japan",
      "Reykjav\\u00edk, Iceland",
      "New York, USA"
    ],
    [
      "Vienna, Austria",
      "Rovaniemi, Finland",
      "Prague, Czech Republic"
    ],
    [
      "Prague, Czech Republic",
      "London, England",
      "Edinburgh, Scotland"
    ],
    [
      "Placeholder",
      "Edinburgh, Scotland",
      "Montr\\u00e9al, Canada"
    ]
  ],
  "randomSeed": 876,
  "route": [
    "Reykjav\\u00edk, Iceland",
    "Vienna, Austria",
    "Prague, Czech Republic",
    "Placeholder"
  ]
}
```

In this case, the way the game works is driven by the cookie that is generated. We now know which elf we are looking for, so it's just a matter of playing the game until we have to make a decision on the elf that is being described.

![We Found the Elf!](/assets/img/2021_sans_hhc/obj/obj02/picture_5.png){: width="750"}
<p align="center"><strong>Figure 3: We Beat the Game Without Playing!</strong></p>

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-1">< Objective 1</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-3">Objective 3 ></a></p>
