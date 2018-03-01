---
layout: page
title: CyberThreat18 CTF challenge write-up - "Network A"
permalink: /post/cyberthreat18-ctf-na/
excerpt: Write-up of one of the CTF challenges from CyberThreat18, specifically we will be doing some packet capture analysis, protocol reverse engineering, and abusing flaws in the protocol to get the flag.
---

## Introduction
I recently attended a new cyber security conference in London called [CyberThreat18](https://www.cyberthreat2018.com/) hosted by the National Cyber Security Centre and SANS Institute.

Over the two-day period, the event included a Capture The Flag (CTF) competition, broken into four sessions, in which teams and individuals raced to crack the challenges and collect the most points.

This is a write-up of one of the challenges called "Network challenge A".

## Challenge description
We're given a link to download a zip file which contains the challenge assets; a packet capture file (PCAP) named `somepcap.pcapng` and an RSA private key file in PEM format named `somepem.pem`. We're also instructed that the flag we require needs to be acquired from "the service" running on `ctf-ch7.cyberthreat2018.com`. Time to crack open the PCAP!

## PCAP analysis
Opening the PCAP, we see a single TCP conversation between two hosts; a client and what we can safely assume to be "the service" alluded to from the challenge description, which in the PCAP appears to be running on port `31337`.

Following the TCP conversation in Wireshark, we see the following:

![TCP conversation between a client and the service of interest](/images/ct18-ctf-na-2.png)

Coloured blue is the data sent by the service to the client, and coloured red is the data sent by the client to the service.

At the beginning of the conversation, we see the service sent the client an RSA public key, followed by the client also sending the service an RSA public key. What follows that is... gibberish, to the best of our current estimations. Showing the data as a hexdump instead, however, reveals a bit more structure to us...

![TCP conversation shown as a hex dump](/images/ct18-ctf-na-3.png)

In particular, we see that the messages exchanged between the client and the service (after the exchanging of public keys) are always of a fixed size - 256 bytes (or 2048 bits).

This is noteworthy, because the public keys that were exchanged were 2048-bit keys, and a property of asymmetric algorithms is that the size of the data which you wish to encrypt with it cannot exceed the key size. Another property of asymmetric encryption is that the size of the encrypted data "going in" matches the size of the data "coming out".

At this point, we might theorize that the messages exchanged are encrypted asymetrically using RSA, with a high likelihood that the plaintext is padded to the 2048 bits in length prior to encryption. Specifically, when the client wishes to send a message to the service, it pads it, and RSA-encrypts it using the public key which was sent to it by the service at the start of the conversation, and vice versa for service to client communications.

## Taking a peek at the data
Recall that, as well as the PCAP file, we were also provided with a RSA private key file, called `somepem.pem`.

From this private key file, we can calculate the corresponding public key using the `openssl` tool.

```
$ openssl rsa -in somepem.pem -pubout
writing RSA key
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAjIlaZyLpXHCodCbhSREA
mpRP9vjBqELq5AodBu7zXCjZspyb/jTNGzEek4QWvqIosoZ5MDgAK1TRUG8R4Hqh
qaz19cuD1quOovKRqqJEgaRsbRx8uW5vjsG6ge6kBJk9EOlkJe3COqXzsJWTIKc0
GeHfnp88v4C1Qge3BX3bQ7K4prcAWyxGEsh14bQ/oFY4MIq0aAr4+dzYP/hWxCqt
Jz8m3R5bOW/k2J7O8a9c8A7DVM6/HGIaKyLrLNuAqwhLFJM32jSLiGOxfDHDE1Aq
/QGjEPUImUXVRok7nHjbNIDJNNCbCLzr/rWGEYVWUTia/wPcUpYVv9S5/BdiTA5s
PQIDAQAB
-----END PUBLIC KEY-----
```

A-ha, this is the same as the public key which was sent from the client to the service in our packet capture, which means that we should be able to decrypt the messages in one of the directions in the TCP conversation - those from the service to the client.

Let's attempt to decrypt the first encrypted message from the service to the client. By switching to the YAML view in Wireshark's we can grab the first message in Base 64 encoded form, and experiment with openssl's `rsautl` mode to try to decrypt it.

```
$ echo ZLgeZ+9IuHmxqfdMfOb/CdaWo/xK6hzHz0HDAXBSST2PfwqPGJ0Ly+P8q/ZwfGoD7n9LxL8p+m+f \
       k1K41QYpKphoAp+zioORUU9qDlq64ht+IWn30FKnmVaJokwUpMLnY10gPYD1MEQQFWhbIlXYSpCZ \
       X/xVvOr/emXn+xkg1KAY8q4vG5n02vmF1Rmp5ltBpqm2PqCBXDHDRO6g2259RF4NAkhF3+y+DrCK \
       /8NTVFhDVRFm5QZ0BLCWUfFJwLLFpzbIv/Q/FlnZuA87d/lgpGYZ3ajHCGHbcOdcjoanYdzGY2v3 \
       Zh5iPm6L5AviusEii2VVcT3aQkMvLybPEHyjpw== | \
>   openssl base64 -d | \
>   openssl rsautl -inkey somepem.pem -oaep -decrypt
Commands:

quit
authenticate
adduser
help
getflag
```

Bingo! The padding employed is PKCS#1 OAEP. We repeat this on the other messages sent from the service to the client to get the rest of this side of the conversation. I've used the convention of writing `> ???` to indicate the unknown client message, with the server response immediately below.

```
> ???
Commands:

quit
authenticate
adduser
help
getflag
> ???
Insufficient permissions! Please authenticate and try again.
> ???
Authentication successful.
> ???
User successfully added!
```

## Getting interactive

At this point, we can throw together a small (terribly written!!) python script to interact with the service directly, which according to the challenge description is running on `ctf-ch7.cyberthreat2018.com`, most likely on port `31337` according to the PCAP.

{% highlight python %}
from Crypto.Cipher import PKCS1_OAEP as cip
from Crypto.PublicKey import RSA
import socket
import base64

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("ctf-ch7.cyberthreat2018.com", 31337))

