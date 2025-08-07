# Mosaic

Mosaic is a low-level social media protocol in early development.

Mosaic does not define social media application behavior, it only provides interoperable
support services such as identity, standardized cryptography, bootstrapping, data
formats, client-server protocol for posting and retrieving social media data,
network transports, mechanisms for edit/delete, etc. Mosaic intends to be as unopinionated
as possible. Social media applications themselves are then built on top of Mosaic and
can have wildly different purposes and methods of operation.

This specification is rendered with [mkdocs](https://www.mkdocs.org) using the
[material](https://squidfunk.github.io/mkdocs-material/) framework.

This specification is being developed in parallel to a
[core library](https://github.com/MikeDilger/mosaic-core). The
findings from development feed back into this specification.

## Introduction

### What user's want

The Internet is social. People want to communicate with each other. People want to post
articles for the world to read. They want to comment on other people's posts. They want
to have threaded discussions. They want to have chat room conversations that perhaps
are not saved. Sometimes they want private rooms where only invited people can participate.
People want feeds of updates from people they follow. They want to control what shows up
in these feeds. They want secure private messaging. They want privacy. They want to edit
their posts, and/or delete some of them. And there are thousands of more ways that people
want to engage with other people socially over the Internet, from livestreaming and
cooperative gaming to sharing pictures or sending money.

### What is wrong with how user's currently do social media

Mostly these desires are met currently by commercial services such as X (formerly Twitter),
Reddit, YouTube, and Discord, to name just a few of the most popular. But there are number of
new problems that commercial solutions bring to the table:

* There is little interoperability between these systems, and little desire to provide such.
* There are pressures to control and limit the kind of content that people can engage with
  within advertiser-friendly constraints.
* There is pressure to advertise.
* There are pressures to censor content which happens to run counter to the desires of the
  powerful.
* Users are lower-class participants compared to advertisers, and compared to the powerful that
  run the platforms and can (and do) ban users at a whim.
* There are legal agreements which are usually rather onerous.
* Anonymity is often not well supported.

### Respect for Users

Mosaic on the other hand is created by the users and for the users. It is our philosophy
that users and service providers are equal-class participants. Everybody is equally respected.

Mosaic is not the only social media protocol to take this user-first approach. Nostr and
Secure Scuttlebutt (among others) also take this approach.

Note that other well-known distributed protocols do not quite achieve it. Mastodon elevates the
server over the users, making servers into fiefdoms. Bluesky pretends in theory to respect the
user, but in practice it walks the same path as the commercial solutions do.

### Identity

Identity is the core mechanism by which Mosaic puts the power back into the hands of users.
A Mosaic identity is just a digital signature keypair. Anybody can create one randomly.
There are
115,792,089,237,316,195,423,570,985,008,687,907,853,269,984,665,640,564,039,457,584,007,913,129,639,936
possible keypairs in a 256-bit cryptosystem, so there is no chance that you will accidently
choose the same one as somebody else, or that somebody will stumble upon your secret key.

Once you create your own identity, you can create Mosaic records and say anything you want to
say, and digitally sign the record with this keypair. Over time people will come to know you by
your keypair.

In order to share records, you'll need to do a few additional things:

1. Arrange for a server (or more typically, multiple servers) to host them.
2. Advertise to the world which servers your identity is currently hosting it's records at.

### Service Providers

Service providers are also respected users of Mosaic. They provide servers where records
can be hosted.

Service providers can do anything they want with regards to people's records:

* They can selectively censor posts they don't want to host.
* They can ban users.
* They can block abusive IP addresses.
* They can charge for hosting services.
* They can violate the Mosaic specification (of course if they do, they are no longer Mosaic
  servers by definition).

You might at first think that, for users, this situation is no different than the commercial
situation we have now. But it is very different. Because the user controls their identity,
not the service provider. Let's look at what servers cannot do:

* They cannot stop a user from posting their content somewhere else.
* They cannot stop a user from switching to another service provider.
* They cannot delete a user's identity.
* They cannot interfere with the user's followers or who follows the user.
* They cannot edit posts or create fake posts that appear to be from the user.

Users can change service providers at any time and Mosaic is designed such that this is
seamless and transparent to all those people following the user.

One caveat is that when a user arranges to use a new provider, they will wish to copy their
data to this new provider. If they only had one prior provider and that prior provider
banned them, they could lose access to their data. This is solvable in multiple ways including
using multiple service providers for redundancy, and keeping a backup of your data locally.

Mosaic makes it straightforward to have multiple service providers.

And since Mosaic is an open protocol, anybody can stand up a server. That means users can
self-host their own server.

Because nobody can control what a server does other than the server's operator, users must be
judicious about which servers they choose to use, and clients (which act on behalf of users)
should automatically detect when servers are failing to meet the user's needs, so that the
user can be alerted to the situation and take the appropriate action.

Just like with webservers, there is no limit to how many Mosaic servers can exist (nostr has
already thousands), and we expect that everybody can find a server willing to hosts their
records, or at the very least, stand up their own server to do it. Servers need only be
Internet accessible; they don't need to be in data centers. While Mosaic is defined as a
client-server protocol, nothing stops this from being deployed in a peer-to-peer fashion.
That is, clients can also behave as servers if they are coded as such.

### Advertising your Service Providers

Users advertise their service providers in the bittorrent mainline (Kademlia) DHT.
