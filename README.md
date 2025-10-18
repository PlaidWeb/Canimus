# Canimus

The Canimus feed format is a syndication format to allow for federated discovery of streamable music in a platform-agnostic manner.

## Rationale

Music discovery, consumption, and streaming is locked down by large corporations that have made a mess of things. The streaming providers have to cater to the whims of the major record labels, and have created a two-tier system where neither discovery nor payments are even remotely fair. Independent musicians are hit hardes by this, as they have to pay money to play a game that is stacked against them, and what little listenership they get is largely divvied up to the major labels. The specific payment strategy that's used has also led to a glut of minimum-effort content and AI slop and bot-driven streaming that tries to game the system, hurting everyone, big and small alike.

There are several disparate attempts to build a better world for musicians, but many of them are built on protocols that were not designed for this use case in mind. ActivityPub and RSS were simply not designed with the specific needs of distribution and discovery of musical content, and most of the existing attempts are built on top of those.

Canimus (Latin for "We Sing") is an attempt to build a lightweight syndication protocol that anyone can join in on, and which provides the much-needed structure for subscribing to musicians' streaming content while also providing the information necessary to provide fair support for musicians, while also being Web-native.

## Scope

This document is meant to explain the publishing mechanism for music providers (i.e. musicians and record labels) to provide their works to syndication, and some of the mechanisms through which the consumers (i.e. radio stations and streaming players) might use this feed in order to make use of this data. It is not intended to be a fully complete implementation guide, nor does it place any specific usage patterns on the exposed data beyond suggestions of how it might be used.

As an informal document is it meant only to be descriptive and aspirational. As systems grow and develop it will almost certainly be the case that some aspects of this need to be modified or expanded upon, and at a certain point it will need to be formalized into a standard.

## Feed standard

A feed is formatted as structured data, provided in a commonly-parseable format that provides nested key-value pairs and arrays of data. Every hierarchical layer represents a single entity, which may contain other entities.

From an implementation standpoint, JSON is likely the simplest to implement and to build validation tools for. However, there can be a case for using an alternate format such as HTML+mf2, which allows the same data to be both human-readable and embed semantics in the form of microformats.

Other formats such as XML and YAML are also plausible.

For the sake of this document, the assumption will be that the feed is written in JSON format, and that there is a standard for JSON discovery from the associated webpages, ideally a `<link>`. One suggestion for this would be something like:

```html
<link rel="alternate" type="application/json+canimus" href="/path/to/feed.json">
```

in the HTML `<head>`, with the usual provisions for alternate discovery via `Link:` HTTP response headers.

### Entity definitions

These are the entities which can be defined by the feed.

The following attributes apply to all types of entity:

* `type`: The type of entity being defined
* `name`: The common name of the entity
* `url`: The canonical URL for an HTML representation of the current entity, e.g. the webpage for the label/artist/album/track
* `uid`: An opaque, permanent string identifier to uniquely identify this entity; optional but recommended, and defaults to `url` if not specified

* `images`: A collection of images that are relevant to the display of the content, including:

    * `icon`: A small icon to represent the item
    * `artwork`: Primary artwork to be displayed in a player (primarily relevant to an album or track, but can also be used as a band fallback for things without artwork, for example)
    * `photo`: A larger photographic image representing the item (headshots, profile images, etc.)

    Each of these keys maps to a dictionary of properties, including:

    * `alt`: The accessibility alt-text of the image
    * `media`: A list of media for the image, each a dictionary with properties including:
        * `src`: The URL to access the image
        * `width` and `height`: The pixel dimensions of the image
        * `type`: The MIME content type of the image (e.g. `image/png`, `image/jpeg`)

* `summary`: A short description of the entity, intended to be one single line of plain text.
* `description`: A longer description of the entity, formatted as HTML, to be sanitized by the consumer.

