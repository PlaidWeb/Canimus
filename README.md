# Syncopub

A living standard for publishing and synchronizing music collection data for music streaming platforms

## Rationale

## Scope

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

* note that UIDs need to be namespaced to the feed in some way to prevent malicious overwrites
* note the hierarchical definition of a collection (i.e. top level could be an artist, or an album, or even a track, or an array of such)

### Artist

### Album

### Track

* allow for multiple content-types (`audio/mp3`, `audio/ogg-vorbis`, etc.)

### Playlist

### Deletion

### Related links

#### Pagination



todo:

* define scope and rationale
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
*