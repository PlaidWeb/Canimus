# Canimus

The Canimus feed format is a syndication format to allow for federated discovery of streamable music in a platform-agnostic manner.

Canimus is the Latin first-person plural progressive form of [canō](https://en.wiktionary.org/wiki/cano#Latin), which has several meanings:

* We sing
* We play
* We make sound sound
* We chant
* We resound
* We prophesize

Or, in short, *we make music*.

## Rationale

Music discovery, consumption, and streaming is locked down by large corporations that have made a mess of things. The streaming providers have to cater to the whims of the major record labels, and have created a two-tier system where neither discovery nor payments are even remotely fair. Independent musicians are hit hardes by this, as they have to pay money to play a game that is stacked against them, and what little listenership they get is largely divvied up to the major labels. The specific payment strategy that's used has also led to a glut of minimum-effort content and AI slop and bot-driven streaming that tries to game the system, hurting everyone, big and small alike.

There are several disparate attempts to build a better world for musicians, but many of them are built on protocols that were not designed for this use case in mind. ActivityPub and RSS were simply not designed with the specific needs of distribution and discovery of musical content, and most of the existing attempts are built on top of those.

Canimus (Latin for "We Sing") is an attempt to build a lightweight syndication protocol that anyone can join in on, and which provides the much-needed structure for subscribing to musicians' streaming content in a way that enables fair payments, while also being Web-native.

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

