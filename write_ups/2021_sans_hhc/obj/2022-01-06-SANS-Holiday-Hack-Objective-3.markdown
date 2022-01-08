---
layout: post
# title: Thaw Frost Tower's Entrance
date: 2022-01-06 21:05 -500
categories: jekyll update
---

# Thaw Frost Tower's Entrance

**Objective**: The goal of Objective 3 is to gain access to Jack Frost Tower by melting the front door  
**Location**: Outside Jack Frost Tower  
**Hint Terminal**: [Greasy Gopherguts](/write_ups/2021_sans_hhc/term/2022-01-07-SANS-Holiday-Hack-Grepping-for-Gold)  
**Hints**:
1. Linux Wi-Fi Commands THe [iwlist](https://linux.die.net/man/8/iwlist) and [iwconfig](https://linux.die.net/man/8/iwconfig) utilities are key for managing Wi-Fi from the Linux command line
2. Web Browsing with cURL: [cURL](https://linux.die.net/man/1/curl) makes HTTP requests from a terminal - in Mac, Linux, and modern Windows!
3. Adding Data to cURL requests: When sending a POST request with [data](https://www.educative.io/edpresso/how-to-perform-a-post-request-using-curl), add --data-binary to your curl command followed by the data you want to send.

## Write Up

Approaching Jack Frost Tower, we can see that the entrance is frozen shut.

![Frozen Door](/assets/img/2021_sans_hhc/obj/obj03/picture_2.png){: width="750"}
<p align="center"><strong>Figure 1: Jack Tower's Frozen Door</strong></p>

It looks like Grimy McTrollkins is frozen out of Jack Frost Tower and needs help getting back in. Talking to them, nudges us to use the Wi-Fi dongle we collected during Objective 1. Let's open up a prompt for the dongle (badge > Items > Open WiFi CLI).

![Wi-Fi Dongle CLI](/assets/img/2021_sans_hhc/obj/obj03/picture_3.png){: width="750"}
<p align="center"><strong>Figure 2: Wi-Fi Dongle CLI</strong></p>

After the prompt is loaded we see that iwlist and iwconfig are installed in the terminal.

Looking at the man page for iwlist, it looks like there is a scanning option that will give us a list of Access Points that are in range. All we need is the interface to use, so lets run `iwconfig` to get that.

![`iwconfig` Output](/assets/img/2021_sans_hhc/obj/obj03/picture_5.png){: width="750"}
<p align="center"><strong>Figure 3: <em>iwconfig</em> Output</strong></p>

We see our only interface is `wlan0`, so lets craft our scan command.

`iwlist wlan0 scanning`

![`iwlist` Scan Results](/assets/img/2021_sans_hhc/obj/obj03/picture_6.png){: width="750"}
<p align="center"><strong>Figure 4: <em>iwlist</em> Scan Results</strong></p>

Looking at the output, we have the radio information for the wireless thermostat in the building, including the ESSID "FROST-Nidus-Setup".

Reading the man page for `iwconfig` it looks like we can connect to the network using the `essid` option. Let's run `iwconfig wlan0 essid FROST-Nidus-Setup`

![Successful Connection!](/assets/img/2021_sans_hhc/obj/obj03/picture_7.png){: width="750"}
<p align="center"><strong>Figure 5: Thermostat Connection</strong></p>

It looks like we have successfully associated with the thermostat. Lets do some recon!

Running `curl http://nidus-setup:8080/`, we are met with a nice landing area and places to go from here. We want to increase the temperature of the thermostat, surely there is an api command for that....

Let's check the api docs with `curl htpp://nidus-setup:8080/apidoc`

![Read the Docs](/assets/img/2021_sans_hhc/obj/obj03/picture_11.png){: width="750"}
<p align="center"><strong>Figure 6: Nidus Thermostat API Calls</strong></p>

It looks like we can access the cooler api without registering the device. The output of the API doc gives us a lot of clues including the command to run as well as a temperature threshold, so let's try setting the temperature to something a little more comfortable...

`curl -XPOST -H 'Content-Type: application/json' --data-binary '{"temperature": 70}' http://nidus-setup:8080/api/cooler`

It looks like we got a warning!

![API Output](/assets/img/2021_sans_hhc/obj/obj03/picture_9.png){: width="750"}
<p align="center"><strong>Figure 7: Defrosting....</strong></p>

And the door is now open!

![Open Sesame!](/assets/img/2021_sans_hhc/obj/obj03/picture_10.png){: width="750"}
<p align="center"><strong>Figure 7: Defrosted Door</strong></p>

Let's go see what mischief Jack Frost is conducting on this holiday season!

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-2">< Objective 2</a> :: <a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-4">Objective 4 ></a></p>