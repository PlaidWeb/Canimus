# Receivers

This document describes some of the considerations in place for implementing a receiver. It is meant to be descriptive, not prescriptive.

## Feed subscriptions

Generally-speaking, the player should periodically refresh the feeds that it has knowledge of at some regular interval, as well as supporting [WebSub](https://en.wikipedia.org/wiki/WebSub) for immediate "pings" when an update takes place.

If a feed or its content disappears in some way other than being explicitly deleted, it should be up to the player to handle this in a graceful manner. For example, content which has just gone offline should be listed as "temporarily unavailable," and then after a certain period of unavailability it could be removed.

The current page of the feed should always be fully processed. Any archival URLs should be retrieved if they were not already processed. This includes both `next` and `previous`.

## Collisions

In order to prevent things like malicious takeovers of band names, as well as the very common case of multiple bands having the same name, artists should be namespaced to the URL of the feed that populates the data.

Multiple feeds may represent the same entities, which can be verified by both sources providing links with a `rel` of `this` which indicate this endorsement; for example, if feed A and feed B refer to each other with such a link, it can be assumed that their content may be merged. Additionally, any `rel` of `canonical` should be used to indicate which one "wins" in the case of conflicts, or should be used as the source of ground truth.

## Private libraries

One of the ways that the major commercial streaming platforms acquired their early mindshare with the listening public was to allow listeners to upload their own private purchased libraries, enabling them to bring their purchased collection with them everywhere. This functionality should also be made available in a receiver.

Rather than requiring that people upload their (often gigantic) collections to a server somewhere and the copyright nightmares that would entail, one possible mechanism would be for the player engines to allow people to self-host their own collection, possibly using a service that they run from their home network (á la Plex).

This is an implementation detail of the receiver, but one basic suggestion would be for users to be able to run a simple media server that presents a Canimus feed to their collection and requires some simple authentication path between this collection and the receiver, e.g. an API token or HTTP basic auth credentials.

The music listened to on these private libraries should lead to public recommendation and discovery data, but the private library content itself should not be made accessible to other users of the same receiver or the network in general. Individual users *may* be able to invite others to connect to their collection, but the burden is then on them to deal with whatever copyright-related issues this presents.

## Discovery

Listeners should be able to provide a profile of the music they are listening to, similar to so-called "scrobblers." This should be an opt-in mechanism. A listening feed should contain the following information at the very least:

* Artist, title, and associated album metadata of the song
* If available, the canonical HTML informational URL of the song and album, and the public `media` URL(s)
* If available, the validated MusicBrainz identifier of the song
* The listener's disposition (played, skipped, partial, etc.) and any ratings information that the listener provided
* If part of a public playlist, the playlist URL

This information could be used to build a per-player recommendation database, and possibly allow listeners to subscribe to each other as a form of peer-to-peer discovery and recommendation.

The most logical means of formatting the feed is as a Canimus feed with a `scrobble` entity (which would in turn build on `playlist`), but it could also be provided in other formats for compatibility with other services (Libre.fm, ListenBrainz, etc.)
