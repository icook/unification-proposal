The case for a unified cryptocurrency toolkit
=============================================

I've been working with Bitcoin and cryptocurrency in general for about 9 months
now, and in that time I've really felt the want for a quality toolkit that is
flexible and well documented. The numerous other projects with part way
overlapping goals in this space make for a fragmented ecosystem and I believe
hinder cryptocurrency adoption by making it less approachable for developers.
For example, in the Python realm I know of several projects that achieve very
similar goals:

* python-bitcoinlib is probably the most well known/general, but it Bitcoin only
* stratum-mining, probably the most used stratum software uses its own custom
  serialization libs.
* p2pool uses yet another set custom made tools
* My own project, powerpool, has globed together its own called cryptokit from
  various sources
* @richardkiss has pycoin and pycoinnet
* @spesmilo's Electrum uses its own libraries

And in the non-Python realm I'm not as knowledgeable, but these are what I know
of:

* spesmilo's sx (C++)
* @conformal GoLang libraries they built
* bitcoinj

I believe that establishing a library that is sufficiently flexible and
generic would gradually lead to a unification of effort among developers
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
* Research and Analysis. These needs are wide and varied, but something like
  python-bitcoinlib unusually covers it well.

So I would like make an RFC of sorts, attempting to define requirements for a
community maintained library that satisfies most general use cases. Below is a
very rough outline of my goals.

Goals
===============
The ultimate goal here would be to develop an open source cryptocurrency
toolkit that has enough users and value that it was relatively
self-sustaining. That is, that it was consistently getting better due to
contributions from a wide range of individuals, instead of a single central
maintainer. The sections below should be guided by this.

At it's core, any successful open source software is made by it's community,
but communities don't just pop up when there's code. I think the most important
thing missing in most of the aforementioned projects are steps to facilitate
and actively encourage community, so I've outlined here what I think would help
make that happen.

Encouraging contribution
------------------------
Steps that should be taken to make contribution as easy as possible.

### Quality Assurance

One of the maintainers jobs is reviewing contributions. The easier this is the
more quickly contributions will get accepted, and thus the more likely people
are to contribute again. Or on the flip side, if review isn't sufficient
(sloppy) then the quality of code base will decline and it's user base (health)
will decline as well.

Main tasks included
* Making new code doesn't break things (Automated testing with TravisCI)
* Ensuring there's sufficient testing coverage (Coveralls)
* Maintaining uniform coding standards (Manually with PyLint or Flake8)

### Welcoming Attitude

Open source can be rough sometimes, depending on the culture of the project.
Ideally the maintainers should be open and supporting, finding more eloquent
ways to tell someone "this is shit".

### Contributors List

Thanking contributors in release announcements and adding them to contributors
docs will help make contributors feel welcomed and appreciated.

Easing (New) Developer Experience
---------------------------------
The easier a library is to use, the more people will use it.

> “If a new user has a bad time, it’s a bug” - Logstash maintainer Jordan Sissel

### Documentation

The docs should be thorough and cover common use cases well. Actively
requesting documentation contributions will help. ReadTheDocs has an edit on
Github button which is great.

### Organization

Having a logical project layout makes exploring the code much more
straightforward. This isn't a problem for small projects, but becomes
increasingly problematic as projects grow. Refactoring has a high (end user)
cost, so a bit of though early on helps prevent problems down the road.

### Coding Standards

Readability is important for a project that expects lots of eyes on the
code base. Establishing and sticking to standards helps a lot. How to do this is
well covered in other places, so I won't dwell on it here.

### Answer Questions

IRC, mailing lists, or Github issues.

### API

Exposing a logical and well thought out API makes using the package pleasant,
leading to greater adoption. Flask, Requests, etc are good examples. Actually
building things with the library helps highlight the pain points.

Proposed Implementation
=======================

*Note*: These ideas still all very green, and very open to suggestions and
changes. This is basically a brain dump, and an ideal *end point*.

Feature Summary
---------------

### Network Generalization

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

### Wallet Support

* Support for BIP032 style keys. Most of this work is done in pycoin.
* Support for deserializing/serializing wallets generated by core.
* A flexible, uniform format for storing wallets that is human readable. Likely
  a JSON format similar to that of Electrum.
* Configurable key stretching for wallet encryption.

### Core client compatibility

* Ability to serialize/deserialize data structures both on disk and over the
  wire from the Core client.
* Complete ability to run consensus critical validation.
* Ability to mirror most if not all generation semantics that the core client
  implements (for instance calculating fee amounts, or generating block
  templates)

Module Layout
-----------------

* String handling and serialization - provide generic wrapper classes for all
  data structures and handle python2/3 compatibility
* ECDSA - compartmentalize all ECDSA implementations. Probably do some magic
  like [this](https://github.com/gorakhargosh/watchdog/blob/master/src/watchdog/observers/__init__.py).
* cli - Provide a cli for the toolkit
* Network specific data structures
    * Generic structure logic - validation, fee calculation, etc
    * Network messages
    * Network serialization
    * Key handling
    * Core client disk serialization (same as net, but logical separation is
      worth the minimal effort)
* Wallet management (not core wallets)
* Network definitions - A wrapper class that specifies all classes to use for a
  specific network. [POC here](https://github.com/icook/cckit)

Rough Timeline
-----------------

* Design a network parameter system that meets the goals. That is, more
  concretely:
    * A user can define their own network config outside of the module
    * "Active" network configurations are not global (currently an issue in
      python-bitcoinlib). A few possible ways to implement this.
    * Network definitions specify all various data structures and parameters.
      Networks with same data structures as Bitcoin (IE, Litecoin) will simply
      use the Bitcoin data structures (IE class will inherit from
      Bitcoin).
* Decide on uniform serialization and string handling. python-bitcoinlib and
  pycoin's are similar, but not quite the same.
* Decide on coding standards, and adapt all incoming code to said standards.
* Merge featuresets of richardkiss/pycoin and petertodd/python-bitcoinlib
    * Migrate all serialization to selected system
    * Abstract use of network parameters
    * Merge tests
* Add testing, coverage, docs.
* Write/adapt a blockchain class. Possibly similar to SAX XML parsing.
* Write use case examples, improve documentation

Misc
=======================

Maintainers
-----------------------

Ideally I'd like to get the support of a few other developers who have
experience in this area and are willing to help either move a project in this
direction or create one from scratch. It is also possible that the project
could be crowd-funded to an extent, although explaining its value to
cryptocurrency users is a challenge.

License
-----------------------

This is an area that I'm not particularly knowledgeable about. I know a lot of
the existing project have slightly different licenses, and it's likely this
project would end up taking bits and pieces from many diverse projects, so it's
unclear the problems that might create.

In general BSD or similar would be preferred over GPL or similar.

Language Choice
---------------

I believe that Python makes an excellent choice, given:

* Wide adoption/use in many current crypto projects.
* Good cross platform capabilities.
* Good knowledge base across proposed maintainers of this project.

The main drawbacks include:

* Cannot be used as a library in other languages like a lower level language
  such as Rust, C/C++.
* Poor concurrency support.
* Headaches with 2.7->3 bytes/strings/Unicode.

While the drawbacks are real, I think the best course of action would be to
build the library in Python, then slowly port chunks of functionality to a
lower level library and use them in Python via FFI.
