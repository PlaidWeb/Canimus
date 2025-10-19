# Canimus

The Canimus feed format is a syndication format to allow for federated discovery of streamable music in a platform-agnostic manner.

Canimus is the Latin first-person plural progressive form of [canō](https://en.wiktionary.org/wiki/cano#Latin), which has several meanings:

* We sing
* We play
* We make sound sound
* We chant
* We resound
* We prophesize

Or, in short, *we make music*.

## Rationale

Music discovery, consumption, and streaming is locked down by large corporations that have made a mess of things. The streaming providers have to cater to the whims of the major record labels, and have created a two-tier system where neither discovery nor payments are even remotely fair. Independent musicians are hit hardes by this, as they have to pay money to play a game that is stacked against them, and what little listenership they get is largely divvied up to the major labels. The specific payment strategy that's used has also led to a glut of minimum-effort content and AI slop and bot-driven streaming that tries to game the system, hurting everyone, big and small alike.

There are several disparate attempts to build a better world for musicians, but many of them are built on protocols that were not designed for this use case in mind. ActivityPub and RSS were simply not designed with the specific needs of distribution and discovery of musical content, and most of the existing attempts are built on top of those.

Canimus (Latin for "We Sing") is an attempt to build a lightweight syndication protocol that anyone can join in on, and which provides the much-needed structure for subscribing to musicians' streaming content in a way that enables fair payments, while also being Web-native.

This is an expansion on the ideas stated in "[A fair independent streaming platform](https://beesbuzz.biz/blog/11155-A-fair-independent-streaming-platform)."

## Glossary

The following terms are used to describe the different parts of the system:

* **feed**: The document that describes musical content for the network
* **collection**: A collection of musical content
* **receiver**: A system that subscribes to and processes feeds to add into a local musical content store
* **player**: The user-facing interface that is used to browse and listen to music known by a receiver

## Documents

For more detailed information on each part of the system, please consult the following sub-documents:

* [feed](feed.md): Defines the feed format
* [receiver](receiver.md): Defines (broadly) how a receiver may consume feeds
* [payments](payments.md): Defines (broadly) some ideas for how musicians may be supported in the network



## Player considerations

For the purpose of this document, "player" refers to the sum total of the player experience, namely the server component and whatever user-facing software connects to it, be it a web client, mobile device, or desktop application.

### Subscriptions

Generally-speaking, the player should periodically refresh the feeds that it has knowledge of at some regular interval, as well as supporting [WebSub](https://en.wikipedia.org/wiki/WebSub) for immediate "pings" when an update takes place.

If a feed or its content disappears in some way other than being explicitly deleted, it should be up to the player to handle this in a graceful manner. For example, content which has just gone offline should be listed as "temporarily unavailable," and then after a certain period of unavailability it could be removed.

The current page of the feed should always be fully processed. Any archival URLs should be retrieved if they were not already processed. This includes both `next` and `previous`.

### Collisions

In order to prevent things like malicious takeovers of band names, as well as the very common case of multiple bands having the same name, artists should be namespaced to the URL of the feed that populates the data. If multiple feeds canonically represent the same artist, there should be some mechanism for declaring the equivalence, such as using a bidirectional `rel:"me"` link between the two profiles, and ideally a `rel:"canonical"` that declares the one that should be used as the ground truth.

### Private libraries

One of the ways that the major DSPs acquired their early mindshare with the listening public was to provide a means for listeners to provide their own private purchased libraries so that they can continue to listen to their existing collections from anywhere. Any successful player engine should also allow this to happen.

Rather than requiring that people upload their (often gigantic) collections to a server somewhere and the copyright nightmares that would entail, it would be a good idea for the player engines to allow people to self-host their own connection, possibly using a service that they run from their home network (á la Plex).

Discussion of how this self-hosting approach may work is somewhat out-of-scope, but a very basic suggestion would be for users to be able to run a simple ingestion script that formats a Canimus feed and then to run a basic webserver that protects the feed and content behind some form of simple authentication, such as by providing a bearer token or HTTP basic-auth credentials.

Ideally, the data from these private collections would feed into public recommendation algorithms, but

### Discovery

Listeners should be able to publicly provide a profile of the music they are listening to, similar to so-called "scrobblers." This should be an opt-in mechanism. A listening feed should contain the following information at the very least:

* Artist, title, and associated album metadata of the song
* If available, the canonical HTML informational URL of the song and album
* If available, the validated MusicBrainz identifier of the song
* The listener's disposition (played, skipped, partial, etc.) and any ratings information that the listener provided
* If part of a public playlist, the playlist URL

This information can be used to build a per-player recommendation database, and possibly allow listeners to subscribe to each other as a form of peer-to-peer discovery and recommendation.

The most logical means of formatting the feed is as a Canimus feed with a `scrobble` entity (which would in turn build on `playlist`), but it could also be provided in other formats for compatibility with other services (Libre.fm, ListenBrainz, etc.)

### Payments

It is up to the player to collect information about which listeners have listened to which artists and send out payments accordingly.

One possible mechanism for this would be for the player to track a running "support balance" for each listener and artist, and periodically check to see which artists have reached a particular payment threshold and then encourage the listener to make a payment at a fair rate.

[TODO: rewrite this horrible mess for clarity! Maybe make an example ledger sheet or something]

For one example of how this could work, every artist and listener can maintain a running balance throughout the month. Every unit of time spent listening will transfer some rate from the listener's balance to the artist's, at a fair rate to be determined by the listener; for example, a listener who typically listens to 100 hours of music a month and thinks that's worth $10 could set their payment rate to 10¢/hour. At the end of the month, any artist whose balance exceeds a certain threshold will get that amount paid directly (zeroing out their balance), and what's left over could be divvied up among the top-performing artists such that those payments exceed the threshold but then give those artists a negative balance.

For example, if the payment threshold is $5, and there's $12 left in the pot after all artists who earned more than $5 are paid, then the artists with the two highest remaining balances each receive $6, and their balances then go negative for the next round.

In this way, even artists who don't get a lot of listening would end up randomly getting at least some amount of money in a means similar to [Floyd-Steinberg dithering](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering), and it will average out in the long run.

Content streamed from private libraries would not necessarily be subject to the balance accrual, or the balance for the library itself could be refunded back to the listener, possibly with statistics about which bands they listened to most from their library (so that it can be left up to the listener's discretion as to whether to send additional support to the artist they ***most definitely*** bought the music from already).
