# Nostr, a Basic tour

Nostr is a very lightweight open protocol that "has a chance to work" (as per the project doc) as a decentralized social media platform. The protocol specs are defined in NIPs (Nostr Improvement Proposals) and can be found [here](https://github.com/fiatjaf/nostr/tree/master/nips).

The basis of the protocol is a WebSocket server (called a nostr-relay) that handles and stores a very simple data structure called an [`Event`](https://github.com/fiatjaf/nostr/blob/master/nips/01.md#events-and-signatures). It looks like the following:

```
{
  "id": <32-bytes sha256 of the the serialized event data>
  "pubkey": <32-bytes hex-encoded public key of the event creator>,
  "created_at": <unix timestamp in seconds>,
  "kind": <integer>,
  "tags": [
    ["e", <32-bytes hex of the id of another event>, <recommended relay URL>],
    ["p", <32-bytes hex of the key>, <recommended relay URL>],
    ... // other kinds of tags may be included later
  ]
  "content": <arbitrary string>,
  "sig": <64-bytes signature of the sha256 hash of the serialized event data, which is the same as the "id" field>,
}
```

Events are always signed (using Schnorr sigs) and they contain structured data that can have semantic meanings. A Schnorr type XOnlyPubkeys as defined in [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki#design) (and currently in use with Bitcoin Taproot) are used as "identities" throughout the protocol.

A nostr-client is an APP that can talk with a nostr-relay and can subscribe to any set of events using a [`Subscription Filter`](https://github.com/fiatjaf/nostr/blob/master/nips/01.md#communication-between-clients-and-relays). The filter represents all the set of nostr `Events` that a client is interested in.

There is no sign-up or account creation for a client. Clients are identified with their pubkeys. Every time a client connects to a relay, it submits its subscription filters and the relay streams the "interested events" to the client as long as they are connected.

A relay can cache the client subscriptions but it doesn't have to. Clients are supposed to handle everything at the "client-side", and relays can be dumb as a rock.

Clients don't talk to each other. But relays can. This allows relays to fetch data for a client that it doesn't have. And Clients get to subscribe to events outside of their connected relays.

This at first glance gives the image of the uselessness of Nostr as a protocol, (why not just sign and dump raw JSON and let clients figure it out?), but on a deeper look, the "dumb-server, smart client" model can be found to have some massive engineering benefits, especially in decentralized protocol designs.

This document is an outline of how these dumb servers, smart clients, and the Bitcoin network, e2e encryption can come together to solve the problem of "Decentralized Social Networks", DSNs (a buzzword I just came up with).

# The Problem Statement

If you haven't been living under the rocks for the last 2 years, you already know the current emergence and a dire cry from the market to have "Twitter alternatives". Social media platforms that don't act against the incentives of their users.

This need gave rise to alternate social media platforms like [Gab](https://gab.com/) and [Mastodon](https://mastodon.social/about). The recent announcement from the [Ex Twitter lead](https://twitter.com/jack/status/1204766078468911106?s=20) already hints that this is gonna be the next big problem to solve. So disclaimer: I don't say it's an easy problem to solve, and I don't think Nostr solves it all. But it at least "has a chance".


The core problem of creating a decentralized media platform *is not technical, it's social.*

Creating a social media (or chat app) is probably the most textbook challenge you solve as a new software dev. The core structure of the system is rather simple.
 - A database to store stuff,
 - A network interface to talk to clients.
 - Some filtering to fetch query data as fast as possible.

Of course, it's a lot more complex than this in real life. But the crux of this design remains the same for all social media designs out there.

So why can't we just build it and be done with it?

The problem is, it has to be a "decentralized" system, which can only succeed via "network effects" and emerging engineering consensus over a set of protocols by the developer ecosystem. Otherwise, we create the same problem we set out to solve.

And this is where things get messy. If you build your perfect social media today, how can you convince other devs to build on it? If devs don't build features, why would the users come? If users don't come what's the point of that media platform in the first place?

The examples of Gab and Mastodon clearly show, that just having the code open source is not enough. The building process and designing of the standards have to be open also. Otherwise one ends up with a small team of people, mostly voluntarily working on an activist project, and eventually becoming the "benevolent dictators" of that platform.

Because they have to satisfy real-life design constraints of such platforms, while providing their product at scale, they end up creating a small cohort of a team dedicated to designing the approach of the platform. This makes casual and fun applications using the platform difficult for client devs. At some point, they might as well decide to design their own little protocol, but eventually, they will reach the same roadblock. Nobody wants to voluntarily build on a platform that you designed for a specific niche.

Also storing data is costly. It takes resource, maintenance, and time for the part of the "server owner". All the people currently hosting a Mastodon instance are doing it voluntarily and users just rely on them for being nice and not shut down the instance. The age-old problem of "creative commons" appears.

So can we do better?

# An Alternate Approach, The Dumb Nostr Way

What if instead of building the perfect social media, we just build the most basic lego it needs to create such things and let developer consensus emerge in open over this basic standard unit of the puzzle.

This is what Nostr does.

To do this it takes the following approach.

Specify the smallest unit of a social data format (an `Event`), and let agreement emerge naturally among devs to build on this. This defines the core of the protocol. The bare minimum backbone of things that everyone needs to agree on, to be part of this network.

Nostr defines these protocol rules as NIPs. And mentions a set of `mandatory` NIPs. Rules that need to be implemented to talk the Nostr protocol.

On top of these `mandatory` NIPs, `optional` NIPs can be defined by anyone. Relays are free to choose their set of supported NIPs.

The `Event` data can be extended in the `tag` field by having more tag items defined by future NIPs.

An `Events` can be thought of as a generic data store. There is no restriction on the content that can be put inside it.

As odd as might seem, such a simple protocol is getting more dev attention than many of the "well designed" existing alternate social media.

The project already has many [example implementation](https://github.com/fiatjaf/tonostr#small-list-of-software-that-implement-the-nostr-protocol-somehow) and the list is growing every day.

The Telegram chat has around 400 members and growing every day.

Why? "Because it's so simple".

This simplicity allows anyone with interest to easily write a JSON streamer and start talking the protocol to any existing relay.

A real-time list of current running Nostr relays can be found [here](https://nostr-registry.netlify.app/).

There already exists two work-in-progress twitter-like apps [branle](https://branle.netlify.app/) and [NOSTR-twitter](https://github.com/arcbtc/nostr).

People are also coming up with new extra details added on top of basic NIPs almost regularly.

The simplicity of the protocol allows devs to quickly converge on the open standard, and have all the complexity on the client side. The entire app experience would be handled by clients, and relays would remain dumb data servers. This allows devs to move and iterate fast on the client apps while being compatible with any available relays.

This also adds to client compatibility. It's possible to have two different apps, but still, be able to see each other's posts. The platform can be decentralized at its core, and clients become compatible with each other via a simple storage protocol. That's the neat thing with the "dumb server, smart client" model. quick agreement on rudimentary standard, faster iteration on cool client apps.

Complexities can be customized at the client layer while interoperability is achieved at the relay layer.

# The Missing Pieces

Once we get to an agreement on what the core lego looks like. The only remaining pieces left are DOS protection, and relay runner incentivization. And thanks to Bitcoin, we can now solve these two problems in one go.

### Putting Bitcoin inside Nostr

The next big task is to stop relay spamming and monetization of storage services offered by relays.

A robust social network cannot be built if its infrastructure rests on a flimsy foundation of "voluntarism". And as we know "if a product is free, you are the product". There should be native integration of these future media platforms into the Bitcoin blockchain.

The one-stop solution to do this is to use [BDK](https://github.com/bitcoindevkit/bdk). A highly performant bitcoin wallet library, flexible enough to handle multiple kinds of bitcoin interfaces and DBs. Added with some new NIPs to define `payment request` and `payment response` Event kinds.

The payment can be either a one-time onchain transaction or a stream of LN payments between Client and Relay, for each event they publish. (Will need BDK + LDK which is under active development). Relay can set their feerate in sats/byte, they can choose to set it to 0 if they wanna be "voluntary".

This gives a good way for high-maintenance public relays to monetize their service, at the same time protecting them from DOS.

# But Wait, There's More.

A Nostr relay is just a dump of simple JSON data. Fetched via a `subscription` filter. This allows nostr to be a generic data-sharing platform between clients. With Bitcoin inside, now we are talking about Bitcoin scripts, descriptors, DLC contracts, and other Bitcoin DeFi information, shared via a social media infrastructure.

It can be even better. Another out-of-bound protocol can be devised on top of nostr to share e2e "encrypted subscriptions". This allows the possibility for *hidden* social networks of people to selectively open up their posts to particular trusted entities.

These posts can be DLC contracts, descriptors describing multisig between many parties, DLC oracle publishing only to subscribed members, and a lot more.

Because Nostr uses pubkeys for identities, each such post can be created via different pubkeys, so they don't get caught in other people's subscriptions of your events. Each pubkey can have a user given Nym. Nyms will be as common as nicknames.

A network of *web-of-trust* can be built on top of these encrypted and hidden nostr posts, where people can selectively follow other nyms that their trusted nyms are already following, and publishers can selectively open up posts to their specific network of trust.

Communication can happen via nostr text content, which is an encrypted blob of data.
The subscription for that data, along with the relay address to fetch it from, is also encrypted and shared among intended participants using any out-off-bound protocol.
A shared decryption key can be derived using DH key exchange again via some custom out-off-bound protocol among clients.

What the above describes is a privacy-focused decentralized platform to share any random data among clients, including p2p trade ads, betting with DLC, or anything else. The relays don't have any visibility on the data they are storing. And clients are smart enough to make sense of this data and act on it.

This is more than just "Social Media". It's a "Social Network" of people coordinating with each other just by agreeing on some basic protocol. `Nostr` and `Bitcoin`. Thus the name "Distributed Social Networks" (DSN).

In this model what we end up with is:
 - A Highly interoperable stupid relay protocol.
 - A flexible extension model freely deployed by clients.
 - Bitcoin native integration to facilitate the "internet of money" and DOS protection at the same time.
 - Strong privacy guaranteed via e2e encryption of subscription data and out-off-bound custom protocols.
 - A decentralized publication layer for clients to publish public as well as private content.
 - Client-side complexity to interpret these contents and have native financial contract generation UI using the powers of Bitcoin.

And unlike everything web3.0, this doesn't involve another "Blockchain" (bummer, I know).

# Path Ahead

As good as it may sound, we are not there yet. And a lot of engineering designs are required to realize these dreams. There are unknown problems ahead that need to be dealt with. The design decision of these relays and clients needs to be carefully laid out. Just having a simple protocol is not good enough.

These relays should be efficient, robust, rigorously peer-reviewed in open public, guaranteed at the basic level of security. The work has to be done in open public, and the elements should be designed to be as flexible as possible to satisfy diverse client developer needs.

If this thing needs to extend out into professional services that people can deploy in their servers, and serious products are built out of it, we need more than just hobby codes and example apps.

The path ahead lies in complex design decisions (with simple elements) and keeping in mind these whole set of ideas from the very start is important to guide what a Nostr infrastructure in rust might look like.

# Introducing rust-nostr

rust-nostr is a project at the idea stage that aims to solve the above. The idea is to provide a complete one-stop suite of nostr infrastructure, which is modular, easily extensible, has strong security guarantees, well documented, very easy for devs to customize as per their own need, very easy to just download, deploy and manage in their servers.

The entire structure is still TBD, but a rough outline of how rust-nostr will look like is following.

 - A binary crate producing `nostrd`. A lightweight and efficient rust implementation of Nostr-Relay. `nostrd` will come with a set of supported NIPs. Basic NIPs can be included by default. Extra NIPs can be specified at build time be via feature flags.
 - A `nostr-cli` that can be used as a manager of `nostrd` on the server-side. It can also talk the nostr protocol with any other relay and can be used as a cli nostr client. Maintenance access can be provided to a relay via basic or cookie authentication.
 - A rich `nostr-API` library. Included in the project, that can be used as an easy dev tool for devs to build their nostr clients. These APIs can then be exposed via ffi to other languages and will give developers a one-stop tool to build their cool Nostr clients.
  - Finally, a `Bitcoin` layer using BDK to facilitate the "money interface".
 - All these components will be using each other and will be maintaining thorough integration checks in the project pipeline to make sure they are compatible with each other as well as compatible with the global `nostr` and `Bitcoin` protocol.
 - An out-off-bound e2e secret sharing protocol for clients to talk to each other to share encrypted subscription data and relay references.


 Once all these are laid out, it's then possible to use a `rust-nostr` suit to come up with all sorts of more complex clients doing Bitcoin Defi among each other.

# Concluding **Remarks**
So far this just a primitive idea, and I don't even know what the possible unknown challenges are. I expect many. This is my attempt to chalk down a way forward to reach both Bitcoin Defi and DSNs to solve each other's problems, using a very dumb protocol, and highly intelligent clients.

Over time I will try to sketch up a basic design of `nostrd`. The idea needs more polishing and a thorough examination of available techs and possible different approaches.

If you happen to care about this problem and feel like lending a hand, or if you have any suggestions or comments in general, hit me up at @rajarshimaitra on Twitter. DM's open.







