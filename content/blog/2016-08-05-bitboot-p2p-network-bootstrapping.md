---
title: "Bitboot: P2P Network Bootstrapping"
date: 2016-08-05
lastmod: 2025-03-28
summary: "A P2P network bootstrapping library with no centralized server dependency"
tags:
  - projects
---
> _TL;DR: [Bitboot](https://github.com/tintfoundation/bitboot) allows arbitrary groups of computers on the internet to find each other without any local dependencies and without setting up a central server.  It can be used to bootstrap new nodes on a p2p network or to simply find your other computer whose IP changes all the time._

I've been working with a few custom [peer-to-peer](https://en.wikipedia.org/wiki/Peer-to-peer) (p2p) networks lately, and one issue that I kept running into was figuring out how to bootstrap the networks.  If only a few nodes are out there running, and their IP addresses may change without warning, how could I reliably find those nodes when starting up a local node?

On most "distributed" p2p networks, clients are initially bootstrapped by connecting to a handful of hard-coded, centralized nodes (yes, even [Bitcoin](https://github.com/bitcoin/bitcoin/blob/37d83bb0a980996338d9bc9dbdbf0175eeaba9a2/src/chainparams.cpp#L116) and [BitTorrent](https://github.com/qbittorrent/qBittorrent/blob/5e114c0f2ead8077061e09e8debf89dfa0d526dc/src/base/bittorrent/session.cpp#L1567)).  Every new peer-to-peer network must solve this same challenge, usually by hardcoding centralized bootstrap servers.  I really didn't want to have to do that - especially for a small test network.

After hitting my head against a wall for a few hours, I worked out the framework for a proof of concept (with an anonymous friend) that would solve the issue by relying on an existing network - namely, the friggin' enormous [BitTorrent distributed hash table](https://en.wikipedia.org/wiki/Mainline_DHT).  The particular characteristics of a [Kademlia DHT](https://en.wikipedia.org/wiki/Kademlia) (the type of DHT BitTorrent uses) allow you to create nodes and with IDs and a concept of distance between those IDs.  This means that you can select certain IDs when you join the network that are near particular points - what I call "rally points."  By choosing IDs that have special characteristics (preset byte values in the ID) and close distances to the rally point, you can find other "special" nodes in the massive BitTorrent DHT rather quickly.  Once you've identified them, you can then try to connect to them on whatever port your other network runs on.  Since nodes on the BitTorrent DHT are running as full nodes, they don't harm the existing network in any way (and, in fact, they strengthen the network by adding additional storage space and redundancy).

We decided to call this particular method **bitboot** and created an implementation [on github](https://github.com/tintfoundation/bitboot).  It's in NodeJS because the [best BitTorrent DHT library we could find](https://www.npmjs.com/package/bittorrent-dht) was also in NodeJS.  When you run bitboot, you give it a [magic name](https://en.wikipedia.org/wiki/Magic_number_(programming)) to uniquely identify the network you'd like to join.  Bitboot then joins the existing [BitTorrent DHT](https://en.wikipedia.org/wiki/Mainline_DHT) and finds other nodes with the same magic name.  It does this by selecting a rally point to hang out near based on the magic name where it will meet other nodes with the same magic name value.  Also, the ID it uses is carefully selected so other nodes can pick it out as a bitboot peer based on the value of the magic name (in case other non-member nodes are hanging out around the rally point).

There's a command line program to give it a try - check it out at [github.com/tintfoundation/bitboot](https://github.com/tintfoundation/bitboot).
