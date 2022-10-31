---
title:  "What is an Ethereum account?"
date:   2022-10-15 13:00:00 +0200
toc: false
categories: blockchain
tags: blockchain math
---

Something fun about being new to a topic is giving yourself permission to ask silly questions. In my case, being new to the blockchain world one of my first questions was: what exactly is an Ethereum account?

When I think of a traditional bank the first thing that comes to mind are safe deposits. A physical place where you can store your valuables and lock it with a key. In the case of crypto assets, it's tempting to believe we are physically storing them in our laptop and then locking them with a private key.

It can be surprising to realize that is not really how things work. We never have any bitcoins our dogecoins in our machine. What exists is a public record of all accounts and how many units of a particular currency are assigned to them.

The way I imagine it, is a little closer to a modern-day bank account. In a bank these days, there is no safe with my name attached to it. What exists is a database of all clients and their balances. This database is private, of course, and completely controlled and maintained by the bank. Something very convenient with banks is that you can always visit an office and as long as you can prove your identity they will be happy to let you make any transaction.

A blockchain is analogous to a bank database, a public one. Not only is it public, it is maintained by a large number of people to make sure nobody can make arbitrary changes to it. Another difference is that instead of having the full names of people and their identities, a blockchain has seemingly random strings that represent an account.

Ok, it turns out we don't physically store anything in our laptops but instead have a random identifier that represents us and a public database with a list of identifiers and balances. So how do we create one of these identifiers?

Here is where it gets technical. Anybody can create a random string of numbers and letters. The tricky part is to be able to prove you are the one who created this identifier, you are the owner of the account. There is a whole sub branch of computer science dedicated to this problem and it's called [public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography). The gist of it is that through mathematical functions you can create two numbers. One that is public and another one that proves you created that public number, a private one.

Although not used often anymore, prime numbers are a good way to understand public and private keys. It's easy to multiple two numbers and get the result but it's really hard to take a longer number and find out its factors. Let's say we have the numbers 18,947 and 65,701. Multiplying them to obtain 1,244,836,847 is trivial but if we are given the number 1,244,836,847 and asked what are its factor we are going to have a harder time finding them. We will have to multiply all possible combination of numbers until we get to that one (or divide it by all numbers from 1 to the square root of n).

In this case, we can say the private key is the combination of factors (18,947 and 65,701) while the public key is the number that results from their multiplication (1,244,836,847). The security of this framework comes from knowing that is really hard to factor numbers (imagine doing that by hand!).

Which brings us back to our question, what is an Ethereum account? it turns out an Ethereum account is a representation of the public address described above while the private key is what allows us to prove ownership of it.

How can we create one of these accounts? the answer is kind of funny. We can create them out of thin air. How so? you might ask. The first step is to come up with our private key (like the two prime numbers in the example above). Since Ethereum doesn't user prime numbers and instead uses [elliptic-curve cryptography](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography) we have to come up with a random number of 256 bits (or 64 hex characters). Once we have this random number we can use an algorithm to derive the public key (equivalent to multiplying the two prime numbers in the example). One extra implementation detail is that what we know as an Ethereum address is not really the plain public key. It is a hash of this public key. It's a one-way function that creates a signature.

In summary:
* The private key is a random number of 256 bits.
* The public key is derived from the private key (one way function).
* The address is derived from the public key (another one way function).

Below is an example in python:

```python
# make sure you have installed the web3 package
from eth_keys import keys
import secrets

# random number of 256 bits or 32 bytes
random_number_256 = secrets.token_bytes(32)

# we use the random number to create a private key object
# but remember the random number is indeed the priavte key
priv_key = keys.PrivateKey(random_number_256)

# the public key is derived from the private key
pub_key = priv_key.public_key

# the eth address is
address = pub_key.to_checksum_address()

print("Private key: ", priv_key)
print("Public key:  ", priv_key.public_key)
print("Address:     ", pub_key.to_checksum_address())
```

Here is the result of running that snippet:

```
Private key:  0x057b3944f32d301e247c1a36e3c830a19e0c178b9cf5db93f1df6d2e036e8c7e
Public key:   0x60adff74163fa04db6a36bc7737d4c7c58b38a7f9c6f6f389cd4538c598a33c05126557c5b1c4a8c30d6ab7ac1c6dcdf01597f40eda719d835f4d9ddc5c9befd
Address:      0x064140480AcA6dc0f4436BcFC363a25BE4128163
```

There you go, our own way of creating Ethereum accounts.

---

### Wait... but what if?

For the security-conscious people out there this might look very suspicious. What if the address "0x0641***" already exists and belongs to somebody else? well, in that case the previous owner would be very unlucky (or we would be very lucky?). Since we have the private key for that account, we can execute transactions transactions and transfer money out of it.

Then what prevents us from starting to randomly generate private keys and see if there is any money in the accounts associated to them? nothing, really. At least technically speaking. We can use the script above to generate addresses and then check them in [etherscan.io](https://etherscan.io/) to see if they have any money. The challenge here is the number of total possible addresses. We start with a random number of 256 bits. This means we have [2^256](https://www.wolframalpha.com/input/?i=2%5E256) possibilities, roughly ~1.158x10^77. That's one with 77 zeros after it. A pretty big number. An unimaginably big number. For comparison, the universe is about 14 billion years old or ~4.4x10^17 seconds. Even if we could try billions and billions of random numbers per second it would still take us many times over the age of the universe to go through all possibilities.

As long as the number is truly random the chances of stumbling upon an existing account are almost nil. I would be more worried about how this number is created. Are there any chances that the program used to generate it can fail?   
