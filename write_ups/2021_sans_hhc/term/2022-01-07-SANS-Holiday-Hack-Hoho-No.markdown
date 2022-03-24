---
layout: post
# title: "Hoho...No"
date: 2022-01-07 07:57 -500
categories: jekyll update
---

# HoHo...No

**Objective**: Eve Snowshoes needs help configuring Fail2Ban to stop attacks on the Kringle Castle network  
**Location**: Santa's Office  
**Hints**:   
1. The KringleCon Fail2Ban [talk](https://www.youtube.com/watch?v=Fwv2-uV6e5I) is tremendously helpful for this.

## Write Up

After watching the talk, we know there are a few key things we need to configure:
1. An action definition file - this tells Fail2Ban what to do when a filter criteria is met
2. A jail definition file - this tells Fail2Ban what to do with offending hosts
3. A filter file - this tells Fail2Ban what to look for.

Let's start with our action file.

## Action Configuration

When we open up the terminal, we are immediately given a set of instructions on how to ban and unban IP addresses

![Instructions](/assets/img/2021_sans_hhc/term/hoho_no/picture_1.PNG){: width="750"}
<p align="center"><strong>Figure 1: How to Block IPs</strong></p>

The terminal specifies that to add IP addresses to the block list, we must use `/root/naughtylist add <IP address>`, and to take off the block list, we must use `/root/naughtylist del <IP address>`

With this information we can create our action configuration. 

Our action configuration must live in `/etc/fail2ban/action.d`. So let's make a file in there called `hoho_action.conf` and populate it:

```
[Definition]
actionban   = /root/naughtylist add <ip>
actionunban = /root/naughtylist del <ip>
```

With this configured, we can move on to our jail

## Jail Configuration

Along with instructions on how to add and remove offensive IP addresses from the blocklist, we are given a criteria for how many failures within a given amount of time is defined as malicious. Specifically `if an IP generates 10 or more failure messages within an hour, then it must be added to the naughty list`.

We have our criteria, let's make our jail. Our jail configuration must live in `/etc/fail2ban/jail.d`, so let's make a file there called `hoho_jail.conf` and populate it:

```
[hoho_jail]
enabled  = true                # Enforce this Jail
logpath  = /var/log/hohono.log # Audit this log
findtime = 1h                  # Time criteria
maxretry = 10                  # Maximum number of retries
bantime	 = 30m                 # How long to ban
filter   = hoho_filter         # Which filter is associated with this jail
action   = hoho_action         # Which action configuration file to use
```

Let's work on our filter configuration

## Filter Configuration

Fail2Ban uses what is called *failregex* to define filters the highlight potentially malicious or anomalous activity.

The filter configuration must live in `/etc/fail2ban/filter.d`. Let's create an empty filter configuraiton with `touch hoho_filter.conf`. We will populate it as we find log entries that signify failures.

The [documentation](https://fail2ban.readthedocs.io/en/latest/filters.html) is fairly robust, but the information given in the talk is enough to create our filters.

The first thing we need to do is look through the log file to see what key phrases we can use for failure messages in our `hoho_filter.conf`.

Doing an intial `cat` on the log file, we see a couple of things:
1. There is an entry for valid heartbeats, so a failure for a heartbeat could use the keyword `invalid`
2. We see general requests that have completed successfully, so we could use `unsuccessful` as a keyword to look for general failures
3. We see malformed requests being sent. These could be considered failures as well since the host is failing to connect
4. We see successful logins, so there could feasibly be unsuccessful logins as well

Let's grep for `invalid` first and see what we get:

`grep --ignore-case invalid /var/log/hohono.log`

This produces several `invalid` heartbeat log entries.

![Invalid Heartbeats](/assets/img/2021_sans_hhc/term/hoho_no/picture_2.PNG){: width="750"}
<p align="center"><strong>Figure 2: Invalid Heartbeats</strong></p>

With this, let's populate `hoho_filter.conf` with our first entry.

```
[Definition]
failregex = Invalid heartbeat '.*' from <HOST>$
```

It is important to note that `failregex` already filters through the date time stamp on the log, so we don't have to generate regex for the timestamp.

The `'.*'` looks for anything that is between the single quotes, as we want all invalid heartbeats, regardless of what was sent.

The `<HOST>` keyword tells Fail2Ban where the IP address is within the string, and uses that in the action configuration to act against the IP.

Let's keep looking for other failures, grepping for `unsuccessful`:

```
grep --ignore-case unsuccessful /var/log/hohono.log
```

This did not generate any entries, so let's try `fail` as a keyword:
```
grep --ignore-case fail /var/log/hohono.log
```

It looks like this keyword produced entries for failed logins for accounts:

![Failed Logins](/assets/img/2021_sans_hhc/term/hoho_no/picture_3.PNG){: width="750"}
<p align="center"><strong>Figure 3: Failed Logins</strong></p>

Let's go back to our `hoho_filter.conf` and generate a regex for these entries. I will build upon what we've already created to show the evolution of the configuration

```
[Definition]
failregex = Invalid heartbeat '.*' from <HOST>$
			Failed login from <HOST> for ([a-z]{1,9})$
```

I chose `[a-z]{1,9}` as the regex filter because the largest username that was used was 9 characters. It is entirely plausible that usernames larger than 9 characters could be used, so this would have to change in a production system. However, this works for this challenge.

Let's look for malformed requests and generate our regex for these entries

```
grep --ignore-case malformed /var/log/hohono.log
```

![Malformed Requests](/assets/img/2021_sans_hhc/term/hoho_no/picture_4.PNG){: width="750"}
<p align="center"><strong>Figure 4: Malformed Requests</strong></p>

A malformed request that is sent could be form of fingerprinting or fuzzing for an actor to get more information about a system, so let's generate our regex for these entries

```
[Definition]
failregex = Invalid heartbeat '.*' from <HOST>$
			Failed login from <HOST> for ([a-z]{1,9})$
			<HOST> sent a malformed request$
```

We've created regex for 3 out of our four potential filters.

If a username is not correct, it could potentially be rejected, so let's use that as a keyword:

```
grep --ignore-case rejected /var/log/hohono.log
```

And it looks like the hunch is correct:

![Rejected Logins](/assets/img/2021_sans_hhc/term/hoho_no/picture_5.PNG){: width="750"}
<p align="center"><strong>Figure 5: Rejected Logins</strong></p>

So our `hoho_filter.conf` should now look like:

```
[Definition]
failregex = Invalid heartbeat '.*' from <HOST>$
			Failed login from <HOST> for ([a-z]{1,9})$
			<HOST> sent a malformed request$
			Login from <HOST> rejected due to unknown user name$
```

## Get the Ban Hammer

With all of our configurations complete, we can now execute the ban:

```
# service fail2ban restart
# /root/naughtylist refresh
```

![Success](/assets/img/2021_sans_hhc/term/hoho_no/picture_6.PNG){: width="750"}
<p align="center"><strong>Figure 6: We've Successfully Defended the Castle</strong></p>

And just like that, we have successfully found all the offensive systems!

---
<p align="center"><a href="/write_ups/2021_sans_hhc/obj/2022-01-06-SANS-Holiday-Hack-Objective-8">< Objective 8</a> :: <a href="/2021-SANS-Holiday-Hack-Challenge/">Main Write Up ></a></p>