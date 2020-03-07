+++
title = "Ensuring integrity, authenticity and non-repudiation in data transmission using node.js"
date = "2020-02-29"
author = "Glauber Magalhães"
cover = ""
tags = ["node.js", "cybersecurity"]
slug = "ensuring-integrity-authenticity-and-non-repudiation-in-data-transmission-using-nodejs"
description = "This article is proposed to explain a few and well known Cryptographic Primitives to ensure integrity, authenticity and non-repudiation in data transmission using node.js"
showFullContent = false
showComments = true
+++

## Core concepts of Information Security

- **Integrity:** can the recipient be confident that the message has not been modified during its lifecycle?
- **Authentication:** can the recipient be confident that the message was originated from the sender?
- **Non-repudiation:** if the recipient passes the message and the proof to a third party, can the third party be confident that the message was originated from the sender?
- **Availability:** the information must be available when it is needed. This concept will not be covered by this article.

These concepts are also called Security Goals, when we want to apply them to our systems. We can achieve these goals by using Cryptographic Primitives.

Cryptographic primitives are well-established, low-level cryptographic algorithms that are frequently used to build cryptographic protocols for computer security systems. These routines include, but are not limited to, one-way hash functions and encryption functions.

In the table below, we can see the Security Goals that some Cryptographic Primitives can provide. In this article I will be covering examples of HMAC and Digital Signatures.

{{< highlight plain >}}
    Cryptographic Primitive | Hash |    MAC    | Digital Signature
    Security Goal           |      |           | 
    ------------------------+------+-----------+------------------
    Integrity               |  Yes |    Yes    |   Yes
    Authentication          |  No  |    Yes    |   Yes
    Non-repudiation         |  No  |    No     |   Yes
    ------------------------+------+-----------+------------------
    Kind of keys            | none | symmetric | asymmetric
                            |      |    keys   |    keys
{{< / highlight >}}

## Definition of MAC

A Message authentication Code (MAC) is a short piece of information used to authenticate a message. In other words, it's used to confirm that the message came from an expected sender and has not been changed without your knowledge. The MAC value ensures both the **integrity** and **authenticity** of a message, by regenerating it in the recipient using a shared secret **key (K)**.

![MAC](/imgs/ensuring-integrity-authenticity-and-non-repudiation-in-data-transmission-using-nodejs/mac.png)

## Definition of HMAC

A **Keyed-Hash Message Authentication Code** (HMAC) is a MAC obtained by running a cryptographic hash function (like MD5, SHA1 or SHA256) over the data and a shared secret key.

