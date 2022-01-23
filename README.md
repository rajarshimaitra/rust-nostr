# rust-nostr

## Intro
A complete suite of nostr Bitcoin libraries that can be used to develop Decentralized Social Networks (DSN) with integrated Bitcoin financial products.

If you like the sound of it, check out the [vision](VISION.md) doc for the whole idea.

This is an idea intended to be built entirely in open public from the very start. The design questions are completely open and everybody is encouraged to participate, either via code or via ideas. If you are a new rust dev there will be ton of testing works for you to help with to polish up your rust skills.

A [Discord server](https://discord.gg/pFZNEFtH) is created to facilitate higher bandwidth dev discussions. That's the best place to start if you are excited about this.

This repo will describe the state of the project. Issues will be used to open design questions and suggestions in public. If you have a specific design idea, design question, or literally anything, feel free to open an Issue.

## Project Goals

The goal of the project is to finally produce a `nostrd` binary, a `nostr-cli` cli client, and `nostr-API`. Initial inspirations can be taken from existing nostr rust libraries

`nostrd`: will be a highly efficient nostr-relay with multiple opt-in database support, that can be activated by feature flags. It should have a well-defined DB interface, that can be used to add any other DB of choice very easily by the user of the library. By default, it will work with `SQLite` and `sled`. DB will have extensive unit testing. More DB options can be added later easily by external contributors. The difference between having `nostrd` with your own favorite DB, and having it by default should be one PR that doesn't touch any other part of the code, easy to review, and passes all the DB tests. Finally, it will have a Bitcoin wallet, interfaced to a public node (electrum or esplora) or a local bitcoin core node, using BDK. An Existing rust nostr relay is [nostr-rs-relay](https://github.com/scsibug/nostr-rs-relay).

`nostr-cli`: will be a command line maintainer tool that can talk with `nostrd`. It will have functional access to a `nostrd` via RPC through basic or cookie authentication. This should provide all the relay utility functions and logging. This will be used as a "controller" of the relay. `nostr-cli` is to `nostrd` what `bitcoin-cli` is to `bitcoind`. No existing code available for this part.

`nostr-API`: A API module that will expose all the nostr communication rules that client devs can use to build their own nostr clients. Existing nostr API library in rust is [nostr-rs](https://github.com/futurepaul/nostr-rs).

`portal`: An encrypted communication protocol to facilitate subscription sharing. This will be an out of bound protocol that doesn't need to be strictly defined, or heavily security guaranteed. Inspiration can be taken from existing project like [CypherPost](https://github.com/i5hi/cypherpost) and simplified example implementation in rust will be added in this repo. But anybody is free to come up with their own custom implementation of encrypted subscription sharing, and be compatible with the whole idea. This part is the lowest priority for now. Discussion is on route with CypherPost folks to have nostr compatible version of their project, which will basically be an alternate `portal`.

To maintain compatibility between all these parts integration tests are to be made. `nostr-cli` can include `nostr-API` to become not only a controller but a command-line nostr-client also. Then in the integration tests `nostr-cli` will use `nostr-API` to talk with `nostrd` and ensure they are always compatible with each other. These tests can be included in the Github CI pipeline to maintain consistency at each commit.

The basic guiding principles for writing code for this project are:
 - Use extensive documentation to describe all parts of the code.
 - Use rust compiler to its fullest by defining every protocol rule as a struct enum or trait.
 - Use rigorous unit and integration testing to ensure all parts are working as expected (code coverage goal > 80%).
 - Have the code very structured and divided into straightforward units.
 - Avoid complexity at all costs. Be dumb and clear, not clever and obscured.

## Roadmap
- [ ] basic code for `nostrd` with `sqlite`.
- [ ] Support `mandatory` NIPs
- [ ] Support optional NIPs
- [ ] Cover all parts in unit tests
- [ ] Define DB interface
- [ ] Support `sled` DB
- [ ] Implement a RPC interface
- [ ] Basic code for `nostr-cli`
- [ ] Integration test framework with `nostr-cli` and `nostrd`
- [ ] Define `nostr-API`
- [ ] Implement `nostr-API` in `nostr-cli`
- [ ] Add Bitcoin support with `BDK`