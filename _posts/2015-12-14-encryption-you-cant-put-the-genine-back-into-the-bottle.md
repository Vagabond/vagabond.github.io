---
layout: post
title: "Encryption: you can't put the Genie back into the bottle"
description: ""
category: rants
tags: [security]
---
{% include JB/setup %}

I've been hearing a lot of noise in the media about the strong encryption on 'social media' and on phones has been shielding and enabling terrorists and criminals to communicate securely. As someone who has at least some familiarity with encryption and security (although I am by no means an expert), this really sounds like a lot of nonsense.

Leaving aside the politics of it all, and focusing on this as a purely technical issue. The simple fact of the matter is that all the legislation and pressure on tech companies in the world isn't going to put the encryption genie back in the bottle.

We now have, between Diffie-Hellman, RSA and Elliptic Curve Cryptography (ECC) (as well as the new crop of ciphers like AES-GCM and ChaCha/Salsa20) a pretty formidable set of tools for doing strong cryptography. There's also a pretty wide array of hardware based key storage like YubiKeys, trust stores built into CPUs, etc.

Putting all this together with the wide array of places on the internet you can use as a 'dead drop' for publishing messages, governments and 'experts' can call for banning strong encryption all they want, but even if they succeeded in rolling back some of the recent advances in cryptography in consumer devices and services, there's nothing stopping people from using one of the many open source libraries like libsodium, LibreSSL, OpenSSL or GNUTLS of trivially rolling their own.

This whole proposal of regulating strong encryption is basically a flawed idea. We've known how to do strong encryption for ~40 years and, simply by bumping the key size, even those venerable systems are pretty hard to break. The more modern, and freely available, stuff is (probably) even harder to break.

If someone dropped me on a desert island and told me to build a 'secure', end-to-end encryption scheme, using only the software installed on my laptop *right now*, I suspect I could design a system that would be pretty tough to detect, let alone break. It isn't that hard to put the cryptographic building blocks together (heck, that is the whole point of the aforementioned libraries) and build something like Apple's iMessage encryption, or go even further and use file hosting or image sharing websites to publish public keys and encrypted messages. Nobody sane invents their own cryptography, they all use the same well-scrutinized building blocks (although some blocks are better than others).

Now, one could argue for the idea of 'key escrow', where every good citizen shares their private key with the government, who stores it securely until a warrant to intercept their secure communications is signed by a judge, etc. Leaving aside the sheer administrative overhead of that scheme, what is to stop me generating some other key to use, or using someone else's 'unofficial' key. It's madness, akin to when they tried to ban the DVD decryption key (which was just a big number). You can't really regulate information on the internet, people will always find a way. Furthermore, what about things like Ephemeral Diffie-Hellman, where the keys used are thrown away as soon as they serve their purpose, are we going to ban that too (because even nobody involved in that communication can decrypt it afterwards)? I will also leave aside the whole notion of trusting a government to keep sensitive information secure, and to resist the temptation to use that information without a warrant and due process.

In fact, this while obsession with centralizing the internet to make it easier to monitor and record is actually harming the robustness of the internet. I've heard people simultaneously decry the use of strong encryption while also prophesying doom due to 'cyber attacks'. You can't have it both ways. If you weaken the security and the decentralization of the internet to increase your surveillance capabilities, you also make yourself a more tempting target for the dreaded 'cyber warfare'.

Until recently we were living in a golden age of surveillance, while strong
crypto was available, it wasn't widely considered a requirement, nor was it
particularly easy to use. Things have changed, and the people watching us no longer know what we are saying and doing. This is not, historically speaking, a big change, but rather a return to the norm. I understand that this is hard for the people who have grown used to being able to see into people's lives but, for better or for worse, that time is ending and new strategies will have to be developed to respond to that fact.

When politicians or pundits call on 'silicon valley' or 'tech companies' to 'disrupt' terrorists or criminals, or they ask for a discussion about 'golden keys' they are asking the wrong questions and are merely betraying a way of thinking that no longer applies to the modern internet or technology.

TL;DR - you can ban strong crypto all you want, but in doing so you're not going to prevent anyone who really cares about secure communication from using it, you're just validating the need for it.
