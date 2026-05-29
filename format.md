# Canimus format

A Canimus collection is formatted as structured data, provided in a commonly-parseable format that provides nested key-value pairs and arrays of data. Every hierarchical layer represents a single entity, which may contain other entities.

A case can be made for embedding the data directly into webpages using [microformats](https://microformats.org/). However, quite a few of the existing systems for music distribution are implemented as single-page applications and do not offer server-side rendering, and for those existing systems it would be much easier for them to present the data via an API endpoint, for which there is no meaningful advantage to HTML+mf2 or the like. For this reason, it presents a much lower-friction path to having a structured feed format as the primary interchange format.

JSON is likely the simplest to implement and to build validation tools for, as most current web frameworks and languages already have direct first-class support for JSON. However, other formats such as XML and YAML are also plausible and should be considered.

For the sake of this document, the assumption will be that the data is serialized in JSON format, and that there is a standard for JSON discovery from the associated webpages, such as a `<link>`, for example:

```html
<link rel="alternate" type="application/canimus+json" href="/path/to/canimus.json">
```

in the HTML `<head>`, and with a recommendation of also providing a [`Link:` HTTP response header](https://www.w3.org/wiki/LinkHeader).

## Entity definitions

These are the entities which can be defined by the collection.

All attributes are **optional** unless otherwise specified. Attribute names **should** be rendered in all-lowercase and use underscores (`_`) for the word separator. This enables the most compatibility across languages and use cases.

The following attributes apply to all types of entity:

* `type`: The type of entity being defined; **required**
* `name`: The common name of the entity
* `url`: The canonical URL for an HTML representation of the current entity, e.g. the webpage for the label/artist/album/track
* `uid`: An opaque, permanent string identifier to uniquely identify this entity relative to this collection; this defaults to `url` if not specified

    At least one of `url` or `uid` is **required**

* `release_date`: The original release date, in `YYYY-MM-DD` format; **strongly recommended**
* `updated_date`: The date of the last update, in `YYYY-MM-DD` format

    For both of these, partial dates are acceptable in the event that only the year or year and month are available.

    For consistency, this should be serialized as a string even if the serialization format supports datetimes (e.g. YAML).

* `images`: A collection of images that are relevant to the display of the content. This is to be stored as a key-value dictionary, where the key is the type of image, and the value describes the image.

    Possible keys include (but are not limited to):

    * `thumb`: A representative icon for the item (such as a logo)
    * `main`: Primary artwork to be displayed in a player (primarily relevant to an album or track, but can also be used as a band fallback for things without artwork, for example)
    * `photo`: A larger photographic image representing the item (headshots, profile images, etc.)

    Each of these keys maps to a dictionary of properties, or an array of dictionaries, each of which include:

    * `src`: The URL to retrieve the image from; **required**
    * `alt`: The accessibility alt-text of the image; **strongly recommended**
    * `width` and `height`: The nominal display sizes of the image
    * `type`: The MIME content type of the image (e.g. `image/png`, `image/jpeg`, `image/webp`); **strongly recommended**
    * `alternates`: An array of alternate formats, each a dictionary of properties the same as above (excepting `alternates`)

* `summary`: A short description of the entity, intended to be one single line of plain text.
* `description`: A longer description of the entity, formatted as HTML, to be sanitized by the consumer.

* `links`: Associated links; stored as an array of property dictionaries, each of which includes the following attributes:
    * `name`: The display name of the link; **required**
    * `href`: The target of the link; **required**
    * `type`: The content-type of the link
    * `rel`: The relationship of this link to the item. These include, but are not limited to:
        * `canonical`: The URL that is considered the canonical representation of this entity on the web
        * `this`: An alternate URL that is also trusted to represent this entity
        * `alternate`: A URL that represents an alternate version of this entity
        * `support`: Indicates that this URL is where a listener may provide financial support to the artist
        * `purchase`: Indicates that this URL is where a listener may obtain a copy of this content
        * `video`: A place to see a music video for this content
        * `icon`: A small image to represent the link, formatted the same way as it would be in `images`
        * `license`: A full description of the license terms for the item

        Note that more link relationships may be added in the future as additional needs are identified; as such, a link with an unknown `rel` should be either ignored or collected as an "other" type.

* `children`: A list of entities that are contained by this entity.specified.

### <span id="entity-reference">Entity references</span>

Some property types may be a simple display string, or they may be a reference to another entity. In the case of an entity reference, it will be a property bag containing the following properties (all optional unless otherwise specified):

* `uid`: The `uid` of the referenced entity
* `name`: The display name within the context of the entity, e.g. to override the display name
* `rel`: The relationship of this entity
* `label`: A text label describing the relationship

If the entity reference is given as a basic string, it will be interpreted as a `name` that overrides the natural default entity's.

If a `uid` is given, a matching entity **must** appear in the same Canimus document.

While unlikely, it is theoretically possible for a circular reference to occur. Implementations will need to account for this, although the mechanism by which they do that is outside the scope of this document.

## Top level (root)

The top-level entity should have a type of `root`. A `root` entity cannot be contained by other entities.

An older version of the specification called this `feed` and that should be accepted but considered deprecated.

The root entity can contain the following additional attributes:

* `protocol`: Refers to the protocol of the file, i.e. `"Canimus"`
* `version`: Refers to the base Canimus specification version in effect; this can be given as a tag or a commit hash within the [main Canimus repository](https://github.com/PlaidWeb/Canimus), e.g. `"v0.1.0"` or `"411bcce"`.

* `deleted`: Items that have been explicitly removed from the collection, given as a list of property dictionaries which can uniquely identify the content; for example, it **must** contain at least one of `url` and `uid`, and **should** contain other identifying information such as `name`. Deleted items must not appear anywhere else in the collection.

A collection supports the following additional link types, with the `rel` value set accordingly:

* `websub`: A link to a [WebSub](https://en.wikipedia.org/wiki/WebSub) hub, where a receiver can subscribe to immediate updates to this collection
* Links for pagination, as described in the following "Pagination" subsection.

All entity types are valid `children` aside from `root`.

### Pagination

Some collections (such as for a record label or a private storage server) will be much too large for all data to be provided in a single view, and so there must be a means of breaking it up into chunks that can be incrementally retrieved. In order to facilitate this, the collection's `related` links may contain the following link types:

* `self`: The canonical URL to this specific page, if this is an archival page
* `current`: The URL to the current/most recent page of the collection (typically the main URL to the collection itself); **required** if this is not the current page
* `next`: The next page of the collection, in the event that we are paginating
* `previous`: The previous page of the collection, in the event that we are paginating
* `all`: A URL that contains the full content of the collection, if that's feasible/reasonable

Any changes which occur to elements which appeared on prior pages *MUST* appear on the page that is current at the time that the change took place. For example, if a piece of music that was published in January of 2020 was deleted in June of 2025, it's the page reflecting June 2025 that would contain the deletion. Similarly, updates to song metadata would occur in the collection at the time that the update happened. In this way, collection consumers do not need to re-traverse the entire backlog of a large collection to get all updates, and can incrementally update only by retrieving the current page and any pages that haven't already been retrieved.

For this reason, past page URLs must also be stable; if the June 2025 page has a URL of e.g. `https://example.com/canimus/2025-06.json`, then it must *always* be at that URL. In this way, a consumer can stop traversing pages once it has encountered an archival URL that it has already processed.

Note that different pages of a Canimus feed are considered to be separate documents, for the purpose of [entity references](#entity-reference).

## <span id="artist">Artist</span>

An entity of type `artist` is a performing artist. The `name` attribute refers to the primary name under which the artist releases.

Valid `children` types:

* [`album`](#album)
* [`track`](#track)

## <span id="album">Album</span>

An entity of type `album` is a collection of tracks. The `name` attribute refers to the title of the album. It contains the following additional properties:

* `artist`: An [entity reference](#entity-reference) to the releasing artist of this album (also known as "album artist"). It defaults to the `name` and `uid` of the containing `artist`, if any.
* `subtitle`: The subtitle of the album
* `copyright`: The copyright information of the album
* `license`: Any additional license information, e.g. `"CC by-nc-sa"`
* `genre`: An arbitrary string of text that may indicate vaguely what sorts of people might like the music on this album
* `featuring`: An array of additional featured artists as [entity references](#entity-reference), to indicate collaborations

Note that an `album` does not necessarily have to be contained by an `artist` node.

Valid `children` types:

* [`track`](#track)

## <span id="track">Track</span>

An entity of type `track` refers to a playable track. If it is contained by an `album`, then it is given a playback order based on its position in the album's `children`; otherwise it is assumed to be a single.

It has the following additional properties:

* `subtitle`: The subtitle of the track, if any
* `artist`: An [entity reference](#entity-reference) to the releasing artist of this track, which may differ from the album artist; if not specified it should default to the `artist` of the containing `album` or `artist`.
* `featuring`: An array of additional featured artists as [entity references](#entity-reference), to indicate collaborations
* `album`: An [entity reference](#entity-reference) to the album on which this track appears, if any; this is normally implied by the containing `album`, and in the case of a single release, may be blank.
* `composer`: The composer(s) of the track's music
* `lyricist`: The author(s) of the track's lyrics
* `original_artist`: The original performing artist, if this song is a cover
* `duration`: The canonical length of the track, in seconds
* `media`: A set of descriptors providing streamable versions of the audio. This **should** at the very least contain an `audio/mp3` and/or `audio/ogg` version for maximum compatibility, but may also contain other versions. Each descriptor contains the following properties:

    * `type`: The content-type of the media (e.g. `audio/mp3`, `audio/flac`, `video/mp4`, `application/x-mpegURL`, etc.); **required**
    * `src`: The URL at which the media can be played; **required**
    * `size`: The size of the content file, in bytes; **strongly recommended**
    * `duration`: The duration of the media, in seconds, if it differs from the canonical length; **strongly recommended**
    * `label`: The display label of the media, if it differs from the default musical experience (for example, "radio edit" or "music video" or the like)

    There can be multiple media with the same type, differentiated by `size` to indicate different quality levels/bitrates, so that player applications can choose the appropriate quality level based on bandwidth availability.

* `disc`: The physical disc that the track appeared on, in the case of a multi-disc album
* `track`: The physical track number for the track on its disc

* `copyright`: The copyright information of the album (defaults to the containing `album`'s if unspecified)
* `license`: Any additional license information, e.g. `"CC by-nc-sa"` (defaults to the containing `album`'s if unspecified)

* `lyrics`: The lyrics of the song, if any; this should be provided as plain text with a single `\n` between lines, and `\n\n` between verses. Limited Markdown (such as `*emphasis*` and `**boldface**`) may be supported at the discretion of the consumer.
* `genre`: An arbitrary string of text that may indicate vaguely what sorts of people might like this track (defaults to the containing `album`'s if unspecified)

Note that `disc` and `track` are purely for display purposes, and do not affect the natural playback order of the track, which is given by the order of the `track` elements within the containing entity's `children`.

An example track might look like:

```json
{
    "type": "track",
    "artist": "The Example Band",
    "featuring": ["Another Band",
      {
        "name": "Yet another band",
        "uid": "asdf-12345"
      }],
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
        "type": "audio/flac",
        "src": "https://cdn.example.com/artist/album/01 the introductory track.flac",
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

## <span id="playlist">Playlist</span>

==NOTE:== This section of the specification is especially rough and likely to change.

A curated list of music to listen to, including tracks and albums. This can be useful for an artist to publish a "best of" or a mixtape or the like. It has the following additional properties:

* `author`: The author of the playlist

The `children` of this must include at least as much information as is necessary to recreate the entity assuming that this entity is the only data available. For example, for a `track` that is contained within the `catalog`, only `url` or `uid` are strictly necessary, but for music stored in other collections it must contain the full metadata from the other collection.

For example:

```json
{
    "type": "playlist",
    "author": "Example Curator",
    "uid": "20a481cc-a340-4baf-8d89-d0973e3ec4cc",
    "children": [{
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

Valid `children` types:

* [`artist`](#artist)
* [`album`](#album)
* [`track`](#track)

## <span id="events">Events</span>

==NOTE:== This section of the specification is especially rough and likely to change.

An `events` entity represents a time-based event feed to indicate interaction events. It is similar to a `playlist` but is meant to be ephemeral in nature.

The intention is that an individual user could publish a Canimus collection containing an `events` element as part of a greater recommendation system (with users subscribing to each others' feeds), and be similar to a "scrobbling" system such as [Last.fm](https://last.fm/), [Libre.fm](https://libre.fm/), or [ListenBrainz](https://listenbrainz.org/). However, its inclusion is only tentative and it is probably better for social elements to be in their own purpose-specific feed format that is expressed by e.g. ActivityPub, Atom, RSS, or similar.

It provides the following properties:

* `author`: The author/originator of the event list

Valid `children` types:

* [`event`](#event)

## <span id="event">Event</span>

==NOTE:== This section of the specification is especially rough and likely to change.

An `event` entity represents an individual interaction event. It contains the following properties:

* `when`: When the item was added to the feed (i.e. when the event took place); this should be a date in a commonly-parseable format that is also timezone-aware
* `disposition`: What the listener did with the item; this can be one of the following:
    * `play`: Indicates that this was an item played to completion
    * `skip`: Indicates that this item was skipped over, possibly after being partially played
    * `like`: Indicates that this item was enjoyed
    * `dislike`: Indicates that this item was not enjoyed
* `comment`: Any comment left by the listener, expressing why they liked or disliked the item
* `item`: Either the `uid` of the referenced item, or the item itself (see [playlist items](#playlist))
