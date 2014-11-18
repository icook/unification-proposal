The case for a unified cryptocurrency toolkit
=============================================

I've been working with Bitcoin and cryptocurrency in general for about 9 months
now, and in that time I've really felt the want for a quality toolkit that is
flexible and well documented. Certainly python-bitcoinlib is about the closest
I've found. However, the numerous other projects with part way overlapping
goals in this space make for a fragmented ecosystem and I believe hinder
cryptocurrency adoption by making it less approachable for developers. For
example, in the Python realm I know of several projects that achieve very
similar goals:

* stratum-mining, probably the most used stratum software uses its own custom
  serialization libs.
* p2pool uses yet another set custom made tools
* My own project, powerpool, has globbed together its own called cryptokit from
  various sources
* @richardkiss has pycoin and pycoinnet
* @spesmilo's Electrum uses its own libraries

And in the non-Python realm I'm not as knowledgeable, but these are what I know
of:

* spesmilo's sx (C++)
* @conformal GoLang libraries they built
* bitcoinj

I believe that establishing a library of sufficient flexability and
documentation would gradually lead to a unification of effort among developers
in this space, leading to a defacto general purpose toolkit. I think this is a
really important foundation for spreading cryptocurrency adoption, since
developers make applications, and in turn more applications bring more users.

Below I've attempted to identify the major use cases, and roughly which
libraries are meeting each use case.

* Wallet Management. For increased security it's a logical step to store
  private keys on a different server that is not connected directly to
  production systems. (SX, pycoin)
* Altcoin use. Many of the Bitcoin libraries aren't sufficiently general to
  allow using them for Altcoins, and so they get forked and patches and tweaks
  get scattered everywhere. (pycoin is generalized, stratum-mining is forked
  endlessly)
* Survey or special purpose nodes. These are node implementations that want to
  do something that is a significant departure from Core. (python-bitcoinlib,
  pycoinnet, electrum)
* Reasearch and Analysis. These needs are wide and varied, but something like
  python-bitcoinlib ususually covers it well.

So I would like make an RFC of sorts, attempting to define requirements for a
community maintained library that satisfies most general use cases. Below is a
very rough outline of my goals.

Language Choice
===============

I believe that Python makes an excellent choice, given:

* Wide adoption/use in many current crypto projects.
* Good cross platofrm capabilities.
* Good knowledge base across proposed maintainers of this project.

The main drawbacks include:

* Cannot be used as a library in other languages like a lower level language
  such as Rust, C/C++.
* Poor concurrency support.
* Headaches with 2.7->3 bytes/strings/unicode.

While the drawbacks are real, I think the best course of action would be to
build the library in Python, then slowly port chunks of functionality to a
lower level library and use them in Python via FFI.

Feature Summary
===============

**Network Generalization**

* Network parameters should be defined in such a way that altcoins that make no
  innovations on Bitcoin are supported trivially, without modifying the module.
  pycoin is very close to supporting this, but not quite.
* Network parameters should be able to define classes that override the default
  Bitcoin style classes. For instance, if a network encodes its transactions
  slightly differently it should be simple to provide a custom Transaction
  class that drops in seamlessly. IE, any code that references Network specific
  objects will have to know the network context in order to use the correct
  object definition.
* Network context will not be defined globally, as to allow an application to
  perform actions using multiple networks.

**Wallet Support**

* Support for BIP032 style keys. Most of this work is done in pycoin.
* Support for deserializing/serializing wallets generated by core.
* A flexible, uniform format for storing wallets that is human readable. Likely
  a JSON format similar to that of Electrum.
* Configurable key stretching for wallet encryption.

**Core client compatibility**

* Ability to serialize/deserialize data structures both on disk and over the
  wire from the Core client.
* Complete ability to run consensus critical validation.
* Ability to mirror most if not all generation semantics that the core client
  implements (for instance calculating fee amounts, or generating block
  templates)

Community Support Goals
=======================

In order to allow the project to become mainstream, certain efforts should be
made to facilitate adoption and contribution.

* Automated testing and testing code coverage checks. This eases the burden on
  the maintainers and keeps the repository less cluttered by catching obviously
  sloppy contributions.
* Established coding standards/conventions. Code bases that vary their coding
  standard can be frustrating to read and work on.
* Quality documentation. This is probably the biggest lacking area in
  cryptocurrency right now. There's a standard of almost no documentation on
  everything, making getting into crypto a big hurdle. Varying levels of
  stricness could be enforced here for contributions.
* Positive general attitude towards contributors/users, and a goal of making
  comprehensible documentation.

Maintainers
=======================

Ideally I'd like to get the support of a few other developers who have
experience in this area and are willing to help either move a project in this
direction or create one from scratch. It is also possible that the project
could be crowdfunded to an extent, although explaining its value to
cryptocurrency users is a challenge.

License
=======================

This is an area that I'm not particularily knowledgable about. I know a lot of
the existing project have slightly different licenses, and it's likely this
project would end up taking bits and pieces from many diverse projects, so it's
unclear the problems that might create.

In general BSD or similar would be preferred over GPL or similar.
