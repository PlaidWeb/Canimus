# Canimus

The Canimus feed format is a syndication format to allow for federated discovery of streamable music in a platform-agnostic manner.

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

* **feed**: The document that describes musical content for the network
* **collection**: A collection of musical content
* **receiver**: A system that subscribes to and processes feeds to add into a local musical content store
* **player**: The user-facing interface that is used to browse and listen to music known by a receiver

## Documents

For more detailed information on each part of the system, please consult the following sub-documents:

* [feed](feed.md): Defines the feed format
* [receiver](receiver.md): Defines (broadly) how a receiver may consume feeds
* [payments](payments.md): Defines (broadly) some ideas for how musicians may be supported in the network

## FAQ

### Why not use RSS/Atom?

There are several reasons that RSS/Atom are not suitable protocols for this purpose. (Going forward I will refer to both protocols as simply "RSS.")

Currently, RSS does support [audio attachments](https://www.rssboard.org/rss-specification#ltenclosuregtSubelementOfLtitemgt), but its usage is geared entirely towards podcasts. Every article represents a separate episode, and the metadata that applies to each one does a poor job of providing concepts like albums, multiple artists, and so forth.

Additionally, RSS feeds are generally paginated, meaning that they only provide the most recent content, rather than a full archive of all data. An RSS feed *can* provide full content, and there are [extensions](https://datatracker.ietf.org/doc/html/rfc5005) to add pagination as well, but client support is pretty weak and so rare as to be practically nonexistent.

Finally, since RSS is also used for many other things, such as blogs and announcements and so on, there needs to be an additional set of information to distinguish feeds and/or feed entries as being music as opposed to serialized content, and this also requires added client/consumer support.

None of these issues are insurmountable, and if consensus can be built on an extension to RSS that addresses these issues, there is no reason not to support that instead. But RSS in its present form does not provide a standard mechanism for representing a collection of music, and clients need to be built on whatever emerges. Having an RSS enclosure does not magically make everything agree on how things should work, and the RSS specification is vague enough that there are *massive* compatibility problems between implementations even for basic things like blogs and podcasts.

### Why not ActivityPub?

There are several ActivityPub-based music federation projects underway as well. ActivityPub is a great format for pushing out notifications of new content to be sure, and Canimus could indeed be implemented as a layer on top of ActivityPub. However, the promise of ActivityPub is having a universal client/server for realtime updates, and similarly to RSS, is not suitable for providing a browseable collection of specifically-structured data. Most ActivityPub implementations are also oriented towards the idea of two-way communication between client and server, and this is anathema to the notion of a collection being published to a static hosting provider or similar, and also requires a lot of active (and fragile) state to be maintained between the two.

Any implementation of a music collection on top of ActivityPub would still have to implement the collection itself, and maintain standards for how backfilling works and how the collection is shaped, so why not start with a clean implementation that only provides the music collection to begin with?
