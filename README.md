# Chorus

Chorus is a lightweight syndication format for federated music discovery and streaming in a platform- and distribution-agnostic manner.

## Rationale

Music discovery, consumption, and streaming are locked down by large corporations that have made a mess of things. The streaming providers have to cater to the whims of the major record labels, and have created a two-tier system where neither discovery nor payments are even remotely fair. Independent musicians are hit hardes by this, as they have to pay money to play a game that is stacked against them, and what little listenership they get is largely divvied up to the major labels. The specific payment strategy that's used has also led to a glut of minimum-effort content and AI slop and bot-driven streaming that tries to game the system, hurting everyone, big and small alike.

There are several disparate attempts to build a better world for musicians, but many of them are built on protocols that were not designed for this use case in mind. ActivityPub and RSS were simply not designed with the specific needs of distribution and discovery of musical content, and most of the existing attempts are built on top of those.

The Chorus format is a lightweight, web-native protocol that enables an ecosystem of independent music streaming, directly connecting musicians with listeners and removing the entrenched music industry from the equation. This allows us to build a better future for independent music on the Internet while reducing the headache for musicians.

This is an expansion of the ideas brainstormed in "[A fair independent streaming platform](https://beesbuzz.biz/blog/11155-A-fair-independent-streaming-platform)."

## History