The main difference between MAC and HMAC is that MAC is a tag or a piece of information that helps to authenticate a message, while HMAC is a special type of MAC with a [cryptographic hash function](https://en.wikipedia.org/wiki/Cryptographic_hash_function) and a secret key.

HMACs are almost similar to digital signatures. Both enforce **integrity** and **authenticity**. Both use cryptographic keys, and both use hash functions. The main difference is that digital signatures use asymmetric keys, while HMACs use symmetric keys.

HMACs may not enforce **non-repudiation** because the sender and the receiver share the same secret key.

## Definition of Digital Signature

A digital signature is created with a private key, and verified with the corresponding public key of an asymmetric key-pair. Only the holder of the private key can create this signature.

The public keys are available to everyone. The private key is known only by the owner and can’t be derived from a public key. When something is encrypted with the public key, only the corresponding private key can decrypt it. In addition, when something is encrypted with the private key, then anyone can verify it with the corresponding public key.

## Example of HMAC-SHA256 with **node.js**

Due to some HMAC's properties (especially its cryptographic strength), it's highly dependent on its underlying hash function, a particular HMAC is usually identified based on that hash function. So we have HMAC algorithms that go by the names of HMAC-MD5, HMAC-SHA1, or HMAC-SHA256. This last one is cryptographically stronger than the others, so in this article, I'll provide an example of HMAC using SHA256 algorithm. See it below:

### Creating a hash for the message
{{< gist glaubermagal e0a7378c4136a733b5e4e954bafac753 >}}

### Validating the message
{{< gist glaubermagal ec4bdbbea958d69b49d60b8a9d760a4f >}}

Apparently, just comparing the two hashes as strings would be enough, but you should compare the time of generation of hashes in order to avoid timing attacks. For more information about this sort of issue, see [Coda Hale’s blog post](https://codahale.com/a-lesson-in-timing-attacks/) about the timing attacks on KeyCzar and Java’s `MessageDigest.isEqual()`.

Suppose that a client is sending a message to a server, but the message or the hash was modified during its transmission, configuring a [man-in-the-middle attack](https://www.cloudflare.com/learning/security/threats/man-in-the-middle-attack/). In this case, the server should detect the possible attack and reject the request.


**Now, let's play!**

This will generate the HMAC of the `body` object with the `sharedSecret`
{{< highlight bash >}}
    npm run get_hmac
    
    // $ HMAC generated: <COPY_THIS_HMAC>
{{< / highlight >}}

This will validate the HMAC provided by the `get_hmac` method. Try changing the provided HMAC, the `body` or the `sharedSecret` and see what happens!
{{< highlight bash >}}
    npm run --hmac=<PASTE_THE_HMAC_HERE> validate_hmac
    
    // $ Valid HMAC | Error: ...
{{< / highlight >}}

This code is also available on [this github repo](https://github.com/glaubermagal/hmac-example).

## Example of Digital Signature with **node.js**

Digital Signatures work with asymmetric key-pair, one private and others public. In the example below, we will be generating the keys with [OpenSSL](https://github.com/openssl/openssl).

Create a folder for your project and create a `/keys` folder on its root. After that, generate the key-pair. You can follow some simple examples in the lines below:

**Creating a private key on keys folder**
{{< highlight bash >}}
    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -pkeyopt rsa_keygen_pubexp:3 -out private_key.pem
{{< / highlight >}}

**Creating a public key on keys folder**
{{< highlight bash >}}
    openssl pkey -in private_key.pem -out public_key.pem -pubout
{{< / highlight >}}

**Create some document in the project root (follow an artistic example below)**
{{< highlight plain >}}
    IMITATION
    
    by Edgar Allan Poe
    
    A dark unfathomed tide
    Of interminable pride
    A mystery, and a dream,
    Should my early life seem;
    I say that dream was fraught
    With a wild and waking thought
    Of beings that have been,
    Which my spirit hath not seen,
    Had I let them pass me by,
    With a dreaming eye!
    Let none of earth inherit
    That vision on my spirit;
    Those thoughts I would control,
    As a spell upon his soul:
    For that bright hope at last
    And that light time have past,
    And my wordly rest hath gone
    With a sigh as it passed on:
    I care not though it perish
    With a thought I then did cherish.
{{< / highlight >}}

The code below is responsible by creating a digital signature from your document using your private key.


### Sign the document
{{< gist glaubermagal b4bd5ea9d8229fbeb4e55c70ff980308 >}}

The code below is responsible by validating the authenticity, integrity and non-repudiation of your  document using your public key.


### Validating the document
{{< gist glaubermagal c8f2330ed5176acabbc38dfa832e8db4 >}}

**Now, let's play!**

This will write the signature of your document in the file signature.txt:
{{< highlight bash >}}
    npm run sign
    
    // $ Digital Signature: ...
{{< / highlight >}}

This will verify the signature of your document. Try changing the content of the document or the signature and see what happens:
{{< highlight bash >}}
    npm run verify
    
    // $ Digital Signature Verification: (true or false)
{{< / highlight >}}

You can take a look on these code on [my github repo](https://github.com/glaubermagal/digital-signature-example).

## What Cryptographic Primitive should you use?

This depends on the context. Sometimes you will communicate between only two servers on a private network and there's not the necessity of validating **non-repudiation**, so a simple HMAC validation should be enough. Sometimes your system will be shared by an uncountable number of clients and servers, so in this case you might be interested in enforcing the **non-repudiation,** instead of just sharing the secret key with anyone.

## Conclusion

This is a basic explanation about how to implement some Information Security Goals using some known Cryptographic Primitives using node.js. You should be aware that there are many kinds of advanced attacks that should be prevented by using other security layers and strategies.

Thanks for reading! :)



**References**

[https://en.wikipedia.org/wiki/Message_authentication_code](https://en.wikipedia.org/wiki/Message_authentication_code)
[https://codahale.com/a-lesson-in-timing-attacks/](https://codahale.com/a-lesson-in-timing-attacks/)
[https://crypto.stackexchange.com/questions/5646/what-are-the-differences-between-a-digital-signature-a-mac-and-a-hash?newreg=90481a425fb14ff0a7250cde651e77e8](https://crypto.stackexchange.com/questions/5646/what-are-the-differences-between-a-digital-signature-a-mac-and-a-hash?newreg=90481a425fb14ff0a7250cde651e77e8)
[https://resources.infosecinstitute.com/wireless-attacks-unleashed/](https://resources.infosecinstitute.com/wireless-attacks-unleashed/)
[https://en.wikipedia.org/wiki/Stream_cipher_attacks](https://en.wikipedia.org/wiki/Stream_cipher_attacks)
[https://en.wikipedia.org/wiki/Cryptographic_hash_function](https://en.wikipedia.org/wiki/Cryptographic_hash_function)
[https://www.jscape.com/blog/what-is-hmac-and-how-does-it-secure-file-transfers](https://www.jscape.com/blog/what-is-hmac-and-how-does-it-secure-file-transfers)
[https://www.jscape.com/blog/bid/84422/Symmetric-vs-Asymmetric-Encryption](https://www.jscape.com/blog/bid/84422/Symmetric-vs-Asymmetric-Encryption)
[https://www.cloudflare.com/learning/security/threats/man-in-the-middle-attack/](https://www.cloudflare.com/learning/security/threats/man-in-the-middle-attack/)
[https://resources.infosecinstitute.com/non-repudiation-digital-signature/#gref](https://resources.infosecinstitute.com/non-repudiation-digital-signature/#gref)