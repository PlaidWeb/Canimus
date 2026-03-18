# Canimus

Canimus is a syndication format to allow for federated discovery of streamable music in a platform-agnostic manner.

Canimus is the Latin first-person plural progressive form of [canō](https://en.wiktionary.org/wiki/cano#Latin), which has several meanings:

* We sing
* We play
* We make sound
* We chant
* We resound
* We prophesize

Or, in short, *we make music*.

## Rationale

Music discovery, consumption, and streaming is locked down by large corporations that have made a mess of things. The streaming providers have to cater to the whims of the major record labels, and have created a two-tier system where neither discovery nor payments are even remotely fair. Independent musicians are hit hardes by this, as they have to pay money to play a game that is stacked against them, and what little listenership they get is largely divvied up to the major labels. The specific payment strategy that's used has also led to a glut of minimum-effort content and AI slop and bot-driven streaming that tries to game the system, hurting everyone, big and small alike.

There are several disparate attempts to build a better world for musicians, but many of them are built on protocols that were not designed for this use case in mind. ActivityPub and RSS were simply not designed with the specific needs of distribution and discovery of musical content, and most of the existing attempts are built on top of those.

Canimus is an attempt to build a lightweight syndication protocol that anyone can join in on, and which provides the much-needed structure for subscribing to musicians' streaming content in a way that enables fair payments, while also being Web-native.

This is an expansion on the ideas stated in "[A fair independent streaming platform](https://beesbuzz.biz/blog/11155-A-fair-independent-streaming-platform)."

## Glossary

The following terms are used to describe the different parts of the system:

* **format**: The document that describes the data format
* **catalog**: A collection of musical content, typically containing albums and/or individual tracks
* **feed**: An update/event feed
* **receiver**: A system that subscribes to and processes feeds to add into a local musical content store
* **player**: The user-facing interface that is used to browse and listen to music known by a receiver

## Documents

For more detailed information on each part of the system, please consult the following sub-documents:

* [format](format.md): Defines the data format
* [receiver](receiver.md): Defines (broadly) how a receiver may consume Canimus data
* [payments](payments.md): Defines (broadly) some ideas for how musicians may be supported in the network

## FAQ

### Why a new format?

While there are existing feed/collection formats out there, finding one that presents a collection of items in a structured manner that properly represents the world of music is difficult. Music can come in many different shapes, and most commonly it is in the form of collections of albums of songs, and the order of the songs within those albums matters.

Much of the metadata for the items *tends* to be consistent across an entire collection, but there are always exceptions that need to be captured in some way.

Most current formats also exist to present new content in a stream of ephemera, without much attention given to older items.

The Canimus format attempts to encapsulate a collection of music, which can be browsed, revisited, and categorized, while also taking advantage of the overall structure of an album as a sequential series of related songs, without necessarily being limited to that structure.

This format is also intended to be easy to publish and to parse, without any guesswork about what anything actually means. Musicians shouldn't have to sign up for every new distributed platform that springs up, when those platforms could subscribe to a common format as one potential source for music. They should be able to just add it as a format to publish their music to the web in a way that is, hopefully, easy to adapt into other ecosystems.

### Why not use RSS/Atom?

There are several reasons that RSS/Atom are not suitable protocols for this purpose. (Going forward I will refer to both protocols as simply "RSS.")

Currently, RSS does support [audio attachments](https://www.rssboard.org/rss-specification#ltenclosuregtSubelementOfLtitemgt), but its usage is geared entirely towards podcasts. Every article represents a separate episode, and the metadata that applies to each one does a poor job of providing concepts like albums, multiple artists, and so forth.

Additionally, RSS feeds are generally paginated, meaning that they only provide the most recent content, rather than a full archive of all data. An RSS feed *can* provide full content, and there are [extensions](https://datatracker.ietf.org/doc/html/rfc5005) to add pagination as well, but client support is pretty weak and so rare as to be practically nonexistent.

Finally, since RSS is also used for many other things, such as blogs and announcements and so on, there needs to be an additional set of information to distinguish feeds and/or feed entries as being music as opposed to serialized content, and this also requires added client/consumer support.

None of these issues are insurmountable, and if consensus can be built on an extension to RSS that addresses these issues, there is no reason not to support that instead. But RSS in its present form does not provide a standard mechanism for representing a collection of music, and clients need to be built on whatever emerges. Having an RSS enclosure does not magically make everything agree on how things should work, and the RSS specification is vague enough that there are *massive* compatibility problems between implementations even for basic things like blogs and podcasts.

### Why not ActivityPub?

There are several ActivityPub-based music federation projects underway as well. ActivityPub is a great format for pushing out notifications of new content to be sure, and Canimus could indeed be implemented as a layer on top of ActivityPub. However, the promise of ActivityPub is having a universal client/server for realtime updates, and similarly to RSS, is not suitable for providing a browseable collection of specifically-structured data. Most ActivityPub implementations are also oriented towards the idea of two-way communication between client and server, and this is anathema to the notion of a collection being published to a static hosting provider or similar, and also requires a lot of active (and fragile) state to be maintained between the two.

Any implementation of a music collection on top of ActivityPub would still have to implement the collection itself, and maintain standards for how backfilling works and how the collection is shaped, so why not start with a clean implementation that only provides the parts that are important to a music collection to begin with?

### What about [missing feature]?

The specification in its current form is certainly not the final word, and can and should be extended as use cases are uncovered. Every attempt has been made to keep it extensible and flexible, but of course there will be things that are missing as well.

That is why this is hosted as a git repository with [issues](https://github.com/PlaidWeb/Canimus/issues) and [discussions](https://github.com/PlaidWeb/Canimus/discussions).

This is a starting point for something better than what exists currently.

### Why should something use Canimus instead of anything else?

It shouldn't! Different formats are good at different things. Canimus is meant to live alongside other protocols, and it purposefully excludes functionality other than sharing music. There is no intention to add any functionality like real-time status posts, blog entries, or podcasts, all of which are served better by social feed formats such as RSS and Atom.

It is also not meant to be an encyclopedic compendium of all music; this is not a replacement to MusicBrainz, for example, although the end-user's software interoperating with MusicBrainz as a source of ground truth for metadata is certainly desirable.

From a publisher's standpoint, Canimus is just another template to add to a website, to make it easier for music to be discovered and listened to.

From a player's standpoint, Canimus is just another means of obtaining a collection of music.

Nothing about this is exclusive; it's just meant to be simpler to support on both sides.

### What about access control? (paid access, subscriptions, etc.)

Access control is generally better-served at a different level on a content delivery stack than the end format.

The intention is that a Canimus collection, by default, provides that which the musician wants to be listened to, at whatever quality level makes the most sense for what is essentially a free preview.

The hope is that there will eventually be a standard protocol for allowing receivers to manage access tokens in order to fetch full-quality versions or bonus content and the like (with the same mechanism preventing someone from simply republishing the underlying media URLs unprotected). This can take many forms, such as [standard HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication) (particularly bearer tokens), but the ideal long-term goal is that people would use this as a mechanism to find music to purchase and download into their own local collections.

That local collection could then be served up in turn as a private Canimus collection; there is some discussion about how that might work in the [receiver document](receiver.md#private).