A previous iteration of the protocol was called "Canimus," which is the first-person plural progressive conjugation of the Latin word [canō](https://en.wiktionary.org/wiki/cano#Latin), roughly meaning "we make music."

Canimus feeds were initially provided as a proof-of-concept by [Sockpuppet](https://sockpuppet.band/), [Song Fight!](https://songfight.org/), and [Mirlo](https://mirlo.space), and a few consumers implemented support for it, including [Fairplayer](https://fairplayer.band/).

While Canimus was a good starting point, over time several deficiencies led to the need to overhaul key aspects of it, and this was a good opportunity to change the name, as well.

While "Chorus" as a name is a reference to independent musicians making and sharing music together, it has a casual backronym of "Corporate Hell Or Us."

## Glossary

The following terms are used to describe the different roles within the network:

* **publisher**: A source of data into the network, such as a website operated by an independent artist, music label, or publishing platform
* **receiver**: A system that subscribes to and aggregates Chorus collections as a backing storage for a player; also sometimes called a "client"
* **player**: The user-facing interface that is used to browse and listen to music known by a receiver; this may be tightly-coupled with the receiver, or may be connected in some other way (such as through an M3U-generating API or a live audio stream)

## Documents

For more detailed information on each part of the system, please consult the following sub-documents:

* [format](format.md): Defines the Chorus data format
* [receiver](receiver.md): Defines (broadly) how a receiver may consume Chorus data
* [payments](payments.md): Defines (broadly) some ideas for how musicians may be supported in the network
* [examples](examples.json): An example feed demonstrating several different use cases

## FAQ

### Why this format?

While there are existing feed/collection formats out there, finding one that presents a collection of items in a structured manner that properly represents the world of music is difficult. Music can come in many different shapes, and most commonly it is in the form of collections of albums of songs, and the order of the songs within those albums matters.

Much of the metadata for the items *tends* to be consistent across an entire collection, but there are always exceptions that need to be captured in some way.

Most current formats also exist to present new content in a stream of ephemera, without much attention given to older items.

The Chorus format encapsulates a collection of music, which can be browsed, revisited, and categorized, while also taking advantage of the overall structure of an album as a sequential series of related songs, without necessarily being limited to that structure.

This format is intended to be easy to publish and to parse, without any guesswork about what anything actually means. Musicians shouldn't have to sign up for every new distributed platform that springs up, when those platforms could subscribe to a common format as one potential source for music. They should be able to just add it as a format to publish their music to the web in a way that is, hopefully, easy to adapt into other ecosystems.

### Why not use RSS/Atom?

There are several reasons that RSS/Atom are not suitable protocols for this purpose. (Going forward I will refer to both protocols as simply "RSS.")

Currently, RSS does support [audio attachments](https://www.rssboard.org/rss-specification#ltenclosuregtSubelementOfLtitemgt), but its usage is geared entirely towards podcasts. Every article represents a separate episode, and the metadata that applies to each one does a poor job of providing concepts like albums, multiple artists, and so forth.

Additionally, RSS feeds are generally paginated, meaning that they only provide the most recent content, rather than a full archive of all data. An RSS feed *can* provide full content, and there are [extensions](https://datatracker.ietf.org/doc/html/rfc5005) to add pagination as well, but client support is pretty weak and so rare as to be practically nonexistent.

Finally, since RSS is also used for many other things, such as blogs and announcements and so on, there needs to be an additional set of information to distinguish feeds and/or feed entries as being music as opposed to serialized content, and this also requires added client/consumer support.

None of these issues are insurmountable, and if consensus can be built on an extension to RSS that addresses these issues, there is no reason not to support that instead. But RSS in its present form does not provide a standard mechanism for representing a collection of music, and clients need to be built on whatever emerges. Having an RSS enclosure does not magically make everything agree on how things should work, and the RSS specification is vague enough that there are *massive* compatibility problems between implementations even for basic things like blogs and podcasts.

### Why not ActivityPub?

There are several ActivityPub-based music federation projects underway as well. ActivityPub is a great format for pushing out notifications of new content to be sure, and Chorus could indeed be implemented as a layer on top of ActivityPub. However, the promise of ActivityPub is having a universal client/server for realtime updates, and similarly to RSS, is not suitable for providing a browseable collection of specifically-structured data. Most ActivityPub implementations are also oriented towards the idea of two-way communication between client and server, and this is anathema to the notion of a collection being published to a static hosting provider or similar, and also requires a lot of active (and fragile) state to be maintained between the two.

Any implementation of a music collection on top of ActivityPub would still have to implement the collection itself, and maintain standards for how backfilling works and how the collection is shaped, so why not start with a clean implementation that only provides the parts that are important to a music collection to begin with?

ActivityPub itself also doesn't solve the issues of interoperability between different implementations; notably, existing ActivityPub implementations such as Bandwagon, Funkwhale, and PeerTube which all live in the same general problem domain still cannot meaningfully share data with one another, as they speak different vocabularies. Even different versions of the same ActivityPub implementation are not guaranteed to interoperate with one another.

### What about [OpenSubsonic](https://opensubsonic.netlify.app/)?

OpenSubsonic is a pretty good protocol for connecting players to collections! However, it is really built around there being a single collection that's being provided to a user's player, and most players do not support aggregating across multiple OpenSubsonic collections.

What makes a lot of sense is having a single OpenSubsonic collection server that aggregates other sources, including Chorus (acting as a receiver) and other OpenSubsonic implementations, providing a unified library to the user's endpoint.

In this world, Chorus connects collections to sources, and OpenSubsonic connects players to collections.

Basically, OpenSubsonic is solving a different part of the stack, and it absolutely complements Chorus. The two protocols can work together to form a greater whole.

### Why not microformats?

While microformats are a very nice means of embedding metadata into HTML documents without requiring so-called "sidecar" formats, the unfortunate reality is that many content platforms these days are not serving up static HTML; instead they are built as app-style frontends which require Javascript to fetch the actual page content using frameworks like ReactJS and Vue, and while it would be really nice if we were still in a server-side-rendering-first web, that is no longer the case.

Additionally, parsing and presenting microformats can be a lot more complicated even in a pure-static HTML-based ecosystem, and it doesn't provide any greater efficiency than linking to a sidecar file or endpoint.

As such, while microformats are a very nice approach in theory, the practical concerns make it not very compelling for this use case.

Additionally, it presents multiple performance issues; since all of the data is embedded into the page which the data is about, you have to either embed a complte catalog into the root page of a website (which vastly increases the page retrieval overhead for everyone, including mobile web users), or clients need to do a full tree walk in order to find updates to items, and there is no clear mechanism for signifying whether a dependency has to be retrieved. This could be worked around by adding an optional content hash to every linked item, but that increases the complexity for both publishers and consumers.

Microformats also tend to make use of the page URL as the identifier, which adds difficulty to the process of reconciling URL changes. Microformats do support a `p-uid` property which can be used to reconcile this, but it isn't widely supported.

Also, since every entity is mapped to a URL, there is no standard way to present content that does not map to a specific webpage.

### Why not JSON-LD?

JSON-LD is meant to work similarly to microformats, with a large JSON blob embedded into a webpage. While this works well for discovery, it has major performance issues for actual data ingestion, and implications for the performance of the website as a whole. The choices are either to embed a complete catalog into the webpage for the root entity (which slows down every single webpage retrieval unnecessarily), or requires linking the data across webpages per entity and having consumers do a full tree walk, which in turn requires retrieving basically every webpage on a site in order to get the latest information, as there is no standard mechanism for indicating whether a dependency has changed.

JSON-LD is also far less efficient than microformats, as the embedding structure is parallel to the webpage itself, and in many cases has significantly more overhead than the microformats markup.

Additionally, JSON-LD uses the entity's URL as its identifier, which vastly complicates things when it comes to changing the URL of an item, which can happen when a webpage gets reorganized or if an item changes its name.

### What about [missing feature]?

The specification in its current form is certainly not the final word, and can and should be extended as use cases are uncovered. Every attempt has been made to keep it extensible and flexible, but of course there will be things that are missing as well.

### Why should something use Chorus instead of anything else?

It shouldn't! Different formats are good at different things. Chorus is meant to live alongside other protocols, and it purposefully excludes functionality other than publishing music. There is no intention to add any functionality like real-time status posts, blog entries, or podcasts, all of which are served better by social feed formats such as RSS and Atom.

It is also not meant to be an encyclopedic compendium of all music; this is not a replacement to MusicBrainz, for example, although the end-user's software interoperating with MusicBrainz as a source of ground truth for metadata is certainly desirable.

From a publisher's standpoint, Chorus is just another template to add to a website, to make it easier for music to be discovered and listened to.

From a receiver's standpoint, Chorus is just a means of obtaining a collection of music, which can work alongside other mechanisms.

Nothing about this is exclusive; it's just meant to be simpler to support on both sides, and flexible enough to provide real choices to both producers and consumers of music.

Don't think "instead of," but "in addition to."

### What about access control? (paid access, subscriptions, etc.)

Access control is generally better-served at a different level on a content delivery stack than the end format.

The intention is that a Chorus collection, by default, provides that which the musician wants to be listened to, at whatever quality level makes the most sense for what is essentially a free preview.

The hope is that there will eventually be a standard mechanism for allowing receivers to manage access tokens in order to fetch full-quality versions or bonus content and the like (with the same mechanism preventing someone from simply republishing the underlying media URLs unprotected). This can take many forms, such as [standard HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication) (particularly bearer tokens) or something OAuth-based, but the ideal long-term goal is that people would use this as a mechanism to find music to purchase and download into their own local collections.

That local collection could then be served up in turn as a private Chorus collection; there is some discussion about how that might work in the [receiver document](receiver.md#private).

The overall goal of this project is to turn listeners into collectors. The existence of music should not be contingent on the long-term durability of its hosting provider.