* `mbid`: The associated [MusicBrainz identifier](https://musicbrainz.org/doc/MusicBrainz_Identifier), if available and applicable
* `links`: Associated links, including:
    * `related`: Related links for the entity itself; for example:

        * `me`: An alternate URL that is also trusted to represent this entity
        * `canonical`: The URL that is considered the canonical representation of this entity on the web

    * `support`: How to support this entity financially, given as a key-value dictionary mapping the service name (e.g. `ko-fi`, `patreon`, etc.) to the URL
    * `purchase`: Where to purchase a copy of this entity, given as a key-value dictionary mapping the service name (e.g. `bandcamp`, `mirlo`, etc.) to the URL
    * `websites`: A list of key-value pairs for related websites for this entity, given as a list of `label`, `url`, and optional `type`, e.g.:

        ```json
        [{
            "label": "homepage",
            "url": "https://bandpage.example.com/"
        },
        {
            "label": "blog",
            "url": "https://bandpage.example.com/blog"
        },
        {
            "label": "feed",
            "url": "https://bandpage.example.com/feed",
            "type": "application/atom+xml"
        },
        {
            "label": "calendar",
            "url": "https://bandpage.example.com/live/calendar"
        },
        {
            "label": "fediverse",
            "url": "https://fedi.example.com/@exampleband"
        }]
        ```

* `children`: A list of entities that are contained by this entity.

#### Top level

The top-level entity does not have a `type` and refers to the feed itself.

It provides the following additional `related` links (all optional):

* `hub`, which links to a [WebSub](https://en.wikipedia.org/wiki/WebSub) hub, where a consumer can subscribe to immediate updates to this feed
* Pagination links, as described in the "Pagination" section below

#### Collection

An entity of type `collection` is a generic grouping that can contain any number of any other entities. It has the following additional attributes:

* `collection-type`: What type of collection it is; possible values would include `library`, `label`, `distributor`, and so on

#### Artist

An entity of type `artist` is a performing artist. The `name` attribute refers to the primary name under which the artist releases.

#### Album

An entity of type `album` is a collection of songs. The `name` attribute refers to the title of the album. It contains the following additional properties:

* `artist`: The releasing artist of this album (also known as "album artist"). It defaults to the name of the containing `artist`. If no such name is specified, it is up to the consumer how to display the album-level attribution.
* `subtitle`: The subtitle of the album
* `copyright`: The copyright information of the album
* `license`: Any additional license information, e.g. `"CC by-nc-sa"`

#### Track

An entity of type `track` refers to a playable track. If it is contained by an `album`, then it is given a playback order based on its position in the album's `children`; otherwise it is assumed to be a single.

It has the following additional properties:

* `title`: The title of this track
* `subtitle`: The subtitle of the track, if any
* `artist`: The releasing artist of this track, which may differ from the album artist
* `album`: The name of the album on which this track appears, if any; this is normally implied by the containing `album`, and in the case of a single release, may be blank
* `composer`: The composer of the track
* `original-artist`: The original performing artist, if this song is a cover
* `duration`: The canonical length of the track, in seconds
* `media`: A set of descriptors for to streamable versions of the audio. This *should* at the very least contain an `audio/mp3` version for maximum compatibility, but may also contain other versions. Each descriptor contains the following properties:

    * `type`: The content-type of the media (e.g. `audio/mp3`, `audio/flac`, `video/mp4`, `application/x-mpegURL`, etc.) (required)
    * `src`: The URL at which the media can be played (required)
    * `size`: The size of the content file, in bytes (optional but recommended)
    * `duration`: The duration of the media, in seconds, if it differs from the canonical length (optional but recommended)

    There can be multiple media with the same type, differentiated by `size` to indicate different quality levels/bitrates, so that player applications can choose the appropriate quality level based on bandwidth availability.

* `disc`: The physical disc that the track appeared on, in the case of a multi-disc album
* `track`: The physical track number for the track on its disc

* `copyright`: The copyright information of the album (defaults to the album's if unspecified)
* `license`: Any additional license information, e.g. `"CC by-nc-sa"` (defaults to the album's if unspecified)

* `lyrics`: The lyrics of the song, if any; this should be provided as plain text with a single `\n` between lines, and `\n\n` between verses. Limited Markdown (such as `*emphasis*` and `**boldface**`) may be supported at the discretion of the consumer.

Note that `disc` and `tracknum` are purely for display purposes, and do not affect the natural playback order of the track

An example track might look like:

```json
{
    "type": "track",
    "artist": "The Example Band",
    "name": "Introduction",
    "subtitle": "Radio Edit",
    "uid": "13a93b29-4e4b-4967-a077-cbe8491767ec",
    "url": "https://example.com/band/releases/introduction.html",
    "duration": 45,
    "disc": 1,
    "track": 1,
    "media": [{
        "type": "audio/mp3",
        "src": "https://cdn.example.com/artist/album/01 the introductory track.mp3",
        "size": 737280
    }, {
        "type": "audio/mp3",
        "src": "https://cdn.example.com/artist/album/01 the introductory track.mp3",
        "size": 1843200,
        "comment": "high-quality version"
    }, {
        "type": "video/mp4",
        "src": "https://cdn.example.com/artist/videos/the introductory track.mp4",
        "duration": 120,
        "size": 384198472,
        "comment": "music video"
    }]
}
```

### Playlist

A curated list of music to listen to, including tracks and albums. This can be useful for an artist to publish a "best of" or a mixtape or the like. It has the following additional properties:

* `author`: The author of the playlist

The `children` of the playlist must contain complete copies of the metadata for the elements being represented, including any implied attributes such as `artist` and `album`. For example:

```json
{
    "type": "playlist",
    "author": "Example Curator",
    "chilren": [{
        "type": "track",
        "artist": "Example Band",
        "name": "Hit Single",
        "subtitle": "So tired",
        "duration": 120,
        "album": "Self-Titled Album",
        "url": "https://example.com/band/releases/hit-single.html",
        "media": [{
            "type": "audio/mp3",
            "src": "https://cdn.example.com/artist/album/07 hit single.mp3",
            "size": 2949120
        }],
        "disc": 1,
        "track": 3,
        "uid": "efd71467-3b9c-483c-a081-175f6a6f1a74"
    }, {
        "type": "track",
        "artist": "Another band",
        "name": "A bigger fish to fry",
        "url": "https://example.com/other-band/fish.html",
        "media": [{
            "type": "audio/mp3",
            "src": "https://cdn.example.com/other-band/fish.mp3",
            "size": 2949120
        }],
        "uid": "1120a795-e51b-4666-b8f5-1904dd8b568f"
    }]
}
```

As with `album`, the position in the `children` list is what indicates the natural playback order of the song within the playlist; `track` and `disc` are used only for display purposes.

### Deletion

At the top level of the feed, there is an optional attribute, `delete`, which is a list of the unique identifiers of the content that has been removed. These deletion identifiers should use the `uid` property, or the original `url` if the `uid` was not used. The original item should no longer appear anywhere else in the feed or its paginated views.

Consumers of this property should take care to ensure that *only* items which were originally provided by this feed are able to be deleted by it (to avoid malicious removal requests coming from other feeds).

### Pagination

Many feeds will be much too large for all data to be provided in a single view, and so there must be a means of breaking the feed up into chunks that can be incrementally retrieved. In order to facilitate this, the feed's `related` links may contain the following link types:

* `self`: The canonical URL to this specific page, if this is an archival page
* `current`: The URL to the current/most recent page of the feed (typically the main URL to the feed itself)
* `next`: The next page of the feed, in the event that we are paginating
* `previous`: The previous page of the feed, in the event that we are paginating

Any changes which occur to elements which appeared on prior pages *MUST* appear on the page that is current at the time that the change took place. For example, if a piece of music that was published in January of 2020 was deleted in June of 2025, it's the page reflecting June 2025 that would contain the deletion. Similarly, updates to song metadata would occur in the feed at the time that the update happened. In this way, feed consumers do not need to re-traverse the entire backlog of a large feed to get all updates, and can incrementally update only by retrieving the current feed and any pages that haven't already been retrieved.

For this reason, past page URLs must also be stable; if the June 2025 page has a URL of e.g. `https://example.com/feed/2025-06.json`, then it must *always* be at that URL. In this way, a consumer can stop traversing a feed once it has encountered an archival feed URL that it has already processed.

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

### Payments

It is up to the player to collect information about which listeners have listened to which artists and send out payments accordingly.

One possible mechanism for this would be for the player to track a running "support balance" for each listener and artist, and periodically check to see which artists have reached a particular payment threshold and then encourage the listener to make a payment at a fair rate.

[TODO: rewrite this horrible mess for clarity!]

For one example of how this could work, every artist and listener can maintain a running balance throughout the month. Every unit of time spent listening will transfer some rate from the listener's balance to the artist's. At the end of the month, any artist whose balance exceeds a certain threshold will get that amount paid directly (zeroing out their balance), and what's left over could be divvied up among the top-performing artists such that those payments exceed the threshold but then give those artists a negative balance.

For example, if the payment threshold is $5, and there's $12 left in the pot after all artists who earned more than $5 are paid, then the artists with the two highest remaining balances each receive $6, and their balances then go negative for the next round.

In this way, even artists who don't get a lot of listening would end up randomly getting at least some amount of money in a means similar to [Floyd-Steinberg dithering](https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering), and it will average out in the long run.

Content streamed from private libraries would not necessarily be subject to the balance accrual, or the balance for the library itself could be refunded back to the listener, possibly with statistics about which bands they listened to most from their library (so that it can be left up to the listener's discretion as to whether to send additional support to the artist they ***most definitely*** bought the music from already).
