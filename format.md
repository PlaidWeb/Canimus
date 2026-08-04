# Chorus format

A Chorus collection is formatted as structured data, provided in a commonly-parseable format that provides nested key-value pairs and arrays of data. Every hierarchical layer represents a single entity, which may contain other entities.

JSON is likely the simplest to implement and to build validation tools for, as most current web frameworks and languages already have direct first-class support for JSON. However, other formats such as XML are also plausible and should be considered. The document **MUST** be encoded as UTF-8, unless the serialization format has a means of specifying an alternate encoding.

For the sake of this specification, the assumption will be that the data is serialized in JSON format.

> The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/info/rfc2119/).

## <span id="version">Version</span>

The current version of the Chorus specification is [`0.3.1`](https://github.com/PlaidWeb/Chorus/releases/tag/v0.3.1).

The version number follows the principles of [semantic versioning](https://semver.org/), namely, given a version number of `MAJOR.MINOR.PATCH`,

* `MAJOR` increments when a change happens that is not backwards-compatible
* `MINOR` increments when new functionality is introduced in a backwards-compatible manner
* `PATCH` increments when existing functionality is fixed, refined, or clarified

## Discovery

In order for a Chorus document to be discoverable from a web resource, it **SHOULD** be advertised in the form of a relevant HTTP link.

From an HTML or XML document this will most likely be a `<link>` tag, for example:

```html
<link rel="alternate" type="application/Chorus+json" href="/path/to/Chorus.json">
```

in the referring document's `<head>`.

It is also recommended to provide a [`Link:` HTTP response header](https://www.w3.org/wiki/LinkHeader), and for receivers to honor that response header in the event that `<link>` is not available or relevant.

## Style guide

### Attribute names

All attributes are **OPTIONAL** unless otherwise specified. Standard attribute names **MUST** be defined as appearing in `camelCase`.

Attributes starting with a `$` refer to things that are structural to the document, while attributes without this prefix are descriptive of the item itself.

Structural attribute names **MUST NOT** be reused by item attribute names; for example, an item **MUST NOT** define an attribute named `type`.

Attribute names are to be given in English and written in `camelCase` (first letter of the first word lowercase, no separator between words, additional words capitalized). Embedded acronyms are treated as single words; so for example a theoretical attribute of "HTML AJAX Endpoint" would appear as `htmlAjaxEndpoint`.

Attributes with a name of `$comment` are allowed anywhere for documentation purposes; attributes with this name **MUST** be ignored by receivers and **MUST NOT** be used in any future revisions to the specification.

### Forward compatibility

As attributes may be added to the specification in the future, any unknown attribute **MUST** be discarded/ignored by any receivers, and validators **MUST NOT** fail validation based on unknown attributes for a document that are written to a newer version of the specification than the validator. However, validators **MAY** issue a compatibility warning for unknown attributes.

This concern also applies to semantic relationships, such as the `rel` of a link or a marker.

## Data type definitions

### <span id="document">Document</span>

A document is represented by a serialized [entity](#entity), typically in JSON format.

The root entity will typically be of type [`collection`](#collection).

### <span id="item">Item</span>

An "item" is a collection of key-value pairs ("attributes"). It corresponds to the following data types in various languages:

* JavaScript/JSON: `Object`
* Python: `dict`
* PHP: `array` (with named keys)
* Perl: `hash`

#### <span id="localization">Localization</span>

Localization follows the [IETF BCP 47](https://www.rfc-editor.org/info/bcp47) standard.

Locale codes are defined by [RFC 5646](https://datatracker.ietf.org/doc/rfc5646/) (e.g. `en` for English, `en-US` for specifically US English). The lists of language and region codes are given by [ISO-639-1](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes) and [ISO-3166-1 alpha-2](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes), respectively.

Alternate localizations are given by appending `$code` to the attribute name; for example:

```json
{
    "$lang": "en-US",
    "name": "This is my name",
    "name$es": "Este es mi nombre",
    "name$jp": "これが私の名前です"
}
```

Even if an attribute is fully localized, it **MUST** still provide a version without a locale suffix, as localization is considered optional.

Note that any descriptive (i.e. non-structrual) attribute may be localized, which also allows for multiple language and region support for images, media renditions, and so on. Structural attributes (ones that start with a `$`) **MUST NOT** be localized.

The lookup algorithm is defined by [RFC 4647](https://datatracker.ietf.org/doc/rfc4647/). Namely, localized strings must be looked up based on exact matches, from most specific to least; for example, if the attribute `name` is requested in locale `en-US`, then the attribute should be looked up as `name$en-US`, `name$en`, and then finally `name`. A locale of `en-US` shall never receive a string for `en-UK`.

For example, with the following strings:

```json
{
    "summary": "Default",
    "summary$en-UK": "Colour",
    "summary$en-US": "Color",
}
```

a lookup of the attribute `summary` in locale `en-AU` will return `"Default"`.

Sample implementations for attribute lookup are as below.

```python
# Python implementation
def get_attribute_localized(item:dict, attribute:str, locale:str=None):
    if locale:
        tags = locale.split('-')
        while tags:
            key = f"{attribute}${'-'.join(tags)}"
            if key in item:
                return item[key]
            tags.pop()
    return item.get(attribute)
```

```js
// JavaScript implementation
function getAttributeLocalized(item, attribute, locale) {
    if (locale) {
        var tags = locale.split('-')
        while (tags.length) {
            const key = `${attribute}\$${tags.join('-')}`
            if (item[key]) {
                return item[key];
            }
            tags.pop();
        }
    }
    return item[attribute]
}
```

### <span id="uid">Identifier</span>

An identifier uniquely and permanently refers to an entity within a collection. It is a textual string, and may include any printable character. It may or may not be human-readable, but the comparison between identifiers **MUST** be based on an exact match.

Identifier names **MUST** be limited to URI-safe characters: `[A-Za-z0-9:/?#\[\]@!$'()*+,;=._~%-]`

Identifiers **MUST NOT** change due to changes in the underlying entity's attributes. For that reason it is **RECOMMENDED** that an identifier be generated and permanently associated with an entity at the time of its creation. [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)s are a good choice in general, UUID-4 in particular.

### <span id="entity">Entity</span>

An "entity" is an [item](#item) that represents a concrete object in the collection.

All entities support the following attributes:

* `$type`: The type of entity being defined; **REQUIRED**
* `$id`: An opaque, permanent string [identifier](#uid) to uniquely identify this entity relative to this collection; **REQUIRED** (*except* on `collection` itself)
* `$items`: A list of items that are referenced by this entity; an item may be another entity, or an [entity reference](#entity-reference)
* `$lang`: The default [localization](#localization) for display strings; defaults to the `$lang` of the containing entity

    It is **STRONGLY RECOMMENDED** that entities provide a `$lang`, so that localization-aware clients will know what the default localization refers to. This is useful for things such as automatic translation or displaying metadata about the item's language of origin.

    Because `$lang` is inherited from the containing entity, it is appropriate to set a collection-wide default by applying it only to the top-level entity.

* `url`: The canonical [URL](#url) for an HTML representation of the current entity, e.g. the webpage for the label/artist/release/track
* `name`: The common name of the entity
* `lastModified`: The last-modified time of this entity, as a [datetime](#datetime)

* `images`: A collection of images that are relevant to the display of this entity. This is to be stored as a key-value dictionary, where the key is the type of image, and the value is an array of image descriptors.

    Possible keys include (but are not limited to):

    * `thumb`: A representative icon for the item (such as a logo)
    * `main`: Primary artwork to be displayed in a player (primarily relevant to a release or track, but can also be used as a band fallback for things without artwork, for example)
    * `poster`: A larger photographic image representing the item (headshots, profile images, etc.)

    Each of the image descriptors is an [item](#item) with the following attributes:

    * `src`: The [URL](#url) to retrieve the image from; **REQUIRED**
    * `alt`: The accessibility alt-text of the image; **STRONGLY RECOMMENDED**
    * `width` and `height`: The nominal display sizes of the image; **STRONGLY RECOMMENDED**
    * `contentType`: The MIME content type of the image (e.g. `image/png`, `image/jpeg`, `image/webp`); **STRONGLY RECOMMENDED**

    If there are multiple descriptors available, the client is free to select the one that is the closest fit for its own display purposes (for example, selecting the most appropriate resolution or aspect ratio).

* `summary`: A short description of the entity, intended to be one single line of plain text
* `related`: A list of entities which should be seen as related to this entity (for example, associated artists). These **SHOULD** include a `relationship` label.
* `relationship`: A brief explanation of how this entity is related to its containing entity

    For example:

    ```json
    {
        "$id": "artist-fwiffo",
        "$type": "artist",
        "name": "Fwiffo the Great",
        "related": [
            {
                "$id": "artist-zorniwoop",
                "name": "Zorniwoop the Lesser",
                "relationship": "Former name"
            }, {
                "$ref": "artist-orangetheory",
                "relationship": "Our old lead singer's new band"
            }
        ]
    }
    ```

* `tags`: An array of items which contain descriptive tags for categorizing this entity, primarily to aid in content discovery and filtering. Each tag item has the following properties:

    * `name`: The display name of the tag; **REQUIRED**
    * `rel`: The relationship of the tag; tag relationships include, but are not limited to:
        * `genre`: Refers to a musical genre
        * `location`: Refers to the locality in which the entity was recorded or operates
        * `topic`: Refers to what the entity is about
        * `instrument`: Refers to a featured instrument in the arrangemenet
        * `mood`: A conscious state of mind or emotional state being conveyed

        Note that more tag relationships may be added in the future as additional needs are identified, and may be defined arbitrarily by the publisher; as such, a tag with an unknown `rel` should be collected as an "other" type.


* `links`: Associated links; stored as an array of property dictionaries, each of which includes the following attributes:
    * `name`: The display name of the link; **REQUIRED**
    * `href`: The [URL](#url) target of the link; **REQUIRED**
    * `contentType`: The content-type of the link (e.g. `text/html`, `application/rss+xml`, etc.)
    * `rel`: The relationship of this link to the item. These include, but are not limited to:
        * `this`: An alternate URL that is also trusted to represent this entity
        * `alternate`: A URL that represents an alternate version of this entity
        * `support`: Indicates that this URL is where a listener may provide financial support to the artist
        * `purchase`: Indicates that this URL is where a listener may obtain a copy of this content
        * `video`: A place to see a music video for this content
        * `license`: A full description of the license terms for the item

        Note that more link relationships may be added in the future as additional needs are identified; as such, a link with an unknown `rel` should be either ignored or collected as an "other" type.

### <span id="entity-reference">Entity reference</span>

Some [entities](#entity) need to appear multiple times in a collection. For example, artists with their own discographies may also appear in one or more tracks on compilation releases, or may be featured artists on another artist's releases. Similarly, one track may appear in multiple places, such as a label's compilation or in a playlist.

An entity reference is an item with a `$ref` that matches the `$id` of the original entity. It may also have the following additional attributes:

* `name`: The display name in the referenced context
* `relationship`: An explanation of the reference relationship

If a `$ref` appears, its corresponding `$id` ***MUST*** appear in the same Chorus document.

An entity reference is considered to have the same `$type` as the referenced entity, and should be validated accordingly.

A basic example follows:

```json
{
    "$type": "collection",
    "$items": [
        {
            "$comment": "This is an artist being defined directly",
            "$type": "artist",
            "$id": "artist-001",
            "name": "Artist Number 1",
            "url": "https://example.com/example-artist-1",
            "$items": [{
                "$comment": "This is a one-track album with a single track, 'hit single'",
                "$type": "release",
                "$id": "debut-album",
                "name": "Debut Album",
                "$items": [{
                    "$type": "track",
                    "$id": "hit-single",
                    "name": "Hit Single",
                }]
            }, {
                "$comment": "This is a best-of collection which includes 'hit single' and a remix",
                "$type": "release",
                "$id": "best-of",
                "name": "Best-Of Collection",
                "$items": [{
                    "$comment": "This references the original version of 'hit single' but adds a subtitle",
                    "$ref": "hit-single",
                    "subtitle": "original mix"
                }, {
                    "$comment": "This is a new remix of 'hit single' for this album",
                    "$type": "track",
                    "$id": "hit-single-remix",
                    "name": "Hit Single",
                    "subtitle": "Bayside Boys Mix"
                }],
                "related": [{
                    "$comment": "This provides a link back to the original release of 'hit single'",
                    "$ref": "debut-album",
                    "relationship": "Original release"
                }]
            }]
        }
    ]
}
```

### <span id="structure">Structural relationship</span>

A Chorus document does not define a hierarchical tree that must be expanded fully; instead, it defines separate entities with a many-to-many relationship between them, and the containment structure is as a matter of convenience to the publisher in order to limit the amount of repeated information needed to express those relationships.

That is to say that an entity's `$items` *can* directly or indirectly contain an item that is a `$ref` back to itself, such as in the case of an [`artist`](#artist) containing a [`release`](#release) that contains a [`track`](#track) that has an `artist` that is a `$ref` back to the artist with an alternate display name.

For example, this document:

```json
{
    "$type": "container",
    "$items": [{
        "$type": "artist",
        "$id": "my-artist",
        "$items": [{
            "$type": "release",
            "$id": "my-album",
            "$items": [{
                "$type": "track",
                "$id": "my-track"
            }]
        }]
    }]
}
```

is semantically-equivalent to this document:

```json
{
    "$type": "container",
    "$items": [{
        "$type": "artist",
        "$id": "my-artist",
        "$items": [{
            "$ref": "my-album"
        }]
    }, {
        "$type": "release",
        "$id": "my-album",
        "$items": [{
            "$ref": "my-track"
        }]
    }, {
        "$type": "track",
        "$id": "my-track"
    }]
}
```

Both define three elements: an [`artist`](#artist) with a single [`release`](#release) which contains a single [`track`](#track). The serialized structure is different, but the meaning is the same.

### <span id="lyric-text">Lyric Text</span>

In lyrics, the following Markdown-style markup types **MAY** be supported:

* Emphasis (e.g. `*italic*`, `**bold**`)
* Monospace text (e.g. `` `i am a robot bleep blorp` ``)

It is valid for an implementation to display lyric text as the raw string.

Raw HTML tags ***MUST NOT*** be supported; in contexts where the text display is being handled by an HTML renderer (such as in a browser or embedded WebView), entities **MUST** be escaped (for example, converting the text `<hello>` to the HTML `&lt;hello&gt;`).

### <span id="datetime">Dates and times</span>

Dates and times are represented as strings in `YYYY[-MM[-DD[Thh:mm[:ss][+ZZZZ]]]]` format. For example, `2026-06-14T14:42-0700` is equivalent to June 14, 2026 at 2:42 PM in UTC-0700 (e.g. Pacific Daylight Time). This format is similar to [RFC 3339](https://www.rfc-editor.org/info/rfc3339/), but allows the date to be precise only to a month or year, as is (unfortunately) common in a lot of music history.

If a given time lacks timezone information, it will be assumed to be UTC; `14:06:02` and `14:06:02+0000` are therefore equivalent.

A consumer **SHOULD** make use of all available precision, but it is not specified how it treats partial matches between two datetimes with differing levels of precision; for example:

* It is not specified how `2026-06-14T12:34`, `2026-06-14`, `2026-06`, and `2026` sort relative to one another
* `2026-06` must always come after `2026-05` and `2026-05-30`

Per the above, dates may be trivially sorted and filtered lexically, but fully-specified datetimes need to be timezone-aware.

### <span id="duration">Durations</span> and <span id="time-offset">time offsets</span>

Durations and time offsets are given numerically as seconds, and **MUST** be serialized as a number. So, for example, a duration of 1 hour, 23 minutes, and 45.6 seconds is serialized as the number `5025.6`.

A time offset is relative to the start time of the respective media.

### <span id="url">URLs</span>

A URL is a string that references an external resource.

URLs **SHOULD** be given as absolute by publishers; however, receivers **MUST** treat all URLs as potentially-relative to the originating document.

For example, if a document is at `https://example.com/chorus.json`, then a URL of `/foo.mp3` **MUST** be interpreted as `https://example.com/foo.mp3`, and a URL of `//cdn.example.com/bar.ogg` **MUST** be interpreted as `https://cdn.example.com/bar.ogg`.

Example implementations of URL resolution in various languages:

* JavaScript (including Node): [`URL()` constructor](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL)
* Python: [`urllib.parse.urljoin`](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.urljoin)
* PHP: [`php-urljoin`](https://github.com/fluffy-critter/php-urljoin)

## Entity types

These are the types of [entities](#entity) known to the collection format.

### <span id="collection">Collection</span>

The top-level entity **SHOULD** have a type of `collection`. A `collection` entity cannot be contained by other entities.

The `collection` entity can contain the following additional attributes:

* `$protocol`: Refers to the protocol of the file, i.e. `"Chorus"`
* `$version`: Refers to the base Chorus specification [version](#version) in effect
* `$schema`: A URL to a JSON Schema reflective of the version of the protocol in use
* `$deleted`: Items that have been previously published but are now removed from the collection, given as a list of `$id` values

    These entities **MUST NOT** appear anywhere else in the document, and furthermore **SHOULD** only appear if an item was previously published but is to be revoked.

    Any [entity references](#entity-reference) that refer to the original item are also to be removed.

A collection supports the following additional link types, with the `rel` value set accordingly:

* `websub`: A link to a [WebSub](https://en.wikipedia.org/wiki/WebSub) hub, where a receiver can subscribe to immediate updates to this collection

All entity types are valid `$items` aside from `collection`.

### <span id="label">Label</span>

An entity of type `label` refers to a record label.

Valid `$items` types:

* [`release`](#release)
* [`track`](#track)
* [`artist`](#artist)

### <span id="artist">Artist</span>

An entity of type `artist` is a releasing artist. The `name` attribute refers to the primary name under which the artist releases.

Valid `$items` types:

* [`release`](#release)
* [`track`](#track)

### <span id="release">Release</span>

An entity of type `release` indicates a released item, typically an album containing one or more [`track`](#track)s. The `name` attribute refers to the title of the release. It contains the following additional properties:

* `releaseDate`: The original release date, as a [datetime](#datetime)
* `label`: The [`label`](#label) that owns/manages this release. If not specified, it uses any [`label`](#label) associated with the [`artist`](#artist).
* `artist`: The primary [`artist`](#artist) that owns/manages this release (also known as "album artist"). If not specified, it uses the [`artist`](#artist) that contains this `release`, if any.
* `subtitle`: The subtitle of the release
* `copyright`: The base copyright information of the release (e.g. `"℗2025 MecaGorp Ultd; ©2025 Jennifer Example"`)
* `license`: Additional license information, e.g. `"CC by-nc-sa"`
* `licenseUrl`: A link to the additional license information, e.g. `"https://creativecommons.org/licenses/by-nc-sa/4.0/"`
* `genre`: An arbitrary descriptive string of plain text that may indicate vaguely what sorts of people might like this release.

    Note that this is for end-user display purposes, and not for categorization and content filtering; that should be performed using the entity's `tags` properties. Both attributes serve different purposes.

    If no `genre` is specified, then a user-facing client **MAY** form a display string from the entity's `tags` with a `rel` of `genre`.

* `featuring`: An array of additional featured [`artist`](#artist)s, to indicate collaborations; these artists may also have additional properties such as:
    * `role`: The role this artist played in the release

Note that a `release` does not necessarily have to be contained by (or have) an `artist` entity. In this case, it is up to the consumer to decide how to display this.

Valid `$items` types:

* [`artist`](#artist)
* [`track`](#track)

### <span id="track">Track</span>

An entity of type `track` refers to a playable track. If it is contained by a [`release`](#release), then it is given a playback order based on its position in the release's `$items`; otherwise it may be assumed to be a single.

It is **RECOMMENDED** (but not required) that released singles be a [`release`](#release) containing a single `track`, and that any `track`s that are not contained by a [`release`](#release) still appear in the relevant [`artist`](#artist)'s discography.

Also note that standalone tracks **MUST NOT** have a [`label`](#label); to assign a label to a track it must be part of a [`release`](#release).

It has the following additional properties:

* `subtitle`: The subtitle of the track, if any
* `artist`: The primary [`artist`](#artist) that owns/manages this track. If not specified, it uses the [`artist`](#artist) of any containing [`release`](#release).
* `featuring`: An array of additional featured [`artist`](#artist)s, to indicate collaborations; these artists may also have additional properties such as:
    * `role`: The role this artist played in the track
* `composer`: The composer(s) of the track's music
* `lyricist`: The author(s) of the track's lyrics
* `originalArtist`: The original performing artist, if this song is a cover
* `duration`: The canonical length of the track, in seconds
* `discNum`: The physical disc that the track appeared on, in the case of a multi-disc album
* `trackNum`: The physical track number for the track on its disc

    Note that `discNum` and `trackNum` are purely for display purposes, and do not affect the natural playback order of the track, which is given by the order of the `track` items within the containing [`release`](#release)'s `$items`.

* `copyright`: The copyright information of the track (defaults to the containing [`release`](#release)'s)
* `license`: Any additional license information, e.g. `"CC by-nc-sa"` (defaults to the containing [`release`](#release)'s)
* `licenseUrl`: A link to the additional license information, e.g. `"https://creativecommons.org/licenses/by-nc-sa/4.0/"` (defaults to the containing [`release`](#release)'s)

* `lyrics`: The human-readable, non-synchronized lyrics of the track, if any; this should be provided as plain text with a single `\n` between lines, and `\n\n` between verses. [Limited Markdown](#lyric-text) (such as `*emphasis*` and `**boldface**`) **MAY** be supported at the discretion of the consumer.
* `synchronizedLyrics`: Synchronized lyrics, given as a list of items with the following properties:
    * `startTime`: The starting [time offset](#time-offset) of the lyric; **REQUIRED**
    * `duration`: The [duration](#duration) of the lyric, in seconds; **STRONGLY RECOMMENDED**

        Note that lyrics may overlap (such as in the case of duets or staggered multi-part vocals), so if `duration` is not specified it must be inferred by the length of the text, *not* by the start time of the next lyric.

    * `voice`: The name of the voice that is singing/stating the lyric; if provided, this **SHOULD** be human-readable, and **MUST** be consistent throughout the track
    * `text`: The representative text of the lyric, in [limited Markdown](#lyric-text); **REQUIRED**

* `genre`: An arbitrary descriptive string of plain text that may indicate vaguely what sorts of people might like this track (defaults to the containing `release`'s if unspecified).

    If no `genre` is specified, then a user-facing client **MAY** form a display string from the entity's `tags` with a `rel` of `genre`.

* `markers`: An array of marker items to indicate different sections of a track, such as movements, chapters, or other similar metadata. Each array item contains the following properties:

    * `startTime`: The [time offset](#time-offset) where the marker appears; **REQUIRED**
    * `text`: The text label of the marker; **REQUIRED**
    * `rel`: The type of marker, for example, `movement`, `section`, `chapter`, `index`, etc.

* `credits`: An array of detailed credits for the production of the track, containing the following properties:

    * `name`: The name of the person
    * `roles`: An array of production roles (e.g. vocals, instruments, production, coffee, etc.)

* `media`: A list of descriptors providing streamable/listenable renditions of the track. This **SHOULD** contain at least one descriptor entity with a `contentType` of `audio/mp3` for maximum compatibility. Each descriptor contains the following properties:

    * `contentType`: The content-type of the media (e.g. `audio/mp3`, `audio/flac`, `video/mp4`, `application/x-mpegURL`, etc.); **STRONGLY RECOMMENDED**
    * `src`: The URL at which the media can be played; **REQUIRED**
    * `size`: The size of the content file, in bytes; **STRONGLY RECOMMENDED**
    * `description`: A descriptive label for this rendition

    There can be multiple media with the same type, differentiated by `size` to indicate different quality levels/bitrates, so that player applications can choose the appropriate quality level based on bandwidth availability.

    This is not suitable for different versions of a song, however; those should be given either with `related` or `links` as appropriate. That is to say, each of these **MUST** be the same underlying recording.

An example track might look like:

```json
{
    "$type": "track",
    "$id": "13a93b29-4e4b-4967-a077-cbe8491767ec",

    "artist": {
        "$type": "artist",
        "$id": "5ee2099f-8d04-48c8-bc3d-832d3b0b58cc",
        "name": "The Example Band"
    },
    "featuring": [
        {
            "$type": "artist",
            "$id": "b96b4398-1845-43b4-89e8-76b8409f1fbf",
            "name": "Another Band"
        },{
            "$ref": "yet-another-band"
        }
    ],
    "name": "Introduction",
    "subtitle": "Radio Edit",
    "url": "https://example.com/band/releases/introduction.html",
    "duration": 45,
    "disc": 1,
    "track": 17,
    "media": [
        {
            "contentType": "audio/mp3",
            "src": "https://cdn.example.com/artist/album/01 the introductory track.mp3",
            "size": 737280
        },
        {
            "contentType": "audio/mp3",
            "src": "https://cdn.example.com/artist/album/01 the introductory track.hq.mp3",
            "size": 1105920
        },
        {
            "contentType": "audio/flac",
            "src": "https://cdn.example.com/artist/album/01 the introductory track.flac",
            "size": 1843200,
            "description": "lossless version"
        }
    ],
    "markers": [
        {
            "timestamp": 0,
            "rel": "movement",
            "text": "Adagio"
        },
        {
            "timestamp": 15,
            "rel": "movement",
            "text": "Rondo - Vivace"
        },
        {
            "timestamp": 30.7,
            "rel": "movement",
            "text": "Larghetto i risoluzione"
        }
    ],
    "links": [{
        "name": "Music video",
        "contentType": "video/mp4",
        "href": "https://cdn.example.com/artist/videos/the introductory track.mp4",
        "rel": "video"
    }]
}
```
