# Nostr, a Basic tour

Nostr is a very lightweight open protocol that "has a chance to work" (as per the project doc) as a decentralized social media platform. The protocol specs are defined in NIPs (Nostr Improvement Proposals) and can be found [here](https://github.com/fiatjaf/nostr/tree/master/nips).

The basis of the protocol is a WebSocket server (called a nostr-relay) that handles and stores a very simple data structure called an [`Event`](https://github.com/fiatjaf/nostr/blob/master/nips/01.md#events-and-signatures). It looks like the following:

```
{
  "id": <32-bytes sha256 of the serialized event data>
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

Events are always signed (using Schnorr sigs) and they contain structured data that can have semantics. A Schnorr type XOnlyPubkeys as defined in [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki#design) (and currently in use with Bitcoin Taproot) are used as "identities" throughout the protocol.

A nostr-client is an APP that can talk with a nostr-relay and can subscribe to any set of events using a [`Subscription Filter`](https://github.com/fiatjaf/nostr/blob/master/nips/01.md#communication-between-clients-and-relays). The filter represents the set of nostr `Events` that a client is interested in.

There is no sign-up or account creation for a client. Clients are identified with their pubkeys. Every time a client connects to a relay, it submits its subscription filters and the relay streams the "interested events" to the client as long as they are connected.

A relay can cache the client subscriptions but it doesn't have to. Clients are supposed to handle everything at the "client-side", and relays can be dumb as a rock.

Clients don't talk to each other. But relays can. This allows relays to fetch data for a client that it doesn't have. And Clients get to subscribe to events outside of their connected relays.

This at first glance gives the image of the uselessness of Nostr as a protocol, (why not just sign and dump raw JSON and let clients figure it out?), but on a deeper look, the "dumb-server, smart client" model can be found to have some massive engineering benefits, especially in decentralized protocol designs.

This document is an outline of how these dumb servers, smart clients, the Bitcoin network and e2e encryption can come together to solve the problem of "Decentralized Social Networks", DSNs (a buzzword I just came up with).

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

Because they have to satisfy real-life design constraints of such platforms, while providing their product at scale, they end up creating a small cohort of a team dedicated to designing the approach of the platform. This makes casual and fun applications using the platform difficult for client devs. At some point, they might as well decide to design their little protocol, but eventually, they will reach the same roadblock. Nobody wants to voluntarily build on a platform that you designed for a specific niche.

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

As odd as it might seem, such a simple protocol is getting more dev attention than many of the "well designed" existing alternate social media.

The project already got a ton of dev interest and a rich ecosystem of [libraries, apps, relays](https://github.com/aljazceru/awesome-nostr) have been developed by the community in almost no time. And the list is growing every day.

The Telegram chat has around 400 members and growing every day.

Why? "Because it's so simple".

This simplicity allows anyone with interest to easily write a JSON streamer and start talking the protocol to any existing relay.

A real-time list of current running Nostr relays can be found [here](https://nostr-registry.netlify.app/).

There already exists two work-in-progress twitter-like apps [branle](https://branle.netlify.app/) and [NOSTR-twitter](https://github.com/arcbtc/nostr).

People are also coming up with new extra details added on top of basic NIPs almost regularly.

The simplicity of the protocol allows devs to quickly converge on the open standard, and have all the complexity on the client side. The entire app experience would be handled by clients, and relays would remain dumb data servers. This allows devs to move and iterate fast on the client apps while being compatible with any available relays.

This also adds to client compatibility. It's possible to have two different apps, but still, be able to see each other's posts. The platform can be decentralized at its core, and clients become compatible with each other via a simple storage protocol. That's the neat thing with the "dumb server, smart client" model. Quick agreement on rudimentary standard, faster iteration on cool client apps.

Complexities can be customized at the client layer while interoperability is achieved at the relay layer.

# The Missing Pieces

Once we converge on what the core lego looks like. The remaining pieces are DOS protection, relay incentivization and some ways to communicate nostr subscription data between users.

### Putting Bitcoin inside Nostr

Thanks to Bitcoin, relay incentivization and DOS protection can be solved in one go.

A robust social network cannot be built if its infrastructure rests on a flimsy foundation of "voluntarism". And as we know "if a product is free, you are the product". There should be native integration of these future media platforms into Bitcoin.

The one-stop solution to do this is to use [BDK](https://github.com/bitcoindevkit/bdk). A highly performant bitcoin wallet library, flexible enough to handle multiple kinds of bitcoin interfaces and DBs. Added with some new NIPs to define `payment request` and `payment response` Event kinds.

The payment can be either a one-time onchain transaction or a stream of LN payments between Client and Relay, for each event they publish. (Will need BDK + LDK which is under active development). Relay can set their feerate in sats/byte, they can choose to set it to 0 if they wanna be "voluntary".

This gives a good way for high-maintenance public relays to monetize their service, at the same time protecting them from DOS.

### E2E Encrypted Subscription Sharing

Remember, a Nostr relay is just a dump of simple JSON data. Fetched via a `subscription` filter. This allows nostr to be a generic data-sharing platform between clients. With Bitcoin inside, now we are talking about Bitcoin scripts, descriptors, DLC contracts, and other Bitcoin DeFi information, shared via the nostr relay network. But these can be sensitive information, and should not be shared on a public platform in cleartext.

For this, an encrypted nostr subscription sharing mechanism is required. This can be another server facilitating only encrypted subscription data sharing among participants.

This can be achieved by following:
 - Encrypt the [`subscription` + `relay-address`] using a DH shared secret derived from the pubkey of the intended receiver.
 - Post the encrypted data along with the pubkey of the recipient to this server.
 - Recipient client gets notification, downloads and decrypts the data, gets the subscription to fetch the actual data from nostr.
 - The actual data is also a ciphertext encrypted by the same shared secret so the recipient knows how to decrypt that too.

These servers can be very lightweight as they don't need to store all the historical subscription data. They can regularly clear old data, or even can clear it in real-time when it knows the recipient has downloaded it. This will make them very low cost, and it doesn't need to solve the incentivization problem.

These servers don't need to follow any generic protocol. Can be implemented freely via any design. They just need to have a way to connect to clients and know when to notify them when something relevant to them has come.

They are also censorship-resistant like nostr relay. If one goes down or stops working, anyone can spin up another one. Because they don't have to keep a historic record, switching from one server to another does not affect the overall flow of information.

These servers cannot exploit the data also because all they see is an encrypted blob of randomness, so they do not need to be highly secured.

### The Final Picture

So now combining all of these, Nostr, Bitcoin, and Encrypted Subscription sharing, what we have now is a very powerful and *default private* social network that can share data among participants using some very generic and global protocols.

This allows the possibility for *hidden* social networks of people to selectively open up their posts to particular trusted entities.

These posts can be DLC contracts, descriptors describing multisig between many parties, DLC oracle publishing only to subscribed members, and a lot more.

In this framework, the basic unit of "Identity" is a pubkey. Pubkeys are analogous to aliases in the real world. Any person can have any number of aliases. If one alias gets compromised, they can quickly create another one, just like we create a new bitcoin address for every payment.

Using pubkeys as aliases it's then possible to selectively open up to your own private trusted network. You can have one pubkey associated with your global Alias (your Twitter handle that everyone knows), and then have any number of parallel aliases to only communicate among specific groups of people, or use a specific app.

Data related to all these pubkeys will remain completely unrelated and can be distributed across multiple nostr relays.

The final summarised model is:
 - A Highly interoperable and extremely simple relay protocol, nostr.
 - A flexible framework for adding new relay features using `optional` upgrades that relays can opt-in.
 - A encrypted subscription sharing mechanism to pass around nostr subscriptions.
 - Bitcoin native integration to facilitate the "internet of money" and DOS protection at the same time.
 - A decentralized publication layer for clients to publish public as well as private content.
 - Client-side complexity to interpret these contents and have native financial contract generation UI using the powers of Bitcoin.

And unlike everything web3.0, this doesn't involve another "Blockchain" (bummer, I know).

# Path Ahead

As good as it may sound, we are not there yet. And a lot of engineering designs are required to realize these dreams. There are unknown problems ahead that need to be dealt with. The design decision of these relays and clients needs to be carefully laid out. Just having a simple protocol is not good enough.

These relays should be efficient, robust, rigorously peer-reviewed in open public, guaranteed at the basic level of security. The work has to be done in open public, and the elements should be designed to be as flexible as possible to satisfy diverse client developer needs.

If this thing needs to extend out into professional services that people can deploy in their servers, and serious products are built out of it, we need more than just hobby codes and example apps.

What's needed is not another cool nostr app, but a well thought out and designed infrastructure library that can be used by shadowy super coders to build the next cool nostr app with "Bitcoin inside".

# Introducing rust-nostr

rust-nostr is a project at the idea stage that aims to solve the above. The idea is to provide a complete one-stop suite of nostr infrastructure, which is modular, easily extensible, has strong security guarantees, well documented, very easy for devs to customize as per their own need, very easy to just download, deploy and manage in their servers.

The entire structure is still TBD, but a rough outline of how rust-nostr will look like is following.

 - A binary crate producing `nostrd`. A lightweight and efficient rust implementation of Nostr-Relay. `nostrd` will come with a set of supported NIPs. Basic NIPs can be included by default. Extra NIPs can be specified at build time be via feature flags.
 - A `nostr-cli` that can be used as a manager of `nostrd` on the server-side. It can also talk the nostr protocol with any other relay and can be used as a cli nostr client. Maintenance access can be provided to a relay via basic or cookie authentication.
 - A rich `nostr-API` library. Included in the project, that can be used as an easy dev tool for devs to build their nostr clients. These APIs can then be exposed via ffi to other languages and will give developers a one-stop tool to build their cool Nostr clients.
 - `portal` is an encrypted nostr subscription sharing server. Specification of `portal` is not part of the project as it is already a solved problem. This has been well understood in cryptographic literatures and lots of candidate implementation exists in open source. The Signal App itself is an example of a portal, although very difficult to use for this use case. A local team in India have been focused on this problem for specific use case of facilitating p2p Bitcoin trades called [CypherPost](https://github.com/i5hi/cypherpost), which is already a very fitting `portal` implementation. Eventually a stripped down version of a candidate implementation in rust will be added to the project repository. But people are free to develop and use their own portals and still be compatible with the rest of the network.

All of them (except `portal`) will have native Bitcoin + Lightning integration in them via BDK and LDK.

To ensure all parts of the infrastructure are always in sync with each other they will have rigorous integration tests in the project's CI pipeline.

Once all these are laid out, it's then possible to use a `rust-nostr` suit to come up with all sorts of more complex clients doing Bitcoin Defi among each other.

# Concluding Remarks
So far this is just a primitive idea, and I don't even know what the possible unknown challenges are. I expect many. As they say "the devil is in the details". This might seem like a very ambitious project, but it's not.

By limiting the scope of the project to provide very specific building tools, this is pretty much achievable by a few motivated rust devs laying it out. Rust is also the most suitable language to build this in because it lets us strictly define the protocol rules at the compiler level so it reduces the room to go wrong, while also producing very concise and easy to audit code.

By not trying to make "a product" and only solving for the lego pieces, I think this is well within reach.

This project can then pave the way for Bitcoin entrepreneurs to come up with all sorts of applications. The limit in application space is only limited by imagination.

So if you happen to care about this problem and feel like lending a hand, or if you have any suggestions or comments in general, hit me up at @rajarshimaitra on Twitter. DM's open.







