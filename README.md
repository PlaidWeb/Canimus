# Canimus

The Canimus feed format is a syndication format to allow for federated discovery of streamable music in a platform-agnostic manner.

## Rationale

Music discovery, consumption, and streaming is locked down by large corporations that have made a mess of things. The streaming providers have to cater to the whims of the major record labels, and have created a two-tier system where neither discovery nor payments are even remotely fair. Independent musicians are hit hardes by this, as they have to pay money to play a game that is stacked against them, and what little listenership they get is largely divvied up to the major labels. The specific payment strategy that's used has also led to a glut of minimum-effort content and AI slop and bot-driven streaming that tries to game the system, hurting everyone, big and small alike.

There are several disparate attempts to build a better world for musicians, but many of them are built on protocols that were not designed for this use case in mind. ActivityPub and RSS were simply not designed with the specific needs of distribution and discovery of musical content, and most of the existing attempts are built on top of those.

Canimus (Laton for "We Sing") is an attempt to build a lightweight syndication protocol that anyone can join in on, and which provides the much-needed structure for subscribing to musicians' streaming content while also providing the information necessary to provide fair support for musicians, while also being Web-native.

## Scope

This document is meant to explain the publishing mechanism for music providers (i.e. musicians and record labels) to provide their works to syndication, and some of the mechanisms through which the consumers (i.e. radio stations and streaming players) might use this feed in order to make use of this data. It is not intended to be a fully complete implementation guide, nor does it place any specific usage patterns on the exposed data beyond suggestions of how it might be used.

As an informal document is it meant only to be descriptive and aspirational. As systems grow and develop it will almost certainly be the case that some aspects of this need to be modified or expanded upon, and at a certain point it will need to be formalized into a standard.


## Discovery and retrieval



### Feed discovery

* define HTML `<link rel>` and HTTP `Link:` headers for feed discovery

### Expected retrieval mechanism

* to be retrieved by a dumb HTTP client (i.e. no javascript)
* allowance for authentication for the purpose of private collections
* discuss use of last-modified/etags
* note that feeds are *additive*, with deletions being an extra verb to be expressed
* note that it is up to the subscriber to handle content that goes missing, e.g. listing it as "currently unavailable" and only hiding/deleting it after a certain time period of continued missing data

## Feed standard

A feed is formatted as structured data, provided in a commonly-parseable format that provides nested key-value pairs and arrays of data.

From an implementation standpoint, JSON is likely the simplest to implement and to build validation tools for. However, there can be a case for using an alternate format such as HTML+mf2, which allows the same data to be both human-readable and embed semantics in the form of microformats.

Other formats such as XML and YAML are also plausible.

For the sake of this document, the assumption will be that the feed is written in JSON format, and that there is a standard for JSON discovery from the associated webpages, ideally a `<link>`. One suggestion for this would be something like:

```html
<link rel="alternate" type="application/json+canimus" href="/path/to/feed.json">
```

in the HTML `<head>`, with the usual provisions for alternate discovery via `Link:` HTTP response headers.

Every layer is hierarchical, and can contain any number of any other layer beneath it. Any attributes which can inherit from a higher layer should do so, as a fallback.

### Common vocabulary

These are the attributes that make sense at any level in the hierarchy. They inherit unless stated otherwise.

* `url`: The canonical URL for an HTML representation of the content, e.g. the webpage for the label/artist/album/track
* `images`: A collection of images that are relevant to the display of the content, including:

    * `icon`: A small icon to represent the item
    * `artwork`: Primary artwork to be displayed in a player (primarily relevant to an album or track, but can also be used as a band fallback for things without artwork, for example)
    * `photo`: A larger photographic image representing the item (headshots, profile images, etc.)

    Each of these can be provided as a simple URL, or as a set of properties including URL, alt-text, different renditions/resolutions/formats for different viewport sizes (similar to the HTML `<picture>` element), and so on. Additional image types can also be provided for e.g. liner art, alternate covers, and so on.

* `mbid`: The associated [MusicBrainz identifier](https://musicbrainz.org/doc/MusicBrainz_Identifier), if available; this attribute does not inherit
* `links`: Associated links, including:
    * `support`: How to support this entity financially
    * `purchase`: Where to purchase a copy of this entity, ideally given as a key-value dictionary mapping the service name to the URL


### Collection

### Artist

### Album

### Disc/side

### Track

* allow for multiple content-types (`audio/mp3`, `audio/ogg-vorbis`, etc.)

### Playlist

### Deletion

### Related links

#### Pagination



todo:

* hierarchal definition
* attribute vocabulary
* collection (top-level)
* artist
* album
* track
* playlists
* deletions/revocations
* pagination/linked data
* related links (artist page, websub, etc.)
* canonical URLs vs UIDs
* MusicBrainz IDs

## Player considerations

For the purpose of this document, "player" refers to the sum total of the player experience, namely the server component and whatever user-facing software connects to it, be it a web client, mobile device, or desktop application.

### Subscriptions

Generally-speaking, the player should periodically refresh the feeds that it has knowledge of, as well as using a protocol such as [WebSub](https://en.wikipedia.org/wiki/WebSub) for immediate "pings" for when an update should take place.

If a feed or its content disappears in some way other than being explicitly deleted, it should be up to the player to handle this in a graceful manner. For example, content which has just gone offline should be listed as "temporarily unavailable," and then after a certain period of unavailability it could be removed.

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

### Payments