their = s.recv(1024)
their_key = RSA.importKey(their)

f = open('somepem.pem', 'r')
our_key = RSA.importKey(f.read())

our_public = our_key.publickey().exportKey()

s.send(our_public)

def enc(plain):
    cipher = cip.new(their_key)
    crypt = cipher.encrypt(plain)
    return crypt

def dec(crypt):
    cipher = cip.new(our_key)
    plain = cipher.decrypt(crypt)
    return plain

while True:
    cmd = raw_input("> ")
    s.send(enc(cmd))
    print dec(s.recv(256))
{% endhighlight %}

Yes, I know that's not how you should send and receive with python's sockets. Yes, I'm using `s` globally. Yes, some of the naming is poor. Under CTF conditions, this is as good as you're going to get ;P.

So, we execute our client, run the `getflag` command, and we're done right? Right?

```
$ python client.py
> getflag
Insufficient permissions! Please authenticate and try again.
```

Damn, we don't have any credentials for this service. We can't see the credentials that were used in the PCAP because we don't have the RSA private key corresponding to the service's public key (which the client uses to encrypt traffic in that direction). So what are we to do?

## Could you repeat that please?
At this point, I lost _a lot_ of time in the CTF trying various methods of authenticating with the service; I extended my python script to brute-force credentials using the rockyou wordlist (this was slow, and bore no fruit), I tried using the `adduser` command with parameters of varying sizes in case there was an overflow onto some hypothetical `is_authenticated` variable on the stack, I tried to search online for the fingerprint of the service's RSA public key in case it had been chosen from the set of weak RSA keys from back when Debian's PRNG for key generation was broken. No dice.

> "Have you considered a replay attack?"
>
> 🤦

Of course, the answer was the take the message which the client sent to the service immediately before it responded with `Authentication successful.` and send it to the service _before_ dropping us into our interactive session - that way we would be logged in, and be able to request the flag.

{% highlight python %}
from Crypto.Cipher import PKCS1_OAEP as cip
from Crypto.PublicKey import RSA
import socket
import base64

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("ctf-ch7.cyberthreat2018.com", 31337))

their = s.recv(1024)
their_key = RSA.importKey(their)

f = open('somepem.pem', 'r')
our_key = RSA.importKey(f.read())

our_public = our_key.publickey().exportKey()

s.send(our_public)

def enc(plain):
    cipher = cip.new(their_key)
    crypt = cipher.encrypt(plain)
    return crypt

def dec(crypt):
    cipher = cip.new(our_key)
    plain = cipher.decrypt(crypt)
    return plain

login_replay = """dWVSS7A86P0D8b0Ce85Z8Bg6phb3uEWIVJCyoGtp3KjQqbDfxTGweVz+aseNwF9J38msGRZt8Ox6
                  Beyb6d70jByoLNoyVyx7Ws4/lxx2TyauSs/iUcaVF9YvWW87K9QGbInQTjrMBgi2Z5WhL/HNF5Am
                  7LIdBblz79r3t6pwI9A889t3ctZuacjcKLAn/m+0DGlnIMgQpgMLFQea7yhwyX7g65UfF1VPw/cG
                  UmuXIS0QHCpTOz1ve2WbgrBBYdn8tqMiDZySzU9IyAJF0vIVuC03Cc/gsQ+vo+84f0qOWYTmz2Z3
                  AQlbmZiewh70MaIteT3cPhKKKHMkt0AyB2Ws8A=="""

s.send(base64.b64decode(login_replay))

print dec(s.recv(256))

while True:
    cmd = raw_input("> ")
    s.send(enc(cmd))
    print dec(s.recv(256))
{% endhighlight %}

Et voila

```
$ python client.py
Authentication successful.
> getflag
AllYourDataAreBelongToUs
```

(Note: that wasn't actually what the flag was - I can't remember what it was, and the service is only accessible from within the CTF network - you can safely assume it was an equivalently nerdy quote :P)

## Closing remarks
This is a good example of why you never write your own security code if you can help it. This was vulnerable to replay attacks, is succeptible to man in the middle attacks (you could sit it the middle and proxy requests, sending different public keys to the client and the service), and provides no perfect forward secrecy allowing us to decrypt the traffic in the PCAP in the first place...

A massive thank you goes to the team at [Helical Levity](https://twitter.com/HelicalLevity) for putting together the pre-conference challenges and this CTF, as well as to [James Lyne](https://twitter.com/jameslyne) and the folks at [SANS EMEA](https://twitter.com/SANSEMEA) and the [National Cyber Security Centre](https://twitter.com/ncsc).
