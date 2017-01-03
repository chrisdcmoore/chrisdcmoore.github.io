---
title: "EE Bright Box default WPA passphrases are not secure"
layout: page
permalink: /post/ee-brightbox-default-wpa-passphrases-are-not-secure/
excerpt: In this post, we look at some default EE Bright Box passphrases that are in the wild, and speculate the out-of-the-box security offered by them.
---

## Background

The security of consumer wireless routers has come a long way over the years. 

In terms of the technology available, we've gone from completely unprotected wireless networks being the norm, seen the rise and fall of the fundamentally broken WEP algorithm, and arrived at the relative security of WPA/WPA2.

A wireless network protected by WPA/WPA2 using a pre-shared key (which is what you'd encounter in typical home setups), however, is only as secure as the key itself; the handshake between a wireless device and wireless router is all you need to capture in order to perform an offline brute-force attack and recover a weak passphrase; the passphrases needs to be strong.

<!--more-->

## The current state of play

A new broadband customer will typically go through the following process when they join a provider and receive their shiny, new router:
 
 - Plug the router and necessary cables in
 - Enter the default passphrase from the label on the back/bottom of the router into their wireless devices
 - Get on with the rest of their life

The majority of users neither know nor care about the security afforded to their wireless network out of the box - if there's a password to get in, then it's secure, right? 

And so we have the situation where the majority of home wireless networks out there still use the encryption algorithm and passphrase that the router came configured with. For routers made within the last few years the former will almost certainly be WPA/WPA2, but when it comes to the default passphrase, there are manufacturers who still get this wrong.

## EE Bright Box
Case in point is the EE Bright Box, the router provided by UK mobile, phone, television and broadband provider EE. Here's an example of what you'll find on the back of the router:

{: .center}
![ee-brightbox-label.gif](/images/ee-brightbox-label.gif)

We see the default passphrase for the wireless network this router will present to the world is `gum-sleep-free`.

Well, 14 characters isn't bad at all - were it completely random, such as `Cxa?x'(|,&}Yx#`, then we'd have been on to a winner - however this is far from random; it has structure.

## So what's the structure?
I took to the internet, and despite there not being any mainstream articles covering the inadequate security shipped with the EE Bright Box, I quickly stumbled upon a relevant discussion on the HashKiller forum. 

Users of the forum had gathered Bright Box handshakes from across the UK (to which I contributed one of my own) to analyse exactly how susceptible these default passphrases were to dictionary attack. As it turned out the answer is "quite".

All of the harvested passphrases took the form of three "dictionary" words - one of length 3, one of length 4, and one of length 5 - joined by a '-', in any permutation. Here's an exhaustive list of the discovered passphrases posted on the forum:

#### 5-4-3
 - `horse-duck-dog`
 - `route-know-apt`
 - `guest-mean-apt`
 - `nerve-pick-six`
 - `truck-rank-few`

#### 4-5-3
 - `cash-sting-six`
 - `vase-boast-own`
 - `farm-blend-own`
 - `want-dwell-fit`
 - `curb-appal-top`

#### 4-3-5
 - `wait-rob-weary`

#### 3-4-5
 - `dog-duck-horse`
 - `ant-stab-ideal`
 - `cue-reply-such`

#### 3-5-4
 - `gum-sleep-free`
 - `pea-share-nice`
 - `leg-draft-good`
 - `use-teach-thin`
 - `toe-guard-calm`
 - `tea-yield-dear`

#### 5-3-4
 - `alarm-rub-male`
 - `label-fan-cool`

Close inspection of the list shows that there are common words across the different passphrases, too: `apt`, `dog`, `own`, `six`, `duck` and `horse` all appear more than once, suggesting that the "dictionary" from which they are taken may be rather small. Also, as pointed out by one of the forum members, the presence of "appal" also suggests that this is a UK English dictionary, as this is spelt "appall" in US English.

#### Keyspace estimation
Taking the `british-english-small` wordlist, we have 484 three-letter words, 1,919 four-letter words, and 3,557 five-letter words, giving 19,822,364,232 possible passphrases across the six permutations. On modern hardware, you can try nearly 1,000,000 of these per second, yielding the correct passphrase (assuming the consitutent words are in the dictionary chosen above) in little over 7 hours.

#### Conclusion
If you're reading this and have an EE Bright Box with the default passphrase, go change it! Better still, router manufacturers need to do a better job of supplying equipment with secure default configurations; breaking into a wireless network should take an order of years or decades, not hours.
