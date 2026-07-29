# Collection pagination

Some collections (such as for a record label or a private storage server) will be much too large for all data to be provided in a single view, and so there must be a means of breaking it up into chunks that can be incrementally retrieved.

This extension is analogous to [RFC 5005](https://datatracker.ietf.org/doc/html/rfc5005), and provides a mechanism for append-only updates.

## Protocol

In order to facilitate this, the `collection`'s `links` **MAY** contain the following link `rel`s:

* `self`: The canonical URL to this specific page, if this is an archival page
* `current`: The URL to the current/most recent page of the collection (typically the main URL to the collection itself); **REQUIRED** if this is not the current page
* `next`: The next page of the collection, in the event that we are paginating
* `previous`: The previous page of the collection, in the event that we are paginating
* `full`: A URL that contains the full content of the collection, if that's feasible/reasonable

Any changes which occur to entities which appeared on prior pages **MUST** appear on the page that is current at the time that the change took place, and **SHOULD** appear on the original page as well. For example, if a piece of music that was published in January of 2020 was deleted in June of 2025, it's the page reflecting June 2025 that would contain the deletion. Similarly, updates to song metadata would occur in the collection at the time that the update happened. In this way, collection consumers do not need to re-traverse the entire backlog of a large collection to get all updates, and can incrementally update only by retrieving the current page and any pages that haven't already been retrieved.

For this reason, past page URLs should also be stable; if the June 2025 page has a URL of e.g. `https://example.com/Chorus/2025-06.json`, then it should always be at that URL so that a consumer can stop traversing pages once it has encountered an archival URL that it has already processed per HTTP versioning headers (`If-Modified-Since`, `If-None-Match`, etc.).

Note that different pages of a Chorus collection are considered to be separate documents, for the purpose of [entity references](#entity-reference). However, entity identifiers **MUST** be consistent across pages.

## Implementation notes

When updating a collection, a receiver should find the oldest page that has changed, and then update incrementally from there. A receiver **SHOULD** make use of HTTP's standard conditional request (`If-None-Match`/`If-Modified-Since`) mechanism to determine whether a page has been updated. For a detailed reference, see [MDN's article on HTTP conditional requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Conditional_requests).

