# Receivers

This document describes some of the considerations in place for implementing a receiver. It is meant to be descriptive, not prescriptive.

## Feed subscriptions

Generally-speaking, the player should periodically refresh the feeds that it has knowledge of at some regular interval, as well as supporting [WebSub](https://en.wikipedia.org/wiki/WebSub) for immediate "pings" when an update takes place.

If a feed or its content disappears in some way other than being explicitly deleted, it should be up to the player to handle this in a graceful manner. For example, content which has just gone offline should be listed as "temporarily unavailable," and then after a certain period of unavailability it could be removed.

The current page of the feed should always be fully processed. Any archival URLs should be retrieved if they were not already processed. This includes both `next` and `previous`.

### [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/User-Agent)

It's a good idea to declare an appropriate `User-Agent` header that follows the [accepted practices for crawlers and bots](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/User-Agent#crawler_and_bot_ua_strings); namely, you should declare your `User-Agent` to be something like:

    ExamplePlayer/1.2; +https://example.com/player/about.html

This allows site operators to know more about your player application and to be able to contact you in the event that something's going wrong with your update mechanism.

### [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Accept)

The HTTP `Accept` header declares which feed protocols you accept and prefer, and it's always a very good idea to declare it for the best match between client and server.

A good starting point header value, for clients which only can parse the JSON feed format, might be:

    Accept: application/canimus+json,application/json;q=0.9, */*;q=0.5

If you can also parse YAML, then you would do:

    Accept: application/canimus+json, application/json, application/yaml, */*;q=0.5

And so on.

This allows the server to select the appropriate protocol to send back, in the event that it supports multiple protocols. Notably, this also allows implementations to provide a human-readable HTML fallback if no particular content-type is requested, which is a better user experience in general.

### Caching and avoiding over-updating

Like with any client that polls for updates, the receiver should support the standard HTTP [`If-Modified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/If-Modified-Since) and [`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/If-None-Match) mechanisms to ensure that it is only downloading the feed if something has changed.

The short version: Any time you poll the feed, keep track of the values of the `ETag` and `Last-Modified` response headers, and if you have them, add them to the request's `If-None-Match` and `If-Modified-Since` headers, respectively, and if you get a status code of 304, the feed hasn't updated since the last poll.

### Reference implementations

Here is Python stub code for how to implement all of the above, using [requests](https://requests.readthedocs.io/):

```python
def update_feed(feed_url, last_response_headers):
    headers = {
        'User-Agent': 'ExamplePlayer/1.2; +https://example.com/player/about.html',
        'Accept': 'application/canimus+json, application/json;q=0.9, */*;q=0.5'
    }
    if last_response_headers and 'ETag' in last_response_headers:
        headers['If-None-Match'] = last_response_headers['ETag']
    if last_response_headers 'Last-Modified' in last_response_headers:
        headers['If-Modified-Since'] = last_response_headers['Last-Modified']

    r = requests.get(feed_url, headers=headers)

    if r.status == 304: # Not Modified
        return

    # otherwise, process the feed. Remember to store r.headers for later!
```

And here is a similar implementation in Javascript/node:

```javascript
async function updateFeed(feedUrl, lastResponseHeaders) {
    const request = await fetch(feedUrl, {
        headers: {
            'User-Agent': 'ExamplePlayer/1.2; +https://example.com/player/about.html',
            'Accept': 'application/canimus+json, application/json;q=0.9, */*;q=0.5'
            'If-None-Match': lastResponseHeaders['Last-Modified'],
            'If-Modified-Since': lastResponseHeaders['ETag']
        }
    });

    if (request.status == 304) {
        return; // not modified
    }

    // process the feed; remember to store request.headers for later!
}
```

Adapt this to your language and runtime as necessary.

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

One possible means of formatting the feed is as a Canimus feed with a `scrobble` entity (which would in turn build on `playlist`), but it could also be provided in other formats for compatibility with other services (Libre.fm, ListenBrainz, etc.)
